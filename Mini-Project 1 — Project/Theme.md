# 🎨 Material 3 Theme.kt — Complete Setup & Configuration Guide (Jetpack Compose)

> **Goal:** Understand every part of the `Theme.kt` file — how color schemes are built, how dynamic theming works, how contrast levels are handled, and how `AppTheme` ties it all together.
>
> **Audience:** Android developers using Jetpack Compose + Material 3.
>
> **Prerequisite:** [Material3 Color Roles Guide](./Material3_Color_Roles_Guide.md) — understand what each color role means before diving into theme setup.

---

## Table of Contents

1. [What Is Theme.kt?](#what-is-themekt)
2. [File Structure Overview](#file-structure-overview)
3. [Package & Imports Breakdown](#package--imports-breakdown)
4. [Light & Dark Color Schemes](#light--dark-color-schemes)
5. [Medium & High Contrast Schemes](#medium--high-contrast-schemes)
6. [The ColorFamily Data Class](#the-colorfamily-data-class)
7. [The AppTheme Composable — Line by Line](#the-apptheme-composable--line-by-line)
8. [Dynamic Colors (Material You)](#dynamic-colors-material-you)
9. [How to Use AppTheme in Your App](#how-to-use-apptheme-in-your-app)
10. [Adding Contrast Level Support](#adding-contrast-level-support)
11. [Summary Table](#-summary-table)
12. [Interview Q&A](#-interview-qa)
13. [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)

---

## What Is Theme.kt?

`Theme.kt` is the **central theming configuration** for a Jetpack Compose Material 3 app. It:

- Defines **all color schemes** (light, dark, medium contrast, high contrast)
- Provides a **root composable** (`AppTheme`) that wraps your entire app
- Handles **dynamic color** support on Android 12+
- Applies colors and typography through `MaterialTheme`

> 💡 **Think of it as:** The "skin" of your app. Every M3 component reads its colors from the scheme configured here.

### Where Does Theme.kt Live?

```
app/
└── src/main/java/com/example/compose/
    ├── ui/theme/
    │   ├── Color.kt          ← All color values (hex → Color objects)
    │   ├── Theme.kt          ← THIS FILE (scheme assembly + AppTheme)
    │   └── Type.kt           ← Typography definitions
    └── MainActivity.kt       ← Entry point, wraps content in AppTheme
```

---

## File Structure Overview

Here's a **bird's-eye view** of what the Theme.kt file contains:

```
Theme.kt
│
├── Package Declaration
│   └── com.example.compose
│
├── Imports
│   └── Material 3, Compose UI, Android SDK
│
├── Color Schemes (6 total)
│   ├── lightScheme              ← Default light mode
│   ├── darkScheme               ← Default dark mode
│   ├── mediumContrastLightColorScheme  ← Accessibility: medium contrast light
│   ├── highContrastLightColorScheme    ← Accessibility: high contrast light
│   ├── mediumContrastDarkColorScheme   ← Accessibility: medium contrast dark
│   └── highContrastDarkColorScheme     ← Accessibility: high contrast dark
│
├── ColorFamily Data Class
│   └── Helper for grouping color + onColor + container + onContainer
│
├── unspecified_scheme
│   └── Default unspecified ColorFamily
│
└── AppTheme Composable
    └── Root theme function that selects the correct scheme
```

---

## Package & Imports Breakdown

```kotlin
package com.example.compose
```
> Your app's package name. Change this to match your project.

### Key Imports Explained

| Import                                                   | Purpose                                                    |
| -------------------------------------------------------- | ---------------------------------------------------------- |
| `android.app.Activity`                                   | Access to the Activity (used for status bar theming)       |
| `android.os.Build`                                       | Check Android API level (needed for dynamic colors)        |
| `isSystemInDarkTheme()`                                  | Detect if user has dark mode enabled in system settings    |
| `MaterialTheme`                                          | The M3 theme provider composable                           |
| `lightColorScheme()` / `darkColorScheme()`               | Factory functions to create color schemes                  |
| `dynamicDarkColorScheme()` / `dynamicLightColorScheme()` | Generate schemes from device wallpaper (Android 12+)       |
| `Typography`                                             | Type system for M3                                         |
| `@Composable`                                            | Marks composable functions                                 |
| `@Immutable`                                             | Marks data classes as immutable for Compose stability      |
| `Color`                                                  | Compose color class                                        |
| `toArgb()`                                               | Convert Color to ARGB int (for system UI like status bars) |
| `LocalContext`                                           | Get the current Android Context inside a composable        |

---

## Light & Dark Color Schemes

### What Happens Here

The file creates **two primary schemes** by mapping every M3 color role to a specific color value from `Color.kt`:

```kotlin
private val lightScheme = lightColorScheme(
    primary = primaryLight,
    onPrimary = onPrimaryLight,
    primaryContainer = primaryContainerLight,
    // ... 29 more color roles
)

private val darkScheme = darkColorScheme(
    primary = primaryDark,
    onPrimary = onPrimaryDark,
    primaryContainer = primaryContainerDark,
    // ... 29 more color roles
)
```

### The Complete Color Role Mapping (32 roles per scheme)

| #   | Color Role                | Light Value Variable           | Dark Value Variable           |
| --- | ------------------------- | ------------------------------ | ----------------------------- |
| 1   | `primary`                 | `primaryLight`                 | `primaryDark`                 |
| 2   | `onPrimary`               | `onPrimaryLight`               | `onPrimaryDark`               |
| 3   | `primaryContainer`        | `primaryContainerLight`        | `primaryContainerDark`        |
| 4   | `onPrimaryContainer`      | `onPrimaryContainerLight`      | `onPrimaryContainerDark`      |
| 5   | `secondary`               | `secondaryLight`               | `secondaryDark`               |
| 6   | `onSecondary`             | `onSecondaryLight`             | `onSecondaryDark`             |
| 7   | `secondaryContainer`      | `secondaryContainerLight`      | `secondaryContainerDark`      |
| 8   | `onSecondaryContainer`    | `onSecondaryContainerLight`    | `onSecondaryContainerDark`    |
| 9   | `tertiary`                | `tertiaryLight`                | `tertiaryDark`                |
| 10  | `onTertiary`              | `onTertiaryLight`              | `onTertiaryDark`              |
| 11  | `tertiaryContainer`       | `tertiaryContainerLight`       | `tertiaryContainerDark`       |
| 12  | `onTertiaryContainer`     | `onTertiaryContainerLight`     | `onTertiaryContainerDark`     |
| 13  | `error`                   | `errorLight`                   | `errorDark`                   |
| 14  | `onError`                 | `onErrorLight`                 | `onErrorDark`                 |
| 15  | `errorContainer`          | `errorContainerLight`          | `errorContainerDark`          |
| 16  | `onErrorContainer`        | `onErrorContainerLight`        | `onErrorContainerDark`        |
| 17  | `background`              | `backgroundLight`              | `backgroundDark`              |
| 18  | `onBackground`            | `onBackgroundLight`            | `onBackgroundDark`            |
| 19  | `surface`                 | `surfaceLight`                 | `surfaceDark`                 |
| 20  | `onSurface`               | `onSurfaceLight`               | `onSurfaceDark`               |
| 21  | `surfaceVariant`          | `surfaceVariantLight`          | `surfaceVariantDark`          |
| 22  | `onSurfaceVariant`        | `onSurfaceVariantLight`        | `onSurfaceVariantDark`        |
| 23  | `outline`                 | `outlineLight`                 | `outlineDark`                 |
| 24  | `outlineVariant`          | `outlineVariantLight`          | `outlineVariantDark`          |
| 25  | `scrim`                   | `scrimLight`                   | `scrimDark`                   |
| 26  | `inverseSurface`          | `inverseSurfaceLight`          | `inverseSurfaceDark`          |
| 27  | `inverseOnSurface`        | `inverseOnSurfaceLight`        | `inverseOnSurfaceDark`        |
| 28  | `inversePrimary`          | `inversePrimaryLight`          | `inversePrimaryDark`          |
| 29  | `surfaceDim`              | `surfaceDimLight`              | `surfaceDimDark`              |
| 30  | `surfaceBright`           | `surfaceBrightLight`           | `surfaceBrightDark`           |
| 31  | `surfaceContainerLowest`  | `surfaceContainerLowestLight`  | `surfaceContainerLowestDark`  |
| 32  | `surfaceContainerLow`     | `surfaceContainerLowLight`     | `surfaceContainerLowDark`     |
| 33  | `surfaceContainer`        | `surfaceContainerLight`        | `surfaceContainerDark`        |
| 34  | `surfaceContainerHigh`    | `surfaceContainerHighLight`    | `surfaceContainerHighDark`    |
| 35  | `surfaceContainerHighest` | `surfaceContainerHighestLight` | `surfaceContainerHighestDark` |

### Why `private val`?

- **`private`** → These schemes are internal to this file. Only `AppTheme` exposes them.
- **`val`** → Immutable. Color schemes don't change at runtime once selected.

### Where Do the Color Values Come From?

All values like `primaryLight`, `onPrimaryDark`, etc. are defined in the companion file **`Color.kt`**:

```kotlin
// Color.kt (generated by Material Theme Builder)
val primaryLight = Color(0xFF6750A4)
val onPrimaryLight = Color(0xFFFFFFFF)
val primaryContainerLight = Color(0xFFEADDFF)
// ... hundreds more
```

> 🛠️ **Tip:** These are typically auto-generated from the [Material Theme Builder](https://m3.material.io/theme-builder).

---

## Medium & High Contrast Schemes

The file defines **4 additional schemes** for accessibility:

```kotlin
private val mediumContrastLightColorScheme = lightColorScheme(
    primary = primaryLightMediumContrast,
    // ... same structure, different color values
)

private val highContrastLightColorScheme = lightColorScheme(
    primary = primaryLightHighContrast,
    // ...
)

private val mediumContrastDarkColorScheme = darkColorScheme(
    primary = primaryDarkMediumContrast,
    // ...
)

private val highContrastDarkColorScheme = darkColorScheme(
    primary = primaryDarkHighContrast,
    // ...
)
```

### Why 6 Schemes Total?

| #   | Scheme                           | When Used                                   |
| --- | -------------------------------- | ------------------------------------------- |
| 1   | `lightScheme`                    | Default light mode                          |
| 2   | `darkScheme`                     | Default dark mode                           |
| 3   | `mediumContrastLightColorScheme` | Light + user needs more contrast (WCAG AA+) |
| 4   | `highContrastLightColorScheme`   | Light + maximum contrast (WCAG AAA)         |
| 5   | `mediumContrastDarkColorScheme`  | Dark + user needs more contrast             |
| 6   | `highContrastDarkColorScheme`    | Dark + maximum contrast                     |

### Visual Relationship

```
                    ┌─────────────────┐
                    │   Normal Mode   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         ┌─────────┐   ┌──────────┐   ┌──────────┐
         │  Light  │   │  Medium  │   │   High   │
         │ Scheme  │   │ Contrast │   │ Contrast │
         └─────────┘   └──────────┘   └──────────┘
              
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         ┌─────────┐   ┌──────────┐   ┌──────────┐
         │  Dark   │   │  Medium  │   │   High   │
         │ Scheme  │   │ Contrast │   │ Contrast │
         └─────────┘   └──────────┘   └──────────┘
```

> ⚠️ **Note:** The current `AppTheme` composable in this file only switches between `lightScheme` and `darkScheme`. To use contrast variants, you need to add a contrast level parameter (see [Adding Contrast Level Support](#adding-contrast-level-support)).

---

## The ColorFamily Data Class

```kotlin
@Immutable
data class ColorFamily(
    val color: Color,
    val onColor: Color,
    val colorContainer: Color,
    val onColorContainer: Color
)

val unspecified_scheme = ColorFamily(
    Color.Unspecified, Color.Unspecified, Color.Unspecified, Color.Unspecified
)
```

### What Is This?

A **convenience wrapper** to group a complete color family (4 related colors) together.

### Why `@Immutable`?

- Tells Compose this class **never changes internally** after creation
- Enables Compose to **skip recomposition** when the reference hasn't changed
- Critical for **performance** — without it, Compose can't optimize properly

### How It Maps to M3 Color Groups

```
ColorFamily(
    color          = primary,           // The main color
    onColor        = onPrimary,         // Text/icon on the color
    colorContainer = primaryContainer,  // Soft container variant
    onColorContainer = onPrimaryContainer // Text/icon on container
)
```

This pattern applies to: **Primary**, **Secondary**, **Tertiary**, **Error**

### What Is `unspecified_scheme`?

A **default/placeholder** `ColorFamily` where all colors are `Color.Unspecified`:
- Used when you haven't assigned a custom color family yet
- `Color.Unspecified` tells Compose to **use the default** behavior
- Useful for optional custom color family parameters in composables

### Practical Use of ColorFamily

```kotlin
// Define a custom color family for a feature
val premiumColorFamily = ColorFamily(
    color = MaterialTheme.colorScheme.tertiary,
    onColor = MaterialTheme.colorScheme.onTertiary,
    colorContainer = MaterialTheme.colorScheme.tertiaryContainer,
    onColorContainer = MaterialTheme.colorScheme.onTertiaryContainer
)

// Pass it to a component
@Composable
fun FeatureCard(
    title: String,
    colorFamily: ColorFamily = unspecified_scheme
) {
    val containerColor = if (colorFamily.colorContainer != Color.Unspecified)
        colorFamily.colorContainer
    else
        MaterialTheme.colorScheme.surfaceContainer

    Surface(color = containerColor) {
        Text(text = title)
    }
}
```

---

## The AppTheme Composable — Line by Line

This is the **most important part** of the file — the root composable that wraps your app:

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable() () -> Unit
) {
```

### Parameters Explained

| Parameter      | Type                     | Default                 | Purpose                                             |
| -------------- | ------------------------ | ----------------------- | --------------------------------------------------- |
| `darkTheme`    | `Boolean`                | `isSystemInDarkTheme()` | Whether to use dark color scheme                    |
| `dynamicColor` | `Boolean`                | `true`                  | Whether to use wallpaper-based colors (Android 12+) |
| `content`      | `@Composable () -> Unit` | —                       | The app's UI content to wrap with theming           |

### The Color Scheme Selection Logic

```kotlin
val colorScheme = when {
    // STEP 1: Check if dynamic color is enabled AND device supports it
    dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
        val context = LocalContext.current
        if (darkTheme) dynamicDarkColorScheme(context)
        else dynamicLightColorScheme(context)
    }
    
    // STEP 2: If no dynamic color, use manual dark scheme
    darkTheme -> darkScheme
    
    // STEP 3: Otherwise, use manual light scheme
    else -> lightScheme
}
```

### Decision Flowchart

```
AppTheme called
       │
       ▼
  dynamicColor = true   ──── No ───→  darkTheme?
  AND API ≥ 31 (S)?                     │    │
       │                               Yes   No
      Yes                              │    │
       │                               ▼    ▼
       ▼                          darkScheme  lightScheme
  darkTheme?
    │     │
   Yes    No
    │     │
    ▼     ▼
  dynamicDarkColorScheme    dynamicLightColorScheme
  (from wallpaper)           (from wallpaper)
```

### Applying the Theme

```kotlin
MaterialTheme(
    colorScheme = colorScheme,    // Selected color palette
    typography = AppTypography,    // From Type.kt
    content = content              // Your app's UI
)
```

#### What `MaterialTheme` Does:

1. **Provides `colorScheme`** via `CompositionLocal` → All child composables can access `MaterialTheme.colorScheme.xxx`
2. **Provides `typography`** → All child composables can access `MaterialTheme.typography.xxx`
3. **Provides `shapes`** → (Not set here, uses defaults) `MaterialTheme.shapes.xxx`

---

## Dynamic Colors (Material You)

### What Are Dynamic Colors?

Starting with **Android 12 (API 31, "S")**, the system can extract colors from the user's **wallpaper** and generate a complete color scheme automatically. This is called **Material You**.

### How It Works in the Code

```kotlin
dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
    val context = LocalContext.current
    if (darkTheme) dynamicDarkColorScheme(context)
    else dynamicLightColorScheme(context)
}
```

| Function                           | What It Does                                    |
| ---------------------------------- | ----------------------------------------------- |
| `dynamicLightColorScheme(context)` | Creates a light M3 scheme from wallpaper colors |
| `dynamicDarkColorScheme(context)`  | Creates a dark M3 scheme from wallpaper colors  |

### Key Points

- **Requires `Context`** → Retrieved via `LocalContext.current`
- **Only works on API 31+** → The `Build.VERSION.SDK_INT >= Build.VERSION_CODES.S` guard prevents crashes on older devices
- **Falls back gracefully** → On older devices, your manually defined `lightScheme`/`darkScheme` are used
- **User-personalized** → Each user sees different colors based on their wallpaper

### When to Disable Dynamic Color

```kotlin
// Pass dynamicColor = false to always use your brand colors
AppTheme(dynamicColor = false) {
    // This app always uses the Color.kt palette, never wallpaper colors
}
```

**Disable when:**
- You have a **strong brand identity** that must be consistent (e.g., Spotify green)
- Your app uses **specific semantic colors** that would break with wallpaper-derived colors
- You want **consistent screenshots** for documentation/marketing

---

## How to Use AppTheme in Your App

### Basic Usage (MainActivity.kt)

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                // Your entire app goes here
                Scaffold { paddingValues ->
                    HomeScreen(modifier = Modifier.padding(paddingValues))
                }
            }
        }
    }
}
```

### With User-Controlled Dark Mode

```kotlin
@Composable
fun MyApp() {
    var isDarkMode by remember { mutableStateOf(false) }

    AppTheme(darkTheme = isDarkMode) {
        Scaffold(
            topBar = {
                TopAppBar(
                    title = { Text("My App") },
                    actions = {
                        Switch(
                            checked = isDarkMode,
                            onCheckedChange = { isDarkMode = it }
                        )
                    }
                )
            }
        ) { paddingValues ->
            // Content
        }
    }
}
```

### Overriding Theme for a Specific Section

```kotlin
@Composable
fun MainContent() {
    // This section uses the app's theme
    Text("Normal themed text", color = MaterialTheme.colorScheme.onSurface)

    // Force dark theme for a specific section
    AppTheme(darkTheme = true, dynamicColor = false) {
        Card {
            Text(
                "Always dark card",
                color = MaterialTheme.colorScheme.onSurface
            )
        }
    }
}
```

---

## Adding Contrast Level Support

The current `AppTheme` only uses 2 of the 6 defined schemes. Here's how to **extend it** to support all contrast levels:

### Step 1: Define a Contrast Level Enum

```kotlin
enum class ContrastLevel {
    Normal,
    Medium,
    High
}
```

### Step 2: Update AppTheme

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    contrastLevel: ContrastLevel = ContrastLevel.Normal,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }

        darkTheme -> when (contrastLevel) {
            ContrastLevel.High -> highContrastDarkColorScheme
            ContrastLevel.Medium -> mediumContrastDarkColorScheme
            ContrastLevel.Normal -> darkScheme
        }

        else -> when (contrastLevel) {
            ContrastLevel.High -> highContrastLightColorScheme
            ContrastLevel.Medium -> mediumContrastLightColorScheme
            ContrastLevel.Normal -> lightScheme
        }
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        content = content
    )
}
```

### Step 3: Use with Settings

```kotlin
// In your settings/preferences
AppTheme(
    contrastLevel = userPreferredContrast  // From SharedPreferences or DataStore
) {
    MyApp()
}
```

---

## 📊 Summary Table

| Component                        | What It Is                     | Key Details                                                        |
| -------------------------------- | ------------------------------ | ------------------------------------------------------------------ |
| `lightScheme`                    | Default light color palette    | Maps 35 color roles from `Color.kt` light values                   |
| `darkScheme`                     | Default dark color palette     | Maps 35 color roles from `Color.kt` dark values                    |
| `mediumContrastLightColorScheme` | Accessibility light scheme     | Higher contrast for WCAG AA+ compliance                            |
| `highContrastLightColorScheme`   | Max accessibility light scheme | Maximum contrast for WCAG AAA compliance                           |
| `mediumContrastDarkColorScheme`  | Accessibility dark scheme      | Higher contrast dark variant                                       |
| `highContrastDarkColorScheme`    | Max accessibility dark scheme  | Maximum contrast dark variant                                      |
| `ColorFamily`                    | Convenience data class         | Groups `color`, `onColor`, `colorContainer`, `onColorContainer`    |
| `unspecified_scheme`             | Default placeholder            | All `Color.Unspecified` — tells Compose to use defaults            |
| `AppTheme()`                     | Root theme composable          | Selects scheme (dynamic/light/dark) and applies `MaterialTheme`    |
| `dynamicColor` param             | Wallpaper-based theming toggle | `true` = use Material You on API 31+, `false` = use manual palette |
| `darkTheme` param                | Dark mode toggle               | Defaults to system setting via `isSystemInDarkTheme()`             |
| `AppTypography`                  | Typography configuration       | Defined in `Type.kt`, applied via `MaterialTheme`                  |

---

## 🧠 Interview Q&A

### Q1: What is the purpose of Theme.kt in a Jetpack Compose app?

**A:** `Theme.kt` is the centralized theming configuration that defines color schemes (light, dark, contrast variants), selects the appropriate scheme at runtime (based on system settings and device capabilities), and wraps the app content in `MaterialTheme` to provide colors, typography, and shapes to all child composables via `CompositionLocal`.

---

### Q2: What is `isSystemInDarkTheme()` and how is it used?

**A:** It's a Compose utility function that reads the device's system-wide dark mode setting and returns `true` if dark mode is active. It's used as the **default value** for the `darkTheme` parameter in `AppTheme`, so the app automatically follows the user's system preference without any extra code.

---

### Q3: What are dynamic colors and when do they NOT work?

**A:** Dynamic colors (Material You) generate a color scheme from the user's wallpaper. They are only available on **Android 12+ (API 31+)**. They don't work on:
- Devices running Android 11 or below
- When explicitly disabled via `dynamicColor = false`
- On non-Google-certified devices that may not implement the dynamic color system

---

### Q4: Why does `dynamicColorScheme` need a `Context`?

**A:** The dynamic color functions need `Context` to access the system's wallpaper-derived color resources at runtime. These colors are stored in system resource tables that are only accessible through `Context`. In Compose, we retrieve it using `LocalContext.current`.

---

### Q5: Why is `@Immutable` used on the `ColorFamily` class?

**A:** The `@Immutable` annotation tells the Compose compiler that once a `ColorFamily` instance is created, its properties will **never change**. This allows Compose to:
- Skip unnecessary recompositions when a `ColorFamily` is passed as a parameter
- Trust that referential equality checks are sufficient (no deep comparison needed)
- Optimize performance in composable functions that receive `ColorFamily` instances

---

### Q6: What happens if you DON'T wrap your app in `AppTheme`?

**A:** Without `AppTheme` (or any `MaterialTheme`), all Material 3 composables will use their **hardcoded default colors** (usually a purple/teal palette). Your custom colors from `Color.kt` won't be applied, dark mode won't work, and `MaterialTheme.colorScheme.xxx` calls will return the default M3 baseline values.

---

### Q7: Can you nest `AppTheme` calls? What happens?

**A:** Yes, you can nest `MaterialTheme`/`AppTheme` calls. The **innermost theme overrides** the outer one for all composables within it. This is useful for:
- Forcing a dark section inside a light app
- Applying a different color scheme to a specific feature
- Preview composables with different themes

---

### Q8: What is `Build.VERSION_CODES.S` and why is it checked?

**A:** `Build.VERSION_CODES.S` is the constant for **Android 12 (API 31)**. It's checked because `dynamicLightColorScheme()` and `dynamicDarkColorScheme()` only exist on Android 12+. Calling them on older devices would crash the app. The check acts as a **runtime guard** to ensure backward compatibility.

---

### Q9: What is the difference between `lightColorScheme()` and `dynamicLightColorScheme()`?

**A:**

| Aspect          | `lightColorScheme()`                    | `dynamicLightColorScheme()`          |
| --------------- | --------------------------------------- | ------------------------------------ |
| Source          | Manually defined colors from `Color.kt` | Auto-generated from device wallpaper |
| Consistency     | Same on every device                    | Different per user/wallpaper         |
| API Requirement | Any                                     | Android 12+ (API 31+)                |
| Brand Control   | Full control                            | Wallpaper-dependent                  |
| Needs Context   | No                                      | Yes                                  |

---

### Q10: How do you add a custom color that's not part of the M3 color scheme?

**A:** Use `CompositionLocal` to extend the theme:

```kotlin
// Define a custom color local
val LocalCustomAccent = staticCompositionLocalOf { Color(0xFF00BFA5) }

@Composable
fun AppTheme(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalCustomAccent provides Color(0xFF00BFA5)
    ) {
        MaterialTheme(
            colorScheme = lightScheme,
            typography = AppTypography,
            content = content
        )
    }
}

// Usage anywhere in the app
val accent = LocalCustomAccent.current
```

---

## 📝 Quick Revision Cheat Sheet

### 🗂️ File Purpose
- `Theme.kt` = Central theming config for the app
- Defines 6 color schemes (light, dark, + 4 contrast variants)
- Contains `AppTheme` composable — the root wrapper

### 🎨 Color Scheme Selection Priority
1. **Dynamic Color** (if enabled + API 31+) → wallpaper-based
2. **Dark Scheme** (if system/user is in dark mode)
3. **Light Scheme** (default fallback)

### 📐 6 Schemes Defined
| Light           | Dark            |
| --------------- | --------------- |
| Normal          | Normal          |
| Medium Contrast | Medium Contrast |
| High Contrast   | High Contrast   |

### 🏗️ AppTheme Parameters
- `darkTheme` → follows system by default
- `dynamicColor` → `true` by default (Material You)
- `content` → your app UI

### 🧩 ColorFamily
- Groups 4 related colors: `color`, `onColor`, `container`, `onContainer`
- `@Immutable` → Compose optimization
- `unspecified_scheme` → all `Color.Unspecified` placeholder

### ⚡ Key Concepts
- `dynamicColorScheme` needs **Context** (from `LocalContext.current`)
- `Build.VERSION_CODES.S` = Android 12 = API 31
- `MaterialTheme` provides colors + typography via `CompositionLocal`
- Nested `AppTheme` calls = inner overrides outer
- `private val` schemes → encapsulated, only exposed via `AppTheme`

### 🔗 Related Files
- **Color.kt** → Hex values for every color role
- **Type.kt** → Typography definitions (`AppTypography`)
- **MainActivity.kt** → Calls `AppTheme { ... }` at the root

### ⚠️ Common Gotchas
1. Forgetting the API check before calling `dynamicColorScheme` → **crash on older devices**
2. Not wrapping app in `AppTheme` → **default ugly purple theme**
3. Defining contrast schemes but never using them → **wasted code**
4. Using `dynamicColor = true` for brand-heavy apps → **inconsistent branding**
5. Hardcoding `darkTheme = false` → **ignoring user's system preference**

---

## Developer Rules of Thumb

1. **Always wrap your app root in `AppTheme`** → It's the foundation for all M3 theming
2. **Default to system settings** → Use `isSystemInDarkTheme()` instead of hardcoding
3. **Enable dynamic colors unless your brand requires specific colors** → Users love Material You
4. **Guard dynamic color calls with API check** → `Build.VERSION.SDK_INT >= Build.VERSION_CODES.S`
5. **Define all 6 schemes** → Even if you start with just light/dark, have contrast variants ready
6. **Use `@Immutable` on custom data classes** → Especially color-related ones passed to composables
7. **Keep `AppTheme` simple** → Only color scheme selection + MaterialTheme wrapping
8. **Put color values in `Color.kt`** → Never define hex values directly in `Theme.kt`
9. **Test with dynamic colors ON and OFF** → Your manual schemes should look just as good
10. **Support accessibility** → Integrate contrast level selection into your app settings

---

> **Created for:** Interview prep & real-world Android development
> **Design System:** Material 3 (Material You)
> **Framework:** Jetpack Compose
> **Related Guide:** [Material3 Color Roles Guide](./Material3_Color_Roles_Guide.md)
