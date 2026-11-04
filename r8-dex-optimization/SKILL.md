---
name: r8-dex-optimization
description: Diagnoses and fixes DEX code optimization and low obfuscation warnings (<25%) in Android apps for Google Play Console. Use this skill to enable R8 minification (minifyEnabled true, shrinkResources true) and configure essential ProGuard/R8 keep rules for Retrofit, Room, Hilt, Dagger, Coroutines, Gson, Moshi, Glide, and Crashlytics across any Android project.
metadata:
   author: Rolando Selvera
   keywords:
      - R8
      - proguard
      - minifyEnabled
      - obfuscation
      - google play console
      - dex optimization
      - retrofit
      - room
      - hilt
      - dagger
---

## Overview
This skill fixes the Google Play Console issue: **"DEX code optimization is below our threshold"** / **"Obfuscation (<25%)"**. It configures R8 minification and sets up universal, production-ready ProGuard keep rules for all standard Android architecture components and third-party libraries (Retrofit, Room, Hilt, Dagger, Coroutines, Gson, Moshi, Glide, etc.).

## Step 1: Analyze Project Stack & Package Structure
1. Inspect `app/build.gradle` or `app/build.gradle.kts` and `libs.versions.toml` (if present) to detect the project dependencies:
   - **Networking**: Retrofit, OkHttp, Volley
   - **Database**: Room, SQLite, Realm
   - **Dependency Injection**: Hilt, Dagger, Koin
   - **JSON Serialization**: Gson, Moshi, Kotlinx Serialization
   - **Async**: Coroutines, RxJava
   - **UI & Images**: ViewBinding, DataBinding, Glide, Coil
2. Identify the base package name (e.g. `com.example.app`).

## Step 2: Enable R8 Minification and Resource Shrinking

### Groovy (`app/build.gradle`)
```groovy
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            ...
        }
    }
}
```

### Kotlin DSL (`app/build.gradle.kts`)
```kotlin
android {
    ...
    buildTypes {
        getByName("release") {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            ...
        }
    }
}
```

## Step 3: Configure Universal ProGuard Keep Rules (`proguard-rules.pro`)

Add or merge the following rules into `app/proguard-rules.pro`:

```proguard
# 1. GENERAL & CRASH REPORTING (Firebase Crashlytics / Stack Traces)
-keepattributes SourceFile,LineNumberTable
-renamesourcefileattribute SourceFile
-keepattributes *Annotation*,Signature,InnerClasses,EnclosingMethod,RuntimeVisibleAnnotations,RuntimeVisibleParameterAnnotations

# 2. DATA MODELS, DTOs, ENTITIES & NETWORK REQUESTS/RESPONSES
# Prevent R8 from obfuscating fields/classes parsed via JSON, reflection, or DB
-keep class **.model.** { *; }
-keep class **.models.** { *; }
-keep class **.Modelos.** { *; }
-keep class **.dto.** { *; }
-keep class **.entity.** { *; }
-keep class **.entities.** { *; }
-keep class **.data.local.** { *; }
-keep class **.data.remote.** { *; }
-keep class **.response.** { *; }
-keep class **.request.** { *; }

# 3. RETROFIT & OKHTTP
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }
-keepclassmembers,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
-dontwarn okhttp3.**
-dontwarn okio.**
-keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase

# 4. ROOM & SQLITE
-keep class * extends androidx.room.RoomDatabase
-dontwarn androidx.room.paging.**
-keep class **.sqlite.** { *; }
-keep class **.Sqlite.** { *; }

# 5. HILT & DAGGER
-keep class * extends dagger.hilt.android.internal.managers.ViewComponentManager$FragmentContextWrapper
-keepclassmembers,allowobfuscation class * {
    @dagger.hilt.EntryPoint *;
    @dagger.hilt.InstallIn *;
}
-dontwarn com.google.errorprone.annotations.**

# 6. KOTLIN COROUTINES & FLOW
-dontwarn kotlinx.coroutines.**
-keepclassmembers class kotlinx.coroutines.** {
    volatile <fields>;
}

# 7. JSON PARSERS (GSON / MOSHI / KOTLINX SERIALIZATION)
# Gson
-keepclassmembers class * {
    @com.google.gson.annotations.SerializedName <fields>;
    @com.google.gson.annotations.Expose <fields>;
}
-keep class com.google.gson.reflect.TypeToken { *; }
-keep class * extends com.google.gson.reflect.TypeToken

# Moshi
-keepclassmembers class * {
    @com.squareup.moshi.Json *;
    @com.squareup.moshi.FromJson *;
    @com.squareup.moshi.ToJson *;
}

# 8. IMAGE LOADING (GLIDE / COIL)
# Glide
-keep public class * extends com.bumptech.glide.module.AppGlideModule
-keep class * implements com.bumptech.glide.module.GlideModule

# 9. DATABINDING & VIEWBINDING
-keepclassmembers class * extends androidx.databinding.ViewDataBinding {
    public <init>(...);
}
```

## Step 4: Verification
1. Run `./gradlew assembleRelease` or `./gradlew bundleRelease`.
2. Inspect the build output for any unresolved ProGuard/R8 warnings.
3. Test key app flows (Network requests, DB reads/writes, DI injection) on the release APK/AAB build.
