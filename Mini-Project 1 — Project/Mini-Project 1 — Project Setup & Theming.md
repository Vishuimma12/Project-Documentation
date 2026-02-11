# 🏗️ Mini-Project 1 — Project Setup & Theming

> **Objective:** Set up the Taskora Android project from scratch, configure Gradle, add all dependencies, and create a Material 3 theme with Jetpack Compose.

---

## 📖 Table of Contents

1. [Android Build System (Gradle)](#1-android-build-system-gradle)
2. [Settings.gradle.kts](#2-settingsgradlekts)
3. [Project-level build.gradle.kts](#3-project-level-buildgradlekts)
4. [Version Catalog (libs.versions.toml)](#4-version-catalog-libsversionstoml)
5. [App-level build.gradle.kts](#5-app-level-buildgradlekts)
6. [gradle.properties](#6-gradleproperties)
7. [Application Class & Hilt Setup](#7-application-class--hilt-setup)
8. [AndroidManifest.xml](#8-androidmanifestxml)
9. [Material 3 Theming — Colors](#9-material-3-theming--colors)
10. [Material 3 Theming — Typography](#10-material-3-theming--typography)
11. [Material 3 Theming — Theme.kt](#11-material-3-theming--themekt)
12. [MainActivity Setup](#12-mainactivity-setup)
13. [Checkpoint & Verification](#13-checkpoint--verification)
14. [Interview Questions & Answers](#14-interview-questions--answers)

---

## 1. Android Build System (Gradle)

### What is Gradle?

Gradle is the **build automation tool** Android uses. It:
- Downloads libraries (dependencies) from the internet
- Compiles your Kotlin/Java code
- Packages everything into an `.apk` or `.aab` file
- Manages multiple modules (app, libraries, etc.)

### Two Levels of Gradle Files

| File                                 | Scope           | Purpose                                    |
| ------------------------------------ | --------------- | ------------------------------------------ |
| **Project-level** `build.gradle.kts` | Entire project  | Declares plugins, global config            |
| **App-level** `app/build.gradle.kts` | App module only | Dependencies, SDK versions, build features |

### `.kts` vs `.gradle`

- `.gradle` = uses **Groovy** syntax (older)
- `.gradle.kts` = uses **Kotlin DSL** (modern, type-safe, autocomplete works)
- **Always use `.kts`** for new projects

---

## 2. Settings.gradle.kts

This is the **first file Gradle reads**. It defines which modules exist and where to download plugins/dependencies from.

```kotlin
// settings.gradle.kts (Project Root)

pluginManagement {
    repositories {
        google()            // Google's Maven repo (Android, Compose, etc.)
        mavenCentral()      // Open-source libraries
        gradlePluginPortal() // Gradle plugins
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "Taskora"
include(":app")
```

### Key Concepts

- **`pluginManagement`** — tells Gradle WHERE to find plugins (like the Android plugin, Kotlin plugin)
- **`dependencyResolutionManagement`** — tells Gradle WHERE to find libraries
- **`FAIL_ON_PROJECT_REPOS`** — forces all repos to be declared HERE, not in individual modules (cleaner)
- **`include(":app")`** — registers the `app` module. Multi-module projects would list more modules here

---

## 3. Project-level build.gradle.kts

Declares **plugins** used across the project. Use `apply false` because they're applied at the module level.

```kotlin
// build.gradle.kts (Project Root)

plugins {
    // Android Application plugin — needed to build Android apps
    id("com.android.application") version "8.2.2" apply false

    // Kotlin Android plugin — enables Kotlin in Android
    id("org.jetbrains.kotlin.android") version "1.9.22" apply false

    // Hilt plugin — Dependency Injection framework
    id("com.google.dagger.hilt.android") version "2.50" apply false

    // KSP — Kotlin Symbol Processing (replaces kapt, faster)
    id("com.google.devtools.ksp") version "1.9.22-1.0.17" apply false
}
```

### Why `apply false`?

Plugins are **declared** at project-level but **applied** in the module that needs them. This avoids applying the Android plugin to non-Android modules.

---

## 4. Version Catalog (libs.versions.toml)

The **modern way** to manage dependency versions in one central place. Lives at `gradle/libs.versions.toml`.

```toml
# gradle/libs.versions.toml

[versions]
agp = "8.2.2"
kotlin = "1.9.22"
ksp = "1.9.22-1.0.17"
compose-bom = "2024.02.00"
compose-compiler = "1.5.10"
hilt = "2.50"
room = "2.6.1"
retrofit = "2.9.0"
okhttp = "4.12.0"
coroutines = "1.7.3"
lifecycle = "2.7.0"
navigation = "2.7.7"
datastore = "1.0.0"
work = "2.9.0"

[libraries]
# Compose
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
activity-compose = { group = "androidx.activity", name = "activity-compose", version = "1.8.2" }

# Navigation
navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigation" }

# Room
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version = "1.1.0" }

# Retrofit
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }

# Lifecycle
lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle" }
lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycle" }

# DataStore
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }

# WorkManager
work-runtime = { group = "androidx.work", name = "work-runtime-ktx", version.ref = "work" }

# Coroutines
coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "coroutines" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

### Why Version Catalogs?

- **Single source of truth** — change a version once, applies everywhere
- **Type-safe** — `libs.compose.material3` gives autocomplete
- **Shareable** — in multi-module projects, every module uses the same versions

---

## 5. App-level build.gradle.kts

This is where you configure YOUR app module.

```kotlin
// app/build.gradle.kts

plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
}

android {
    namespace = "com.yourname.taskora"
    compileSdk = 34   // API level used to COMPILE your code

    defaultConfig {
        applicationId = "com.yourname.taskora"  // Unique app ID on Play Store
        minSdk = 26         // Minimum Android version supported (Android 8.0)
        targetSdk = 34      // API level you've tested against
        versionCode = 1     // Internal version (integer, increments)
        versionName = "1.0" // User-facing version string

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = true   // Enables R8 code shrinking
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        compose = true       // ENABLE Jetpack Compose
    }

    composeOptions {
        kotlinCompilerExtensionVersion = libs.versions.compose.compiler.get()
    }
}

dependencies {
    // Compose BOM — manages all Compose library versions together
    val composeBom = platform(libs.compose.bom)
    implementation(composeBom)

    implementation(libs.compose.ui)
    implementation(libs.compose.material3)
    implementation(libs.compose.ui.tooling.preview)
    debugImplementation(libs.compose.ui.tooling)
    implementation(libs.activity.compose)

    // Navigation
    implementation(libs.navigation.compose)

    // Room Database
    implementation(libs.room.runtime)
    implementation(libs.room.ktx)
    ksp(libs.room.compiler)    // KSP for annotation processing

    // Hilt Dependency Injection
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    // Retrofit + OkHttp
    implementation(libs.retrofit)
    implementation(libs.retrofit.gson)
    implementation(libs.okhttp)
    implementation(libs.okhttp.logging)

    // Lifecycle
    implementation(libs.lifecycle.viewmodel.compose)
    implementation(libs.lifecycle.runtime.compose)

    // DataStore
    implementation(libs.datastore.preferences)

    // WorkManager
    implementation(libs.work.runtime)

    // Coroutines
    implementation(libs.coroutines.android)
}
```

### SDK Versions Explained

| Property     | Meaning                                                                          | Example          |
| ------------ | -------------------------------------------------------------------------------- | ---------------- |
| `compileSdk` | The API level used to **compile** your code. You can use APIs up to this level.  | 34 (Android 14)  |
| `minSdk`     | The **lowest** Android version your app can install on.                          | 26 (Android 8.0) |
| `targetSdk`  | The API level you've **tested** against. Behavior changes above this are opt-in. | 34               |

### `implementation` vs `ksp` vs `debugImplementation`

| Keyword               | When Used                                                       |
| --------------------- | --------------------------------------------------------------- |
| `implementation`      | Runtime dependency — included in your app                       |
| `ksp`                 | Compile-time only — generates code (Room, Hilt)                 |
| `debugImplementation` | Only in debug builds (UI tools, Compose preview)                |
| `api`                 | Like `implementation`, but exposes the library to other modules |

### What is Compose BOM?

**BOM = Bill of Materials.** It's a single version that locks all Compose libraries to compatible versions. Instead of managing `compose-ui:1.6.1`, `compose-material3:1.2.0` separately, BOM ensures they all work together.

---

## 6. gradle.properties

Global config for your Gradle build.

```properties
# gradle.properties

# AndroidX Migration
android.useAndroidX=true

# Enable Jetifier (converts old Support Library to AndroidX)
android.enableJetifier=true

# Kotlin code style
kotlin.code.style=official

# Faster builds — parallel execution & daemon
org.gradle.parallel=true
org.gradle.daemon=true
org.gradle.caching=true

# Increase JVM memory for large projects
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8

# Non-transitive R classes (recommended for performance)
android.nonTransitiveRClass=true
```

### Key Properties Explained

| Property                      | What It Does                                                  |
| ----------------------------- | ------------------------------------------------------------- |
| `android.useAndroidX`         | Uses modern AndroidX libraries instead of old Support Library |
| `android.enableJetifier`      | Auto-converts third-party libs that still use Support Library |
| `org.gradle.parallel`         | Builds independent modules in parallel (faster)               |
| `org.gradle.caching`          | Reuses outputs from previous builds (faster)                  |
| `android.nonTransitiveRClass` | Each module only sees its OWN resources (prevents conflicts)  |

---

## 7. Application Class & Hilt Setup

### What is the Application Class?

The `Application` class is the **first thing created** when your app launches — even before any Activity. It lives for the entire lifetime of the app.

**Use cases:**
- Initialize libraries (Hilt, Analytics, Crash reporting)
- Hold app-wide state
- Create notification channels

```kotlin
// TaskoraApplication.kt
package com.yourname.taskora

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp   // ← This single annotation sets up ALL of Hilt
class TaskoraApplication : Application() {

    override fun onCreate() {
        super.onCreate()
        // Initialize anything app-wide here
        // Example: createNotificationChannels()
    }
}
```

### What Does `@HiltAndroidApp` Do?

1. Triggers Hilt's **code generation** at compile time
2. Creates a **dependency container** (component) at the Application level
3. Makes dependency injection available throughout the entire app
4. It's the **root** of Hilt's component hierarchy:

```
Application (@HiltAndroidApp)
  └── Activity (@AndroidEntryPoint)
       └── Fragment (@AndroidEntryPoint)
            └── ViewModel (@HiltViewModel)
```

---

## 8. AndroidManifest.xml

The manifest tells the Android system **everything about your app**.

```xml
<!-- AndroidManifest.xml -->
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".TaskoraApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Taskora"
        tools:targetApi="34">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.Taskora">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
</manifest>
```

### Critical Points

| Attribute                               | Purpose                                         |
| --------------------------------------- | ----------------------------------------------- |
| `android:name=".TaskoraApplication"`    | Registers YOUR Application class (with Hilt)    |
| `android:exported="true"`               | Required on the launcher Activity (Android 12+) |
| `<action MAIN>` + `<category LAUNCHER>` | Makes this Activity the app's entry point       |

---

## 9. Material 3 Theming — Colors

### Concept: Material 3 Color System

Material 3 uses **color roles** instead of individual colors. Each role has a purpose:

| Role        | Purpose                        | Example                   |
| ----------- | ------------------------------ | ------------------------- |
| `primary`   | Main brand color, key actions  | FAB, buttons              |
| `onPrimary` | Text/icons ON primary          | White text on blue button |
| `secondary` | Less prominent elements        | Filter chips              |
| `surface`   | Background of cards, sheets    | Card backgrounds          |
| `error`     | Errors and destructive actions | Red for delete            |

```kotlin
// presentation/theme/Color.kt
package com.yourname.taskora.presentation.theme

import androidx.compose.ui.graphics.Color

// ===== LIGHT THEME PALETTE =====
val PrimaryLight = Color(0xFF1A73E8)         // Google Blue
val OnPrimaryLight = Color(0xFFFFFFFF)       // White text on blue
val PrimaryContainerLight = Color(0xFFD3E3FD) // Lighter blue for containers
val OnPrimaryContainerLight = Color(0xFF041E49)

val SecondaryLight = Color(0xFF5F6368)
val OnSecondaryLight = Color(0xFFFFFFFF)

val TertiaryLight = Color(0xFF0D652D)        // Green for success/completion
val OnTertiaryLight = Color(0xFFFFFFFF)

val BackgroundLight = Color(0xFFF8F9FA)
val OnBackgroundLight = Color(0xFF1F1F1F)

val SurfaceLight = Color(0xFFFFFFFF)
val OnSurfaceLight = Color(0xFF1F1F1F)

val ErrorLight = Color(0xFFB3261E)
val OnErrorLight = Color(0xFFFFFFFF)

// ===== DARK THEME PALETTE =====
val PrimaryDark = Color(0xFFA8C7FA)
val OnPrimaryDark = Color(0xFF062E6F)
val PrimaryContainerDark = Color(0xFF0842A0)
val OnPrimaryContainerDark = Color(0xFFD3E3FD)

val SecondaryDark = Color(0xFFC4C7C5)
val OnSecondaryDark = Color(0xFF2D312F)

val TertiaryDark = Color(0xFF6DD58C)
val OnTertiaryDark = Color(0xFF0A3818)

val BackgroundDark = Color(0xFF1F1F1F)
val OnBackgroundDark = Color(0xFFE3E3E3)

val SurfaceDark = Color(0xFF2D2D2D)
val OnSurfaceDark = Color(0xFFE3E3E3)

val ErrorDark = Color(0xFFF2B8B5)
val OnErrorDark = Color(0xFF601410)

// ===== CUSTOM PRIORITY COLORS =====
val PriorityUrgent = Color(0xFFD93025)
val PriorityHigh = Color(0xFFFA903E)
val PriorityMedium = Color(0xFFFBBC04)
val PriorityLow = Color(0xFF34A853)
```

### How `Color(0xFF...)` Works

```
Color(0xFF1A73E8)
        ││││││││
        ││└┴┴┴┴┴── Hex color: 1A73E8 (R=1A, G=73, B=E8)
        └┴── Alpha: FF = fully opaque (00 = transparent)
```

---

## 10. Material 3 Theming — Typography

```kotlin
// presentation/theme/Type.kt
package com.yourname.taskora.presentation.theme

import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.Font
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp
import com.yourname.taskora.R

// Step 1: Define your font family
// Download Inter font files, place in res/font/
val InterFontFamily = FontFamily(
    Font(R.font.inter_regular, FontWeight.Normal),
    Font(R.font.inter_medium, FontWeight.Medium),
    Font(R.font.inter_semibold, FontWeight.SemiBold),
    Font(R.font.inter_bold, FontWeight.Bold)
)

// Step 2: Create the Typography
val TaskoraTypography = Typography(

    // Large titles (splash screen, headers)
    displayLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 57.sp,
        lineHeight = 64.sp
    ),

    // Screen titles
    headlineLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 32.sp,
        lineHeight = 40.sp
    ),

    headlineMedium = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.SemiBold,
        fontSize = 28.sp,
        lineHeight = 36.sp
    ),

    // Section titles, card titles
    titleLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.SemiBold,
        fontSize = 22.sp,
        lineHeight = 28.sp
    ),

    titleMedium = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 16.sp,
        lineHeight = 24.sp
    ),

    // Body text (descriptions, content)
    bodyLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp
    ),

    bodyMedium = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp
    ),

    // Labels, chips, buttons
    labelLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp
    ),

    labelSmall = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 11.sp,
        lineHeight = 16.sp
    )
)
```

### Font Loading Steps
1. Download font files (`.ttf`) from [Google Fonts](https://fonts.google.com/specimen/Inter)
2. Create folder: `app/src/main/res/font/`
3. Place files: `inter_regular.ttf`, `inter_medium.ttf`, `inter_semibold.ttf`, `inter_bold.ttf`
4. Reference in code as `R.font.inter_regular`

---

## 11. Material 3 Theming — Theme.kt

This is where everything comes together.

```kotlin
// presentation/theme/Theme.kt
package com.yourname.taskora.presentation.theme

import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.platform.LocalContext

// Light color scheme using our custom colors
private val LightColorScheme = lightColorScheme(
    primary = PrimaryLight,
    onPrimary = OnPrimaryLight,
    primaryContainer = PrimaryContainerLight,
    onPrimaryContainer = OnPrimaryContainerLight,
    secondary = SecondaryLight,
    onSecondary = OnSecondaryLight,
    tertiary = TertiaryLight,
    onTertiary = OnTertiaryLight,
    background = BackgroundLight,
    onBackground = OnBackgroundLight,
    surface = SurfaceLight,
    onSurface = OnSurfaceLight,
    error = ErrorLight,
    onError = OnErrorLight
)

// Dark color scheme
private val DarkColorScheme = darkColorScheme(
    primary = PrimaryDark,
    onPrimary = OnPrimaryDark,
    primaryContainer = PrimaryContainerDark,
    onPrimaryContainer = OnPrimaryContainerDark,
    secondary = SecondaryDark,
    onSecondary = OnSecondaryDark,
    tertiary = TertiaryDark,
    onTertiary = OnTertiaryDark,
    background = BackgroundDark,
    onBackground = OnBackgroundDark,
    surface = SurfaceDark,
    onSurface = OnSurfaceDark,
    error = ErrorDark,
    onError = OnErrorDark
)

@Composable
fun TaskoraTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),  // Auto-detect system theme
    dynamicColor: Boolean = true,                 // Android 12+ dynamic theming
    content: @Composable () -> Unit               // Everything inside the theme
) {
    // Choose color scheme
    val colorScheme = when {
        // Dynamic colors from wallpaper (Android 12+)
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    // Apply theme to entire app
    MaterialTheme(
        colorScheme = colorScheme,
        typography = TaskoraTypography,
        content = content
    )
}
```

### How the Theme Wraps Your App

```
TaskoraTheme {                    ← Sets colors + fonts globally
    MaterialTheme.colorScheme     ← Any composable can access current colors
    MaterialTheme.typography      ← Any composable can access current fonts

    Scaffold {                    ← Uses theme colors automatically
        Button { ... }            ← Automatically uses primary color
        Text { ... }              ← Automatically uses onSurface color
    }
}
```

### Using Theme Colors in Any Composable

```kotlin
// Anywhere inside TaskoraTheme:
Text(
    text = "Hello",
    color = MaterialTheme.colorScheme.primary,      // Uses theme primary
    style = MaterialTheme.typography.headlineMedium  // Uses theme font
)
```

---

## 12. MainActivity Setup

```kotlin
// MainActivity.kt
package com.yourname.taskora

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import com.yourname.taskora.presentation.theme.TaskoraTheme
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint   // ← Required for Hilt to inject into this Activity
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()  // Content draws behind system bars

        setContent {
            TaskoraTheme {   // ← Your theme wraps EVERYTHING
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    // Placeholder — you'll replace this with NavGraph later
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        Text(
                            text = "Taskora 🚀",
                            style = MaterialTheme.typography.headlineLarge
                        )
                    }
                }
            }
        }
    }
}
```

---

## 13. Checkpoint & Verification

### Build & Run

1. **Sync Gradle**: File → Sync Project with Gradle Files
2. **Build**: Build → Rebuild Project (fix any errors)
3. **Run**: Click ▶️ on emulator or physical device

### Verify These Work

| Check               | How to Test                               |
| ------------------- | ----------------------------------------- |
| ✅ Project compiles  | Build succeeds without errors             |
| ✅ Theme applied     | You see "Taskora 🚀" with your custom font |
| ✅ Dark mode         | Toggle device dark mode → colors change   |
| ✅ Application class | No crash on launch (Hilt is initialized)  |
| ✅ Dependencies      | No "Unresolved reference" errors          |

### Common Errors & Fixes

| Error                                 | Fix                                                     |
| ------------------------------------- | ------------------------------------------------------- |
| `Unresolved reference: hiltViewModel` | Add `hilt-navigation-compose` dependency                |
| `Cannot find symbol HiltAndroidApp`   | Run Build → Rebuild (Hilt generates code)               |
| `Compose compiler incompatible`       | Match `compose-compiler` version to your Kotlin version |
| `minSdk too low for desugaring`       | Set `minSdk = 26` or enable core library desugaring     |

---

## 14. Interview Questions & Answers

### Q1: What is the difference between `compileSdk`, `minSdk`, and `targetSdk`?

**Answer:**
- **`compileSdk`** — The Android API level used to compile your code. You can reference APIs up to this level. It does NOT affect which devices can run your app.
- **`minSdk`** — The **minimum** Android version your app supports. Devices below this CANNOT install the app. Lower = more devices, but older APIs.
- **`targetSdk`** — The API level you've **tested** against. The system enables behavior changes up to this level. Above this, the system applies backward compatibility.

**Example:** `compileSdk = 34, minSdk = 26, targetSdk = 34` means you compile against Android 14 APIs, support devices from Android 8.0+, and have opted into all behavior changes up to Android 14.

---

### Q2: What is Dependency Injection and why do we use Hilt?

**Answer:**
**Dependency Injection (DI)** is a pattern where objects receive their dependencies from the **outside** rather than creating them internally.

**Without DI (bad):**
```kotlin
class HomeViewModel {
    // ❌ Creates its own dependency — tightly coupled, untestable
    private val repository = TaskRepositoryImpl(TaskDatabase.getInstance().taskDao())
}
```

**With DI (good):**
```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    // ✅ Dependencies provided from outside — loosely coupled, testable
    private val repository: TaskRepository
) : ViewModel()
```

**Why Hilt:** It's Google's recommended DI framework for Android. It eliminates boilerplate, integrates with Android components (Activity, ViewModel), and uses compile-time code generation (faster than reflection).

---

### Q3: What does `@HiltAndroidApp` do?

**Answer:**
It triggers Hilt's code generation at the Application level. Specifically:
1. Generates a `Hilt_TaskoraApplication` base class
2. Creates the **root component** (`SingletonComponent`) that holds all app-wide dependencies
3. Initializes the entire DI graph when the app starts
4. Makes `@AndroidEntryPoint`, `@HiltViewModel`, `@Inject` work throughout the app

Without it, none of Hilt's features will work.

---

### Q4: Explain Compose BOM. Why is it important?

**Answer:**
**BOM (Bill of Materials)** is a single version number that manages all Jetpack Compose library versions.

```kotlin
val composeBom = platform(libs.compose.bom)  // e.g., "2024.02.00"
implementation(composeBom)
implementation("androidx.compose.ui:ui")     // No version needed!
implementation("androidx.compose.material3:material3")  // Automatically compatible
```

**Why important:** Compose has 10+ libraries that must be version-compatible. BOM guarantees all libraries work together, preventing `NoSuchMethodError` or `ClassNotFoundException` at runtime.

---

### Q5: What is `setContent {}` in a Compose Activity?

**Answer:**
`setContent {}` replaces the old `setContentView(R.layout.activity_main)`. It sets a **Composable function** as the Activity's UI content. Inside it, you write your entire UI using Compose functions. It creates a `ComposeView`, attaches it to the Activity's window, and starts the Compose rendering pipeline.

---

### Q6: Why separate `Color.kt`, `Type.kt`, and `Theme.kt`?

**Answer:**
This follows the **Single Responsibility Principle**:
- `Color.kt` — Only defines color values (easy to swap palettes)
- `Type.kt` — Only defines typography (easy to change fonts)
- `Theme.kt` — Assembles colors + typography into a theme

Benefits: If a designer changes the color palette, you edit ONE file. If you switch fonts, you edit ONE file. The theme automatically reflects changes.

---

### Q7: What is `isSystemInDarkTheme()` and how does dynamic theming work?

**Answer:**
- **`isSystemInDarkTheme()`** — A Compose function that reads the device's current dark mode setting and returns `true` or `false`. It's **reactive** — if the user toggles dark mode, the app recomposes with the new theme.
- **Dynamic theming** (Android 12+) — Uses `dynamicLightColorScheme()` / `dynamicDarkColorScheme()` to extract colors from the user's **wallpaper** and apply them to your app. The app's theme matches the device's personality.

---

### Q8: What is `enableEdgeToEdge()` and why use it?

**Answer:**
`enableEdgeToEdge()` makes your app draw behind the system bars (status bar, navigation bar). Instead of white bars with your content below, your content extends to the screen edges for a more immersive, modern look. You then use `WindowInsets` APIs in Compose to add padding where needed so content doesn't go under the system bars.

---

### Q9: What is KSP and how is it different from KAPT?

**Answer:**
Both are **annotation processing** tools (they read annotations like `@Entity`, `@Inject` and generate code).

| Feature        | KAPT                                 | KSP                            |
| -------------- | ------------------------------------ | ------------------------------ |
| How it works   | Generates Java stubs, then processes | Directly processes Kotlin code |
| Speed          | **Slower** (needs stub generation)   | **2x faster**                  |
| Kotlin support | Indirect (through Java stubs)        | Native Kotlin understanding    |
| Status         | Legacy, maintenance mode             | Modern, recommended            |

**Use KSP** for Room and Hilt. KAPT is being phased out.

---

### Q10: What is the Application class lifecycle?

**Answer:**
```
App Process Created
    └── Application.onCreate()      ← FIRST callback, called ONCE
         └── Activity.onCreate()
              └── Activity.onStart()
                   └── Activity.onResume()
                        └── (User interacts)
                   └── Activity.onPause()
              └── Activity.onStop()
         └── Activity.onDestroy()
    └── (More activities come and go...)
└── App Process Killed              ← Application.onTerminate() (NOT guaranteed)
```

The Application class lives for the **entire process lifetime**. It's created before any Activity and destroyed only when the OS kills the process. That's why it's the right place for one-time initialization.

---

> **✅ Mini-Project 1 Complete!** You now have a compilable Android project with proper Gradle config, all dependencies, and a beautiful Material 3 theme. Move on to **Mini-Project 2: Domain Models** next.
