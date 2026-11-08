# Navigation Detection Reference

Exact patterns for building the graph. Record `path:line` for every node and every edge. Use forward slashes in paths and 1-based line numbers.

## 1. Inventory the project

Start with the module list and dependencies. They tell you which navigation, DI, networking, and database stacks to look for.

```kotlin
// settings.gradle.kts
include(":app", ":feature:home", ":feature:detail")
```

```kotlin
// app/build.gradle.kts
implementation("androidx.navigation:navigation-fragment-ktx")
implementation("androidx.navigation:navigation-compose")
implementation("androidx.navigation3:navigation3-runtime")
implementation("androidx.navigation3:navigation3-ui")
implementation("androidx.hilt:hilt-navigation-compose")
implementation("io.insert-koin:koin-androidx-compose")
implementation("com.ramcosta.composedestinations:destinations")
implementation("cafe.adriel.voyager:voyager-navigator")
implementation("com.arkivanov.decompose:decompose")
```

Also check `gradle/libs.versions.toml` when it exists. Use the module list to assign each node to a module and a group.

## 2. AndroidManifest.xml

Extract:

- Every `<activity>`, `<activity-alias>`, and `<service>` that participates in navigation.
- The launcher entry: `MAIN` + `LAUNCHER`.
- Explicit and implicit intent filters.
- `android:parentActivityName` (up navigation).
- `<nav-graph android:value="@navigation/nav_graph" />` when present.
- Deep links: `VIEW` + `BROWSABLE` + `<data>` with scheme, host, and path.
- Notifications, shortcuts, and widgets that open a screen.

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:parentActivityName=".HomeActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="detail" android:pathPrefix="/item" />
    </intent-filter>
    <nav-graph android:value="@navigation/nav_graph" />
