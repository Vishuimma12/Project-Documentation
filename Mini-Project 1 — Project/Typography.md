# 🔤 Material 3 Typography (Type.kt) — Complete Developer Guide (Jetpack Compose)

> **Goal:** Understand how M3 typography is structured, how Google Fonts integration works, and how `AppTypography` customizes text styles across your entire app.
>
> **Audience:** Android developers using Jetpack Compose + Material 3.
>
> **Related Guides:** [Theme Setup Guide](./Material3_Theme_Setup_Guide.md) · [Color Roles Guide](./Material3_Color_Roles_Guide.md)

---

## Table of Contents

1. [What Is Type.kt?](#what-is-typekt)
2. [File Structure Overview](#file-structure-overview)
3. [Imports Breakdown](#imports-breakdown)
4. [Google Fonts Provider Setup](#google-fonts-provider-setup)
5. [Font Families Explained](#font-families-explained)
6. [The Baseline Typography](#the-baseline-typography)
7. [AppTypography — Line by Line](#apptypography--line-by-line)
8. [M3 Typography Scale Deep Dive](#m3-typography-scale-deep-dive)
9. [How Typography Connects to Theme.kt](#how-typography-connects-to-themekt)
10. [Practical Usage in Composables](#practical-usage-in-composables)
11. [Advanced Customization](#advanced-customization)
12. [Summary Table](#-summary-table)
13. [Interview Q&A](#-interview-qa)
14. [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)

---

## What Is Type.kt?

`Type.kt` is the **typography configuration file** for your Compose app. It:

- Sets up **Google Fonts** via the Downloadable Fonts API
- Defines **font families** for different text roles (display vs body)
- Customizes the **M3 Typography scale** (15 text styles) with your chosen fonts

> 💡 **Think of it as:** The "voice" of your app. Colors define the look; typography defines how text *feels*.

### Where Does Type.kt Live?

```
app/
└── src/main/java/com/example/ui/theme/
    ├── Color.kt          ← Color values
    ├── Theme.kt          ← Scheme assembly + AppTheme
    └── Type.kt           ← THIS FILE (typography)
```

---

## File Structure Overview

```
Type.kt
│
├── Imports
│   └── Typography, TextStyle, GoogleFont, FontFamily, etc.
│
├── Google Fonts Provider
│   └── provider (authenticates with Google Play Services)
│
├── Font Families
│   ├── bodyFontFamily    → Inter (for body + label text)
│   └── displayFontFamily → Roboto (for display + headline + title text)
│
├── Baseline Typography
│   └── baseline = Typography()  → M3 defaults
│
└── AppTypography
    └── Custom Typography() with fonts applied to all 15 text styles
```

---

## Imports Breakdown

| Import                    | Purpose                                                                      |
| ------------------------- | ---------------------------------------------------------------------------- |
| `Typography`              | M3 typography class holding all 15 text styles                               |
| `TextStyle`               | Individual text style (font, size, weight, spacing, etc.)                    |
| `FontFamily`              | Groups related fonts into a family                                           |
| `FontWeight`              | Specifies weight (Bold, Normal, Thin, etc.)                                  |
| `sp`                      | Scalable pixels — the unit for text sizes (respects user font size settings) |
| `GoogleFont`              | Represents a Google Font by name                                             |
| `Font` (from googlefonts) | Creates a downloadable font from a Google Font                               |

---

## Google Fonts Provider Setup

```kotlin
val provider = GoogleFont.Provider(
    providerAuthority = "com.google.android.gms.fonts",
    providerPackage = "com.google.android.gms",
    certificates = R.array.com_google_android_gms_fonts_certs
)
```

### What Is This?

A **font provider** that connects your app to **Google Play Services** to download fonts at runtime instead of bundling them in the APK.

### Each Parameter Explained

| Parameter           | Value                                        | Purpose                                                      |
| ------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| `providerAuthority` | `"com.google.android.gms.fonts"`             | Identifies the Google Fonts content provider on the device   |
| `providerPackage`   | `"com.google.android.gms"`                   | Package name of Google Play Services                         |
| `certificates`      | `R.array.com_google_android_gms_fonts_certs` | Security certificates to verify the font provider is genuine |

### How Downloadable Fonts Work

```
┌──────────────┐     Request Font      ┌────────────────────┐
│   Your App   │ ────────────────────→  │  Google Play       │
│  (Type.kt)   │                        │  Services (GMS)    │
│              │ ←────────────────────  │                    │
└──────────────┘     Font Data          │  Font Cache        │
                                        └────────────────────┘
                                               │
                                               ▼
                                        ┌────────────────────┐
                                        │  Google Fonts API   │
                                        │  (1400+ fonts)      │
                                        └────────────────────┘
```

### Benefits of Downloadable Fonts

| Benefit            | Explanation                                          |
| ------------------ | ---------------------------------------------------- |
| **Smaller APK**    | Fonts aren't bundled; downloaded on demand           |
| **Shared Cache**   | Multiple apps using the same font share one download |
| **Easy Switching** | Change font by name string, no new assets needed     |
| **Auto Updates**   | Gets latest font versions automatically              |

### ⚠️ Important: Certificate Setup

The `certificates` reference (`R.array.com_google_android_gms_fonts_certs`) must exist in your `res/values/` folder. It's typically auto-generated when you add the Google Fonts dependency.

**Required in `res/values/font_certs.xml`:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <array name="com_google_android_gms_fonts_certs">
        <item>@array/com_google_android_gms_fonts_certs_dev</item>
        <item>@array/com_google_android_gms_fonts_certs_prod</item>
    </array>
    <!-- Dev and prod cert arrays follow -->
</resources>
```

### Required Gradle Dependencies

```groovy
// build.gradle (app)
dependencies {
    implementation "androidx.compose.ui:ui-text-google-fonts:1.6.0"
}
```

---

## Font Families Explained

### Body Font Family — Inter

```kotlin
val bodyFontFamily = FontFamily(
    Font(
        googleFont = GoogleFont("Inter"),
        fontProvider = provider,
    )
)
```

- **Font:** [Inter](https://fonts.google.com/specimen/Inter) — a clean, modern sans-serif designed for **screen readability**
- **Used for:** Body text, labels — anything the user reads in paragraphs or UI controls
- **Why Inter?** Excellent legibility at small sizes, optimized for digital screens, variable font support

### Display Font Family — Roboto

```kotlin
val displayFontFamily = FontFamily(
    Font(
        googleFont = GoogleFont("Roboto"),
        fontProvider = provider,
    )
)
```

- **Font:** [Roboto](https://fonts.google.com/specimen/Roboto) — Android's signature typeface
- **Used for:** Display text, headlines, titles — large, prominent text
- **Why Roboto?** Material Design's default, familiar to Android users, excellent at large sizes

### The Two-Font Strategy

```
┌─────────────────────────────────────────────────┐
│                   YOUR APP UI                    │
│                                                  │
│   ┌─────────────────────────────────────────┐   │
│   │  "Welcome Back"     ← displayFontFamily │   │
│   │   (Roboto)            (Headlines/Titles)│   │
│   └─────────────────────────────────────────┘   │
│                                                  │
│   ┌─────────────────────────────────────────┐   │
│   │  "Here's what happened while you were   │   │
│   │   away. Check your notifications for    │   │
│   │   updates."          ← bodyFontFamily   │   │
│   │                        (Body/Labels)    │   │
│   │   (Inter)                               │   │
│   └─────────────────────────────────────────┘   │
│                                                  │
│   [ CONTINUE ]  ← bodyFontFamily (Label)        │
│     (Inter)                                      │
└─────────────────────────────────────────────────┘
```

> 💡 **Design Principle:** Using two complementary fonts creates **visual hierarchy**. The display font grabs attention, the body font enables comfortable reading.

---

## The Baseline Typography

```kotlin
val baseline = Typography()
```

### What Is This?

Creates a `Typography` object with **all M3 default values** — default sizes, weights, line heights, and letter spacing.

### Why Create a Baseline?

Instead of defining every property from scratch, we **copy** the defaults and only override `fontFamily`. This ensures:

1. **Correct M3 sizing** — All font sizes follow the official Material 3 type scale
2. **Proper line heights** — M3 has specific line height ratios for readability
3. **Correct letter spacing** — Each text style has optimized letter spacing
4. **Less code & fewer bugs** — Only change what you need

### M3 Default Values (What `Typography()` Contains)

| Style            | Size | Weight       | Line Height | Letter Spacing |
| ---------------- | ---- | ------------ | ----------- | -------------- |
| `displayLarge`   | 57sp | 400 (Normal) | 64sp        | -0.25sp        |
| `displayMedium`  | 45sp | 400          | 52sp        | 0sp            |
| `displaySmall`   | 36sp | 400          | 44sp        | 0sp            |
| `headlineLarge`  | 32sp | 400          | 40sp        | 0sp            |
| `headlineMedium` | 28sp | 400          | 36sp        | 0sp            |
| `headlineSmall`  | 24sp | 400          | 32sp        | 0sp            |
| `titleLarge`     | 22sp | 400          | 28sp        | 0sp            |
| `titleMedium`    | 16sp | 500 (Medium) | 24sp        | 0.15sp         |
| `titleSmall`     | 14sp | 500          | 20sp        | 0.1sp          |
| `bodyLarge`      | 16sp | 400          | 24sp        | 0.5sp          |
| `bodyMedium`     | 14sp | 400          | 20sp        | 0.25sp         |
| `bodySmall`      | 12sp | 400          | 16sp        | 0.4sp          |
| `labelLarge`     | 14sp | 500          | 20sp        | 0.1sp          |
| `labelMedium`    | 12sp | 500          | 16sp        | 0.5sp          |
| `labelSmall`     | 11sp | 500          | 16sp        | 0.5sp          |

---

## AppTypography — Line by Line

```kotlin
val AppTypography = Typography(
    // DISPLAY — Largest text, visual impact
    displayLarge = baseline.displayLarge.copy(fontFamily = displayFontFamily),
    displayMedium = baseline.displayMedium.copy(fontFamily = displayFontFamily),
    displaySmall = baseline.displaySmall.copy(fontFamily = displayFontFamily),

    // HEADLINE — Section headers
    headlineLarge = baseline.headlineLarge.copy(fontFamily = displayFontFamily),
    headlineMedium = baseline.headlineMedium.copy(fontFamily = displayFontFamily),
    headlineSmall = baseline.headlineSmall.copy(fontFamily = displayFontFamily),

    // TITLE — Component/card titles
    titleLarge = baseline.titleLarge.copy(fontFamily = displayFontFamily),
    titleMedium = baseline.titleMedium.copy(fontFamily = displayFontFamily),
    titleSmall = baseline.titleSmall.copy(fontFamily = displayFontFamily),

    // BODY — Paragraph/content text
    bodyLarge = baseline.bodyLarge.copy(fontFamily = bodyFontFamily),
    bodyMedium = baseline.bodyMedium.copy(fontFamily = bodyFontFamily),
    bodySmall = baseline.bodySmall.copy(fontFamily = bodyFontFamily),

    // LABEL — Button text, captions, tabs
    labelLarge = baseline.labelLarge.copy(fontFamily = bodyFontFamily),
    labelMedium = baseline.labelMedium.copy(fontFamily = bodyFontFamily),
    labelSmall = baseline.labelSmall.copy(fontFamily = bodyFontFamily),
)
```

### The `.copy()` Pattern

```kotlin
baseline.displayLarge.copy(fontFamily = displayFontFamily)
```

This means:
1. Take the **default M3 `displayLarge` style** (57sp, Normal weight, 64sp line height, -0.25sp spacing)
2. **Copy everything** as-is
3. **Only replace** the `fontFamily` with our custom `displayFontFamily` (Roboto)
4. All other properties remain at M3 defaults

> 💡 The `.copy()` pattern is a Kotlin data class feature. It creates a new instance with specified changes while keeping everything else identical.

### Font Family Assignment Map

| Text Styles                  | Font Family         | Font   | Why                                           |
| ---------------------------- | ------------------- | ------ | --------------------------------------------- |
| `displayLarge/Medium/Small`  | `displayFontFamily` | Roboto | Large decorative text — impact matters        |
| `headlineLarge/Medium/Small` | `displayFontFamily` | Roboto | Section headers — needs to stand out          |
| `titleLarge/Medium/Small`    | `displayFontFamily` | Roboto | Titles — prominent but smaller than headlines |
| `bodyLarge/Medium/Small`     | `bodyFontFamily`    | Inter  | Reading text — readability is priority        |
| `labelLarge/Medium/Small`    | `bodyFontFamily`    | Inter  | UI controls — clarity at small sizes          |

### Visual Font Assignment

```
displayFontFamily (Roboto)          bodyFontFamily (Inter)
─────────────────────────           ────────────────────────
├── displayLarge  (57sp)            ├── bodyLarge   (16sp)
├── displayMedium (45sp)            ├── bodyMedium  (14sp)
├── displaySmall  (36sp)            ├── bodySmall   (12sp)
├── headlineLarge (32sp)            ├── labelLarge  (14sp)
├── headlineMedium(28sp)            ├── labelMedium (12sp)
├── headlineSmall (24sp)            └── labelSmall  (11sp)
├── titleLarge    (22sp)
├── titleMedium   (16sp)
└── titleSmall    (14sp)

9 styles → Display font              6 styles → Body font
(Visual impact)                       (Readability)
```

---

## M3 Typography Scale Deep Dive

Material 3 defines **5 categories** with **3 sizes** each = **15 text styles**.

### 1. Display (3 styles)

| Style           | Size | Real-World Usage                                     |
| --------------- | ---- | ---------------------------------------------------- |
| `displayLarge`  | 57sp | Hero text, splash screen title, landing page number  |
| `displayMedium` | 45sp | Marketing headline, featured stat, time display      |
| `displaySmall`  | 36sp | Section hero, onboarding title, date/calendar number |

```kotlin
// Hero section
Text(
    text = "Welcome",
    style = MaterialTheme.typography.displayLarge
)
```

> ⚠️ **Use sparingly!** Display styles are for short, impactful text only. Never for paragraphs.

### 2. Headline (3 styles)

| Style            | Size | Real-World Usage                          |
| ---------------- | ---- | ----------------------------------------- |
| `headlineLarge`  | 32sp | Screen title, main section header         |
| `headlineMedium` | 28sp | Sub-section header, dialog title          |
| `headlineSmall`  | 24sp | Card group title, settings section header |

```kotlin
// Screen title
Text(
    text = "Your Tasks",
    style = MaterialTheme.typography.headlineLarge
)
```

### 3. Title (3 styles)

| Style         | Size | Real-World Usage                                |
| ------------- | ---- | ----------------------------------------------- |
| `titleLarge`  | 22sp | TopAppBar title, expanded card title            |
| `titleMedium` | 16sp | List item title, card title, dialog subtitle    |
| `titleSmall`  | 14sp | Tab label, navigation label, compact card title |

```kotlin
// App bar
TopAppBar(
    title = {
        Text(
            text = "Settings",
            style = MaterialTheme.typography.titleLarge
        )
    }
)
```

### 4. Body (3 styles)

| Style        | Size | Real-World Usage                                      |
| ------------ | ---- | ----------------------------------------------------- |
| `bodyLarge`  | 16sp | Primary body text, article content, description       |
| `bodyMedium` | 14sp | Secondary body text, list item subtitle, comment text |
| `bodySmall`  | 12sp | Caption, timestamp, fine print, helper text           |

```kotlin
// Article content
Text(
    text = articleBody,
    style = MaterialTheme.typography.bodyLarge
)

// Timestamp
Text(
    text = "2 hours ago",
    style = MaterialTheme.typography.bodySmall,
    color = MaterialTheme.colorScheme.onSurfaceVariant
)
```

### 5. Label (3 styles)

| Style         | Size | Real-World Usage                                         |
| ------------- | ---- | -------------------------------------------------------- |
| `labelLarge`  | 14sp | Button text, prominent tab label, text field label       |
| `labelMedium` | 12sp | Chip text, menu item, navigation bar label               |
| `labelSmall`  | 11sp | Status indicator, badge text, overline, very small label |

```kotlin
// Button (uses labelLarge by default)
Button(onClick = { }) {
    Text("Submit")  // Automatically uses labelLarge
}

// Overline/status
Text(
    text = "ACTIVE",
    style = MaterialTheme.typography.labelSmall,
    color = MaterialTheme.colorScheme.primary
)
```

### Complete Hierarchy Visualization

```
SIZE (sp)   STYLE              CATEGORY    USAGE
────────────────────────────────────────────────────────────
57sp   ██████████████████ displayLarge     Hero text
45sp   ███████████████   displayMedium    Feature numbers
36sp   ████████████      displaySmall     Section hero
32sp   ██████████        headlineLarge    Screen title
28sp   █████████         headlineMedium   Section header
24sp   ████████          headlineSmall    Group header
22sp   ███████           titleLarge       App bar title
16sp   █████             titleMedium      Card title
14sp   ████              titleSmall       Compact title
16sp   █████             bodyLarge        Main content
14sp   ████              bodyMedium       Secondary text
12sp   ███               bodySmall        Caption/helper
14sp   ████              labelLarge       Button text
12sp   ███               labelMedium      Chip/menu text
11sp   ██                labelSmall       Status/badge
```

---

## How Typography Connects to Theme.kt

In `Theme.kt`, `AppTypography` is passed to `MaterialTheme`:

```kotlin
// Theme.kt
MaterialTheme(
    colorScheme = colorScheme,
    typography = AppTypography,    // ← From Type.kt
    content = content
)
```

This makes all 15 text styles available everywhere via:

```kotlin
MaterialTheme.typography.displayLarge
MaterialTheme.typography.bodyMedium
MaterialTheme.typography.labelSmall
// etc.
```

### The Connection Flow

```
Type.kt                    Theme.kt                    Any Composable
────────                   ─────────                   ──────────────
AppTypography ──────→ MaterialTheme(            Text(
  displayLarge             typography =              style = MaterialTheme
  headlineLarge              AppTypography              .typography
  bodyLarge              )                             .bodyLarge
  labelLarge                                       )
  ...
```

---

## Practical Usage in Composables

### Using Typography Styles

```kotlin
@Composable
fun ArticleCard(title: String, body: String, date: String) {
    Card {
        Column(modifier = Modifier.padding(16.dp)) {
            // Title
            Text(
                text = title,
                style = MaterialTheme.typography.titleMedium,
                color = MaterialTheme.colorScheme.onSurface
            )

            Spacer(modifier = Modifier.height(8.dp))

            // Body preview
            Text(
                text = body,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                maxLines = 3,
                overflow = TextOverflow.Ellipsis
            )

            Spacer(modifier = Modifier.height(12.dp))

            // Date caption
            Text(
                text = date,
                style = MaterialTheme.typography.bodySmall,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}
```

### Common M3 Component → Typography Mapping

| Component                  | Default Typography Style |
| -------------------------- | ------------------------ |
| `TopAppBar` title          | `titleLarge`             |
| `Button` text              | `labelLarge`             |
| `Card` title               | `titleMedium`            |
| `ListItem` headline        | `bodyLarge`              |
| `ListItem` supporting text | `bodyMedium`             |
| `NavigationBar` label      | `labelMedium`            |
| `Chip` label               | `labelLarge`             |
| `Dialog` title             | `headlineSmall`          |
| `TextField` input          | `bodyLarge`              |
| `TextField` label          | `bodySmall`              |
| `Snackbar` text            | `bodyMedium`             |
| `Tab` text                 | `titleSmall`             |

---

## Advanced Customization

### Adding Font Weights

```kotlin
val bodyFontFamily = FontFamily(
    Font(googleFont = GoogleFont("Inter"), fontProvider = provider, weight = FontWeight.Normal),
    Font(googleFont = GoogleFont("Inter"), fontProvider = provider, weight = FontWeight.Medium),
    Font(googleFont = GoogleFont("Inter"), fontProvider = provider, weight = FontWeight.Bold),
)
```

### Customizing Beyond Font Family

```kotlin
val AppTypography = Typography(
    displayLarge = baseline.displayLarge.copy(
        fontFamily = displayFontFamily,
        fontWeight = FontWeight.Bold,        // Override weight
        letterSpacing = (-1).sp              // Tighter tracking
    ),
    bodyLarge = baseline.bodyLarge.copy(
        fontFamily = bodyFontFamily,
        lineHeight = 28.sp                   // More breathing room
    ),
    // ...
)
```

### Using Local/Bundled Fonts Instead

```kotlin
// Instead of Google Fonts, use fonts from res/font/
val bodyFontFamily = FontFamily(
    Font(R.font.inter_regular, FontWeight.Normal),
    Font(R.font.inter_medium, FontWeight.Medium),
    Font(R.font.inter_bold, FontWeight.Bold),
)
```

### Fallback Handling for Google Fonts

```kotlin
val bodyFontFamily = FontFamily(
    Font(
        googleFont = GoogleFont("Inter"),
        fontProvider = provider,
    ),
    // Fallback if Google Fonts fails to load
    Font(R.font.inter_regular)  // Bundled fallback
)
```

> ⚠️ If Google Fonts fails to download (no internet, GMS unavailable), text renders in the **system default font** unless you provide a bundled fallback.

---

## 📊 Summary Table

| Component           | What It Is                | Key Details                                                   |
| ------------------- | ------------------------- | ------------------------------------------------------------- |
| `provider`          | Google Fonts provider     | Authenticates with Google Play Services to download fonts     |
| `bodyFontFamily`    | Inter font family         | Used for body (3) and label (3) = 6 text styles               |
| `displayFontFamily` | Roboto font family        | Used for display (3), headline (3), title (3) = 9 text styles |
| `baseline`          | Default M3 Typography     | All 15 styles with official M3 sizes, weights, spacing        |
| `AppTypography`     | Custom Typography         | Copies baseline, replaces only `fontFamily` per style         |
| `.copy()` pattern   | Kotlin data class feature | Change specific properties while keeping all others           |
| `displayLarge`      | Largest text style        | 57sp — hero text, splash screens                              |
| `bodyMedium`        | Default body text         | 14sp — most common reading text                               |
| `labelLarge`        | Button/control text       | 14sp, Medium weight — buttons, chips                          |

---

## 🧠 Interview Q&A

### Q1: What is the M3 Typography scale and how many styles does it contain?

**A:** The M3 Typography scale defines **15 text styles** across **5 categories** (Display, Headline, Title, Body, Label) with **3 sizes** each (Large, Medium, Small). Each style has predefined font size, weight, line height, and letter spacing optimized for its intended use case.

---

### Q2: What is the difference between Downloadable Fonts and bundled fonts?

**A:**

| Aspect     | Downloadable (Google Fonts)        | Bundled (res/font/)   |
| ---------- | ---------------------------------- | --------------------- |
| APK Size   | Smaller (fonts not included)       | Larger (fonts in APK) |
| First Load | Slight delay (download needed)     | Instant               |
| Offline    | May fall back to default           | Always available      |
| Shared     | Cached across apps using same font | Per-app only          |
| Updates    | Auto-updated by GMS                | Manual updates        |

---

### Q3: Why use `.copy(fontFamily = ...)` instead of defining `TextStyle` from scratch?

**A:** Using `.copy()` preserves all M3-optimized defaults (font size, line height, weight, letter spacing) and only changes what you need. Defining from scratch risks:
- Wrong sizes that don't match M3 specs
- Missing letter spacing optimizations
- Incorrect line heights affecting readability
- More code to maintain

---

### Q4: Why are there two separate font families instead of one?

**A:** Using two fonts creates **typographic contrast** and **visual hierarchy**:
- **Display font** (Roboto) → Eye-catching at large sizes for headlines and titles
- **Body font** (Inter) → Optimized for readability at smaller sizes for content text

This is a standard design practice. Using one font for everything reduces visual hierarchy.

---

### Q5: What happens if Google Fonts fails to load (e.g., no internet)?

**A:** The text renders in the **system default font** (typically Roboto on Android). To handle this gracefully:
- Add a **bundled fallback font** in the `FontFamily` constructor
- The system tries Google Fonts first, then falls back to the bundled version
- No crashes — the font system handles failures silently

---

### Q6: How does `MaterialTheme.typography` work under the hood?

**A:** When you pass `AppTypography` to `MaterialTheme(typography = AppTypography)`, it stores the `Typography` object in a `CompositionLocal` called `LocalTypography`. Any child composable can access it via `MaterialTheme.typography`, which reads from this `CompositionLocal`. This is dependency injection via the Compose tree.

---

### Q7: Which typography style does a `Button` use by default?

**A:** `labelLarge` (14sp, Medium weight). All M3 buttons (`Button`, `FilledTonalButton`, `OutlinedButton`, `TextButton`) use `labelLarge` for their text. This is set internally by the button composable's default `TextStyle`.

---

### Q8: What is `providerAuthority` in the Google Fonts provider?

**A:** It's the **content provider URI authority** that identifies the Google Fonts service within Google Play Services (`com.google.android.gms.fonts`). Android's content provider system uses this to locate the correct service for downloading fonts on the device.

---

### Q9: Can you use more than two font families?

**A:** Yes. You can define as many `FontFamily` instances as needed and assign them to different text styles. For example:
- Display → decorative serif font
- Headline/Title → sans-serif brand font
- Body → high-readability font
- Label → monospace or technical font

However, using more than 2–3 fonts can make the UI look inconsistent. The M3 guideline recommends at most **two complementary typefaces**.

---

### Q10: What is `sp` and why is it used for text sizes?

**A:** `sp` (Scalable Pixels) is a unit that respects the user's **system font size preference** (Accessibility settings). Unlike `dp` which only scales with screen density, `sp` also scales when users choose "Large text" in settings. Using `sp` ensures your app is **accessible** to users with visual impairments.

```
dp → scales with screen density only
sp → scales with screen density + user font size preference
```

---

## 📝 Quick Revision Cheat Sheet

### 🗂️ File Purpose
- `Type.kt` = Typography configuration for the app
- Defines font families + customizes M3's 15 text styles
- Used by `Theme.kt` via `MaterialTheme(typography = AppTypography)`

### 🔤 Font Assignment

| Display Font (Roboto)        | Body Font (Inter)         |
| ---------------------------- | ------------------------- |
| `displayLarge/Medium/Small`  | `bodyLarge/Medium/Small`  |
| `headlineLarge/Medium/Small` | `labelLarge/Medium/Small` |
| `titleLarge/Medium/Small`    |                           |
| **9 styles**                 | **6 styles**              |

### 📏 Size Quick Reference

| Category | L / M / S sizes |
| -------- | --------------- |
| Display  | 57 / 45 / 36 sp |
| Headline | 32 / 28 / 24 sp |
| Title    | 22 / 16 / 14 sp |
| Body     | 16 / 14 / 12 sp |
| Label    | 14 / 12 / 11 sp |

### 🎯 Component → Style Defaults
- `TopAppBar` → `titleLarge`
- `Button` → `labelLarge`
- `Card title` → `titleMedium`
- `ListItem` headline → `bodyLarge`
- `NavigationBar` label → `labelMedium`

### ⚡ Key Concepts
- `baseline = Typography()` → M3 defaults
- `.copy(fontFamily = ...)` → change font, keep everything else
- `GoogleFont.Provider` → downloads fonts via Google Play Services
- `sp` → text unit that respects user accessibility settings
- `@Composable` access → `MaterialTheme.typography.bodyMedium`

### ⚠️ Common Gotchas
1. Missing `font_certs.xml` → Google Fonts silently fails
2. No bundled fallback → ugly system font when offline
3. Defining `TextStyle` from scratch → loses M3 defaults
4. Using `dp` for text → ignores accessibility font scaling
5. Using one font for everything → weak visual hierarchy

---

## Developer Rules of Thumb

1. **Always use `MaterialTheme.typography.xxx`** → Never hardcode `TextStyle` with raw sp values
2. **Use `.copy()` to customize** → Preserves M3 defaults you don't want to change
3. **Two fonts max** → Display + body is the sweet spot for visual hierarchy
4. **Always provide a bundled fallback** → Google Fonts can fail silently
5. **Use `sp` for all text sizes** → Ensures accessibility compliance
6. **Match component expectations** → Buttons use `labelLarge`, titles use `titleLarge`, etc.
7. **Don't use Display styles for long text** → They're for short, impactful text only
8. **Test with large font settings** → Ensure your layout doesn't break with 200% font scale
9. **Keep `Type.kt` focused** → Only typography; colors go in `Color.kt`, shapes in `Shape.kt`
10. **Add font weights for rich text** → Load Normal, Medium, and Bold variants for flexibility

---

> **Created for:** Interview prep & real-world Android development
> **Design System:** Material 3 (Material You)
> **Framework:** Jetpack Compose
> **Related Guides:** [Theme Setup](./Material3_Theme_Setup_Guide.md) · [Color Roles](./Material3_Color_Roles_Guide.md)
