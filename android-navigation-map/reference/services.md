# Service Mapping Reference

Use this reference only when the user asked to map the services each screen consumes. The goal is a readable chain:

`Screen → ViewModel → UseCase → Repository → GET /items/{id}`

Resolve statically. If a link cannot be resolved, keep the known part, append `→ ?`, and add the reason to `unresolved`.

Also populate each screen's optional `apis` array automatically for resolved HTTP calls. Do not ask separately. Follow actual screen-triggered calls across each layer; an injected dependency or an endpoint declared on an interface is not evidence that the screen consumes it. Include a method, path and `path:line` source for each endpoint, and trace the call chain through DI to the implementation. For ambiguous bindings, dynamic paths or untraceable calls, record the known facts and source in `unresolved` rather than guessing an endpoint or attaching every candidate to the screen.

## 1. Build a symbol index

Before tracing, build a small index of the project:

- Classes, interfaces, objects, and composables, with their package and file.
- Constructor parameters and annotations (`@Inject`, `@HiltViewModel`, `@Serializable`).
- Injected properties (`@Inject lateinit var`).
- DI modules (`@Module`, `@Binds`, `@Provides`, Koin `module { ... }`).
- API interfaces and their HTTP annotations.
- DAOs and their Room annotations.

Read only the files you need. Use `rg` to locate a symbol, then read the declaration and its call sites. Skip `build/`, generated sources, tests, and samples.

## 2. Screen to ViewModel

Compose:

```kotlin
val viewModel: ProfileViewModel = hiltViewModel()
val viewModel: ProfileViewModel = viewModel()
val viewModel: ProfileViewModel = koinViewModel()
val viewModel: ProfileViewModel = viewModel(factory = ProfileViewModel.Factory)
```

Navigation-scoped:

```kotlin
val viewModel: DetailViewModel = hiltViewModel(backStackEntry)
```

Fragment:

```kotlin
private val viewModel: ProfileViewModel by viewModels()
```

Activity:

```kotlin
private val viewModel: ProfileViewModel by viewModels()
```

Manual creation:

```kotlin
val viewModel = remember { ProfileViewModel(profileRepository) }
```

Rules:

- `hiltViewModel`, `viewModel`, and `koinViewModel` give you the ViewModel type.
- `by viewModels` and `ViewModelProvider` give you the ViewModel type.
- Manual creation is also a direct dependency edge; use the constructor arguments as the next step.
- If the screen has no ViewModel, look for direct repository or use-case calls in the composable, Fragment, or Activity.
- Do not treat `SavedStateHandle`, `Context`, `Resources`, `NavController`, or UI state holders as services.

## 3. ViewModel to dependencies

Read the ViewModel constructor:

```kotlin
@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val getProfile: GetProfileUseCase,
    private val analytics: AnalyticsTracker
) : ViewModel()
```

```kotlin
class ProfileViewModel(
    private val repository: ProfileRepository
) : ViewModel()
```

Rules:

- Constructor parameters and field injection identify candidate dependencies; trace the methods actually called by the screen or its ViewModel before including their operations.
- `SavedStateHandle` is navigation state, not a service.
- If a dependency is a framework type (`Context`, `Application`, `Resources`), skip it.
- If a dependency is an interface, resolve the implementation in the next step.

## 4. Use case and repository to data source

Retrofit:

```kotlin
interface UserApi {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: String): User

    @POST("users")
    suspend fun createUser(@Body body: CreateUserRequest): User
}
```

Build `GET users/{id}` and `POST users`. Include the base URL only when it is a literal and easy to resolve; otherwise keep the path.

Ktor:

```kotlin
suspend fun getProfile(id: String): Profile =
    client.get("users/$id").body()
```

Apollo / GraphQL:

```kotlin
val response = apolloClient.query(GetProfileQuery(id)).execute()
```