</activity>
```

Rules:

- A launcher Activity is a root node. Do not require an incoming edge.
- An implicit intent with no resolvable target becomes an `external` node or an `unknown` edge. Put it in `unresolved` when the destination cannot be determined.
- `parentActivityName` is an up-navigation edge. Mark it `intent` or `deepLink` depending on how it is used; keep the label explicit.

## 3. XML Navigation Component

Scan `res/navigation/*.xml`.

```xml
<navigation
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.HomeFragment">
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/detailFragment"
            app:popUpTo="@id/homeFragment"
            app:launchSingleTop="true" />
        <argument
            android:name="userId"
            app:argType="string"
            android:defaultValue="guest" />
    </fragment>

    <include app:graph="@navigation/checkout_graph" />

    <deepLink app:uri="myapp://detail/{id}" />
</navigation>
```

Extract:

- `<fragment>`, `<activity>`, `<dialog>`, and nested `<navigation>` elements.
- `app:startDestination` as the entry edge into that graph.
- `<action app:destination="...">` as an `action` edge.
- `<argument>` on the destination as navigation arguments.
- `<include app:graph="...">` as an `include` edge or a nested graph.
- `<deepLink app:uri="...">` as a `deepLink` edge.
- `app:popUpTo`, `app:popUpToInclusive`, and `app:launchSingleTop` as behavior metadata, not as forward edges.

Call sites:

```kotlin
findNavController().navigate(R.id.action_home_to_detail, bundleOf("userId" to userId))
findNavController().navigate(HomeFragmentDirections.actionHomeToDetail(userId))
```

Safe Args directions carry the arguments in the generated method signature. Map them to the destination `<argument>` declarations.

## 4. Activities and Intents

Explicit:

```kotlin
startActivity(Intent(context, DetailActivity::class.java))
startActivity(Intent(context, DetailActivity::class.java).putExtra("itemId", itemId))
```

```kotlin
val launcher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result -> ... }
launcher.launch(Intent(context, PickerActivity::class.java))
```

Implicit:

```kotlin
startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))
startActivity(Intent.createChooser(intent, "Open with"))
```

Rules:

- Explicit `Intent(Context, Class)` is a direct edge to that Activity.
- `putExtra` keys become arguments when the target reads them with `get*Extra`.
- Implicit intents with a literal URI can resolve to a deep-link Activity. Otherwise use an `external` node and an `unknown` edge.
- `startActivityForResult` and `ActivityResultLauncher` are edges; the result is not a new edge unless it navigates somewhere.
- `finish()` and system back are not forward edges. Mention that back navigation is implicit.
- `NavDeepLinkBuilder`, `PendingIntent`, notifications, shortcuts, and widgets are entry edges into the target screen.

## 5. Compose Navigation 2

Host:

```kotlin
val navController = rememberNavController()

NavHost(navController = navController, startDestination = Home) {
    composable<Home> {
        HomeScreen(onDetail = { id -> navController.navigate(Detail(id)) })
    }
    composable<Detail> { backStackEntry ->
        val detail = backStackEntry.toRoute<Detail>()
        DetailScreen(detail)
    }
    navigation<CheckoutGraph>(startDestination = Cart) {
        composable<Cart> { CartScreen() }
        composable<Payment> { PaymentScreen() }
    }
}
```

String routes:

```kotlin
composable(
    route = "detail/{itemId}",
    arguments = listOf(navArgument("itemId") { type = NavType.StringType })
) { entry ->
    DetailScreen(entry.arguments?.getString("itemId"))
}
```

Extract:

- `NavHost(startDestination = ...)` as the graph entry.
- `composable<T>` and `composable("route")` as nodes.
- `navigation<T>(startDestination = ...)` as a nested `navGraph` node.
- `navigate(T(...))`, `navigate("detail/$id")`, and `navigate(route)` as edges.
- `popUpTo`, `popUpToInclusive`, `launchSingleTop`, and `restoreState` as behavior metadata.
- `toRoute<T>()` and `@Serializable` properties as arguments.
- `{placeholder}` segments in string routes as arguments.

Type-safe routes:

```kotlin
@Serializable
data class Detail(val itemId: String, val tab: String? = null)
```

The properties of the route class are the arguments. Optional properties have defaults; show the type and mark them optional when useful.

## 6. Compose Navigation 3

```kotlin
val backStack = rememberNavBackStack(Home)

NavDisplay(
    backStack = backStack,
    onBack = { backStack.removeLastOrNull() },
    entryProvider = entryProvider {
        entry<Home> {
            HomeScreen(onDetail = { id -> backStack.add(Detail(id)) })
        }
        entry<Detail> { key ->
            DetailScreen(key)
        }
    }
)
```

Extract:

- `rememberNavBackStack(...)` and `NavDisplay` as the host.
- `entry<T>` as nodes.
- `backStack.add(T(...))` as edges.
- `backStack.removeLastOrNull()` and `clear()` as back-stack behavior, not forward edges.
- `NavKey` data class properties as arguments.
- Scene strategies: `SinglePaneSceneStrategy`, `ListDetailSceneStrategy`, `DialogSceneStrategy`, and `BottomSheetSceneStrategy`. Dialog and bottom-sheet entries become `dialog` and `bottomSheet` nodes.
- `NavDisplay` scene keys and `entryDecorators` only when they affect visible destinations.

## 7. Hybrid apps

Activity or Fragment hosting Compose:

```kotlin
class HomeFragment : Fragment() {
    override fun onCreateView(...): View {
        return ComposeView(requireContext()).apply {
            setContent { HomeScreen() }
        }
    }
}
```

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent { AppNavHost() }
    }
}
```

Compose opening an Activity:

```kotlin
val context = LocalContext.current
Button(onClick = { context.startActivity(Intent(context, DetailActivity::class.java)) })
```

Extract:

- `NavHostFragment` or `FragmentContainerView` in a layout as an `include` edge from the Activity to the first Fragment.
- `ComposeView` and `setContent` as edges from the Fragment or Activity to the Compose screen.
- `startActivity` inside a composable as an `intent` edge to the Activity.
- Fragment transactions (`commit { replace(...) }`) as `navigate` edges when they change the visible screen.
- `AndroidView` hosting a `NavHostFragment` inside Compose is rare; treat it as a host bridge and record it.

## 8. Entry points

Include these as roots or dashed entry edges:

- Launcher Activity.
- Deep links from the manifest and `<deepLink>`.
- `PendingIntent` from notifications.
- App shortcuts (`res/xml/shortcuts.xml`).
- Widgets (`AppWidgetProvider`).
- `onNewIntent` handlers that navigate.

Mark these nodes with `"entry": true` in the graph so they remain visible as entry points even when the graph contains cycles.

## 9. Third-party libraries (best effort)

- **Voyager**: `Navigator`, `Screen`, `push`, `replace`, `pop`, `pushTab`.
- **Decompose**: `ChildStack`, `Component`, `Configuration`, `navigation`.
- **Compose Destinations**: `@Destination`, `DestinationsNavHost`, `navigate(SomeDestination(...))`, `@NavHostGraph`.

Detect the dependency, include what is statically resolvable, and state that support is best-effort. Do not claim full coverage.

## 10. Dynamic and unresolved navigation

Mark these as `unknown` or add them to `unresolved`:

- `navigate(routeVariable)` or `navigate(intent)` where the value is not a literal.
- `startActivity(intent)` with an intent built at runtime.
- Reflection (`Class.forName`), service locators, or plugin architectures.
- Server-driven or feature-flagged destinations.
- Implicit intents with no matching manifest filter.
- Destinations opened only from push notifications or external apps.

Keep the known part of the edge and append `?` when only part of the chain is known. Never invent a target.

## 11. Useful searches

```bash
rg -n "NavHost|composable<|composable\(|navigate\(|popUpTo|startDestination" --glob "*.kt"
rg -n "NavDisplay|rememberNavBackStack|entry<|backStack\.add|NavKey" --glob "*.kt"
rg -n "startActivity|Intent\(|putExtra|getStringExtra|registerForActivityResult" --glob "*.kt"
rg -n "<navigation|<fragment|<activity|<action|<argument|deepLink" --glob "*.xml"
rg -n "NavHostFragment|FragmentContainerView|ComposeView|setContent" --glob "*.{kt,xml}"
rg -n "hiltViewModel|viewModel\(|by viewModels|koinViewModel|ViewModelProvider" --glob "*.kt"
rg -n "@GET|@POST|@PUT|@DELETE|@Query|@Path|client\.(get|post)|@Dao" --glob "*.kt"
```

On Windows, `rg` works in PowerShell. If it is unavailable, use `Select-String` with the same patterns.

## 12. Edge cases

- **Multiple hosts**: record each host and its start destination. The same screen can appear in more than one graph; keep one node and add edges from each host.
- **Nested graphs**: render them as `navGraph` nodes or as subgraphs. Do not flatten away the graph boundary when it explains the flow.
- **Same screen, different arguments**: keep one node and separate edges. The arguments belong to the edge, not only to the node.
- **Conditional UI**: `if` or `when` branches that swap composables without a navigation call are not navigation edges. Mention them only if they change the visible destination.
- **Dialogs and bottom sheets**: include them as nodes when they are destinations or scene strategies. A simple `AlertDialog` with no navigation is not a destination.
- **Deep links**: show the entry edge and the target screen. Do not add URI parameters unless the user asked for deep-link arguments.
- **Back navigation**: system back is implicit. Do not draw a back edge for every forward edge; note it in the summary.
- **Duplicate activities**: `activity-alias` entries point to the same implementation. Merge them into one node and keep the alias in the label.
- **Generated code**: ignore `build/` and generated sources. Use annotations and source declarations instead.
- **Tests and samples**: skip `test`, `androidTest`, `samples`, and `benchmark` modules unless the user asks for them.