Room:

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getUser(id: String): User
}
```

Firebase:

```kotlin
FirebaseFirestore.getInstance().collection("users").document(id).get()
```

Rules:

- Prefer the endpoint or DAO method over the repository method name.
- If a traced repository method calls several endpoints, list each reachable endpoint as a separate chain; do not include endpoints used only by unrelated methods.
- If the repository calls another repository or use case, continue the chain.
- If the call is hidden behind an interface, resolve the implementation first.
- For third-party SDKs, use `SDK: <operation>` only when the call is clearly a service (analytics, billing, auth). Do not list every library call.

## 5. Resolve interfaces with DI

Hilt:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class DataModule {
    @Binds
    abstract fun bindProfileRepository(impl: ProfileRepositoryImpl): ProfileRepository
}

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    fun provideUserApi(retrofit: Retrofit): UserApi = retrofit.create(UserApi::class.java)
}
```

Koin:

```kotlin
val dataModule = module {
    single<ProfileRepository> { ProfileRepositoryImpl(get()) }
    viewModel { ProfileViewModel(get()) }
}
```

Manual:

```kotlin
object ServiceLocator {
    val profileRepository: ProfileRepository by lazy { ProfileRepositoryImpl(userApi) }
}
```

Rules:

- `@Binds` maps an interface to its implementation. Follow it.
- `@Provides` shows how an instance is built. Use the returned type and, when useful, the constructor arguments.
- Koin `single<T> { Impl(get()) }` maps `T` to `Impl`. Follow `get()` by type when it is explicit.
- Qualifiers (`@Named`, custom `@Qualifier`) disambiguate multiple bindings; include them only when needed.
- If several implementations match, list candidates in `unresolved` and mark the chain with `?`; do not assign their endpoints without evidence that the screen calls them.
- Generated Hilt or Dagger code is not in source; use the annotations.

## 6. Output format

One string per chain:

```json
"services": [
  "ProfileViewModel → GetProfileUseCase → ProfileRepository → GET /users/{id}",
  "ProfileViewModel → AnalyticsTracker → SDK: analytics.logEvent"
],
"apis": [
  { "method": "GET", "path": "/users/{id}", "source": "data/UserApi.kt:3" }
]
```

Guidelines:

- Use `→` between links.
- Omit package names unless two types share a simple name.
- Keep the chain short; collapse a pass-through use case only if it adds no information.
- Deduplicate identical chains.
- Deduplicate `apis` by method and path per screen; the source points to the endpoint declaration or actual HTTP call. Keep non-HTTP Room, Firebase, Apollo and SDK operations in `services`, not `apis` unless their HTTP method and path are independently established.
- For a dynamic method or path that cannot be established, put the reason and source in `unresolved` instead of fabricating an API.
- If a link is unresolved, write `→ ?` at the break point and add an `unresolved` entry with the source.
- If a screen calls a repository directly, start the chain with the screen.

## 7. Uncertainty

Mark the chain as uncertain when:

- The target is chosen at runtime or through reflection.
- A service locator returns a type without an explicit implementation.
- Multiple implementations satisfy the same interface.
- The dependency is created in generated code that is not available.
- The call is inside a lambda passed to another component and cannot be traced statically.

Do not guess an implementation from a class name. Prefer a partial chain with `?` over a wrong chain.

## 8. Performance

- Index class names to files once, then reuse the index.
- Search by symbol, not by reading every file.
- Stop tracing when you reach a framework type or a third-party SDK.
- Cap the chain depth at about eight links; deeper chains are usually noise.
- For large projects, map only the screens the user asked about, or the screens in the rendered graph.

## 9. Examples

Hilt + Retrofit + Compose:

```text
DetailScreen
→ DetailViewModel (hiltViewModel)
→ GetItemUseCase
→ ItemRepository
→ ItemApi
→ GET /items/{itemId}
```

Koin + Ktor:

```text
FeedScreen
→ FeedViewModel (koinViewModel)
→ FeedRepositoryImpl
→ Ktor client
→ GET feed
```

Fragment + Room:

```text
HistoryFragment
→ HistoryViewModel (by viewModels)
→ HistoryRepository
→ HistoryDao
→ SELECT * FROM history
```

## 10. Anti-patterns

- Do not map `SavedStateHandle`, `NavController`, `Context`, `Resources`, or `Application` as services.
- Do not map UI state holders or `StateFlow` as services.
- Do not infer an endpoint from a method name; read the annotation.
- Do not cross module boundaries without a reference in source.
- Do not list every SDK call; include only the ones that represent a consumed service.
- Do not claim a chain is complete when any link is unresolved.
