# 🎨 Material 3 Color Roles — Complete Developer Guide (Jetpack Compose)

> **Goal:** Understand every color role in `MaterialTheme.colorScheme`, know where each is used in real apps, and avoid common mistakes.
>
> **Audience:** Android developers using Jetpack Compose + Material 3.

---

## Table of Contents

1. [How Material 3 Color System Works](#how-material-3-color-system-works)
2. [Primary Colors](#1-primary-colors)
3. [Secondary Colors](#2-secondary-colors)
4. [Tertiary Colors](#3-tertiary-colors)
5. [Error Colors](#4-error-colors)
6. [Background & Surface Colors](#5-background--surface-colors)
7. [Surface Variant & Outline](#6-surface-variant--outline)
8. [Inverse Colors](#7-inverse-colors)
9. [Surface Containers (M3 Tone Layers)](#8-surface-containers)
10. [Scrim](#9-scrim)
11. [Summary Table](#-summary-table)
12. [Interview Q&A](#-interview-qa)
13. [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)

---

## How Material 3 Color System Works

Material 3 uses a **tonal palette** system generated from a **seed color**. Every color has a **role** that tells you *what* it's for and an **on-** counterpart that's readable against it.

### The Golden Rule

> **Every `color` has an `onColor`.** If you fill a surface with `primary`, text on it must use `onPrimary`.

```
┌────────────────────────────────────┐
│  Background: primaryContainer      │
│  Text color: onPrimaryContainer    │
│                                    │
│  ✅ Readable & accessible          │
└────────────────────────────────────┘
```

### Color Pairing Cheat Sheet

| Fill With            | Text/Icon Must Use     |
| -------------------- | ---------------------- |
| `primary`            | `onPrimary`            |
| `primaryContainer`   | `onPrimaryContainer`   |
| `secondary`          | `onSecondary`          |
| `secondaryContainer` | `onSecondaryContainer` |
| `tertiary`           | `onTertiary`           |
| `tertiaryContainer`  | `onTertiaryContainer`  |
| `error`              | `onError`              |
| `errorContainer`     | `onErrorContainer`     |
| `surface`            | `onSurface`            |
| `surfaceVariant`     | `onSurfaceVariant`     |
| `inverseSurface`     | `inverseOnSurface`     |
| `background`         | `onBackground`         |

---

## 1. Primary Colors

### `primary`

- **What:** The brand's main accent color. The most prominent color in the UI.
- **Where Used:** Filled buttons, FAB, active navigation items, key highlights.
- **Real Examples:** "Send" button in Gmail, FAB in Google Keep, selected bottom nav icon.

```kotlin
Button(
    onClick = { /* ... */ },
    colors = ButtonDefaults.buttonColors(
        containerColor = MaterialTheme.colorScheme.primary,
        contentColor = MaterialTheme.colorScheme.onPrimary
    )
) {
    Text("Submit")
}
```

### `onPrimary`

- **What:** Color for text/icons that appear ON a `primary`-colored surface.
- **Where Used:** Text on filled buttons, icon on FAB.

> ⚠️ **Mistake:** Using `onPrimary` on a `surface` background → unreadable text.

### `primaryContainer`

- **What:** A lighter/softer tint of primary. Used for **less prominent** elements.
- **Where Used:** Tonal buttons, chips, active input fields, selected states, cards that need subtle brand color.

```kotlin
// Tonal Button (uses primaryContainer by default)
FilledTonalButton(onClick = { }) {
    Text("Save Draft")
}

// Custom card with container color
Card(
    colors = CardDefaults.cardColors(
        containerColor = MaterialTheme.colorScheme.primaryContainer
    )
) {
    Text(
        text = "Featured Item",
        color = MaterialTheme.colorScheme.onPrimaryContainer
    )
}
```

### `onPrimaryContainer`

- **What:** Text/icon color that sits on `primaryContainer`.
- **Where Used:** Text inside tonal buttons, label on chips, titles in branded cards.

> ⚠️ **Mistake:** Using `onPrimary` on `primaryContainer` → wrong contrast, accessibility violation.

### ❌ Common Beginner Mistakes — Primary

| Mistake                                       | Why It's Wrong                                                           |
| --------------------------------------------- | ------------------------------------------------------------------------ |
| Using `primary` as background for large areas | Primary is for accents, not backgrounds. Use `surface` instead.          |
| Mixing `onPrimary` with `primaryContainer`    | `onPrimary` is designed for `primary`, not `primaryContainer`.           |
| Using `primary` for all buttons               | Reserve filled primary buttons for the **single most important action**. |

---

## 2. Secondary Colors

### `secondary`

- **What:** A supporting color that complements `primary`. Lower emphasis than primary.
- **Where Used:** Filter chips, less prominent buttons, toggles, smaller accents.

```kotlin
FilterChip(
    selected = isSelected,
    onClick = { },
    label = { Text("Category") },
    colors = FilterChipDefaults.filterChipColors(
        selectedContainerColor = MaterialTheme.colorScheme.secondaryContainer,
        selectedLabelColor = MaterialTheme.colorScheme.onSecondaryContainer
    )
)
```

### `onSecondary`

- **What:** Text/icon on `secondary` background.

### `secondaryContainer`

- **What:** Soft tint of secondary. Used for supporting visual elements.
- **Where Used:** Navigation drawer selected indicator, active filter chips, secondary cards.

```kotlin
// Navigation Drawer selected item
NavigationDrawerItem(
    label = { Text("Inbox") },
    selected = true,
    onClick = { },
    colors = NavigationDrawerItemDefaults.colors(
        selectedContainerColor = MaterialTheme.colorScheme.secondaryContainer,
        selectedTextColor = MaterialTheme.colorScheme.onSecondaryContainer
    )
)
```

### `onSecondaryContainer`

- **What:** Text/icon on `secondaryContainer`.

### ❌ Common Beginner Mistakes — Secondary

| Mistake                                   | Why It's Wrong                                            |
| ----------------------------------------- | --------------------------------------------------------- |
| Using `secondary` for the main CTA button | Secondary = supporting role. Main CTA = `primary`.        |
| Ignoring `secondaryContainer`             | Many M3 components default to it. Understand the mapping. |

---

## 3. Tertiary Colors

### `tertiary`

- **What:** A third accent for **additional contrast** when primary and secondary aren't enough.
- **Where Used:** Special badges, accent elements, toggles, complementary highlights, progress indicators.

```kotlin
Badge(
    containerColor = MaterialTheme.colorScheme.tertiary,
    contentColor = MaterialTheme.colorScheme.onTertiary
) {
    Text("NEW")
}
```

### `tertiaryContainer` / `onTertiaryContainer`

- **What:** Softer version for subtle tertiary accents.
- **Where Used:** Input chip selected state, complementary info cards, alternative highlighted sections.

```kotlin
Surface(
    color = MaterialTheme.colorScheme.tertiaryContainer,
    shape = RoundedCornerShape(12.dp)
) {
    Text(
        text = "Pro Tip: Use tertiary for extras",
        color = MaterialTheme.colorScheme.onTertiaryContainer,
        modifier = Modifier.padding(16.dp)
    )
}
```

### ❌ Common Beginner Mistakes — Tertiary

| Mistake                            | Why It's Wrong                                                   |
| ---------------------------------- | ---------------------------------------------------------------- |
| Using tertiary as a second primary | Tertiary is for *occasional* accents, not dominant UI elements.  |
| Skipping tertiary entirely         | It exists to give you a third layer of visual hierarchy. Use it. |

---

## 4. Error Colors

### `error`

- **What:** Indicates errors, destructive actions, required field warnings.
- **Where Used:** Validation error text, destructive button, error snackbar.

```kotlin
// Error text on a form field
OutlinedTextField(
    value = email,
    onValueChange = { email = it },
    isError = !isValid,
    label = { Text("Email") },
    supportingText = {
        if (!isValid) {
            Text(
                text = "Invalid email format",
                color = MaterialTheme.colorScheme.error
            )
        }
    }
)
```

### `onError`

- **What:** Text/icon on `error` background.
- **Where Used:** "Delete" text on a red error button.

### `errorContainer`

- **What:** Softer background for error states. Less alarming.
- **Where Used:** Error banners, validation error backgrounds, error alert cards.

```kotlin
// Error banner
Surface(
    color = MaterialTheme.colorScheme.errorContainer,
    shape = RoundedCornerShape(12.dp)
) {
    Row(modifier = Modifier.padding(16.dp)) {
        Icon(
            Icons.Filled.Warning,
            contentDescription = null,
            tint = MaterialTheme.colorScheme.onErrorContainer
        )
        Spacer(Modifier.width(8.dp))
        Text(
            "Payment failed. Try again.",
            color = MaterialTheme.colorScheme.onErrorContainer
        )
    }
}
```

### `onErrorContainer`

- **What:** Text/icon on `errorContainer`.

### ❌ Common Beginner Mistakes — Error

| Mistake                    | Why It's Wrong                                                                         |
| -------------------------- | -------------------------------------------------------------------------------------- |
| Using `Color.Red` directly | Breaks theming, dark mode, and accessibility. Use `error`.                             |
| Using `error` for warnings | `error` is for **errors only**. Use `tertiary` or custom semantic colors for warnings. |

---

## 5. Background & Surface Colors

### `background`

- **What:** The bottommost canvas color of the app.
- **Where Used:** Behind all content — the root `Scaffold` or `Box` background.

> 📝 In M3, `background` and `surface` are often **the same color**. Google recommends using `surface` for most things.

### `onBackground`

- **What:** Text/icon on `background`.

### `surface`

- **What:** The primary surface color for cards, sheets, dialogs, menus, app bars — essentially any "elevated" component.
- **Where Used:** Card backgrounds, Bottom Sheet, ModalDrawer, TopAppBar, Dialog.

```kotlin
Scaffold(
    containerColor = MaterialTheme.colorScheme.surface
) { paddingValues ->
    // App content
    Card(
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface
        )
    ) {
        Text(
            "Card Content",
            color = MaterialTheme.colorScheme.onSurface
        )
    }
}
```

### `onSurface`

- **What:** Default text/icon color on any surface.
- **Where Used:** Body text, titles, icons throughout the app. **This is your "default text color".**

```kotlin
Text(
    text = "Hello, World!",
    color = MaterialTheme.colorScheme.onSurface // default readable text
)
```

### ❌ Common Beginner Mistakes — Surface

| Mistake                                       | Why It's Wrong                                          |
| --------------------------------------------- | ------------------------------------------------------- |
| Using `Color.White` or `Color.Black` for text | Breaks dark mode. Use `onSurface`.                      |
| Using `Color.White` for card backgrounds      | Breaks dark mode. Use `surface` or a container variant. |
| Confusing `background` vs `surface`           | In M3 they're almost identical. Prefer `surface`.       |

---

## 6. Surface Variant & Outline

### `surfaceVariant`

- **What:** A slightly contrasted surface for visual separation from the main `surface`.
- **Where Used:** Text field backgrounds, search bar fill, divider areas, secondary cards.

```kotlin
OutlinedTextField(
    value = query,
    onValueChange = { query = it },
    colors = OutlinedTextFieldDefaults.colors(
        unfocusedContainerColor = MaterialTheme.colorScheme.surfaceVariant
    ),
    label = { Text("Search") }
)
```

### `onSurfaceVariant`

- **What:** Text/icon on `surfaceVariant`. Also used for **secondary text/icons** (lower emphasis than `onSurface`).
- **Where Used:** Placeholder text, helper/supporting text, trailing icons, subtitle text.

```kotlin
Text(
    text = "Last updated 5 min ago",
    color = MaterialTheme.colorScheme.onSurfaceVariant, // secondary text
    style = MaterialTheme.typography.bodySmall
)
```

### `outline`

- **What:** Borders, dividers, and separators for medium-emphasis elements.
- **Where Used:** OutlinedTextField border, OutlinedButton border, Divider, Card outlines.

```kotlin
OutlinedButton(
    onClick = { },
    border = BorderStroke(1.dp, MaterialTheme.colorScheme.outline)
) {
    Text("Cancel")
}

HorizontalDivider(color = MaterialTheme.colorScheme.outline)
```

### `outlineVariant`

- **What:** A lighter/subtler outline for low-emphasis borders.
- **Where Used:** Subtle card borders, table grid lines, list dividers between items.

```kotlin
HorizontalDivider(
    color = MaterialTheme.colorScheme.outlineVariant,
    thickness = 0.5.dp
)
```

### ❌ Common Beginner Mistakes — Outline

| Mistake                             | Why It's Wrong                                                 |
| ----------------------------------- | -------------------------------------------------------------- |
| Using `Color.Gray` for borders      | Breaks theming. Use `outline` or `outlineVariant`.             |
| Using `outline` for disabled states | Use `onSurface.copy(alpha = 0.38f)` for disabled per M3 specs. |

---

## 7. Inverse Colors

### `inverseSurface`

- **What:** A surface color that is the **opposite** of the current theme (dark in light mode, light in dark mode).
- **Where Used:** **Snackbars**, tooltips — components that need to "pop" against the background.

```kotlin
Snackbar(
    containerColor = MaterialTheme.colorScheme.inverseSurface,
    contentColor = MaterialTheme.colorScheme.inverseOnSurface
) {
    Text("Item deleted")
}
```

### `inverseOnSurface`

- **What:** Text/icon on `inverseSurface`.
- **Where Used:** Snackbar text, tooltip text.

### `inversePrimary`

- **What:** Primary color adjusted for use on `inverseSurface`.
- **Where Used:** Snackbar action button (e.g., "Undo" in the snackbar).

```kotlin
Snackbar(
    containerColor = MaterialTheme.colorScheme.inverseSurface,
    contentColor = MaterialTheme.colorScheme.inverseOnSurface,
    action = {
        TextButton(onClick = { /* undo */ }) {
            Text(
                "UNDO",
                color = MaterialTheme.colorScheme.inversePrimary
            )
        }
    }
) {
    Text("Message archived")
}
```

### ❌ Common Beginner Mistakes — Inverse

| Mistake                                        | Why It's Wrong                                                      |
| ---------------------------------------------- | ------------------------------------------------------------------- |
| Using `inverseSurface` as a regular card color | It's meant for pop-up components like snackbars.                    |
| Using `primary` on snackbar actions            | Use `inversePrimary` to ensure readability on the inverted surface. |

---

## 8. Surface Containers

> **New in Material 3!** These replace the old elevation-based tinting system.

Material 3 introduces **5 levels of surface containers** to create depth hierarchy **without shadows**.

| Token                     | Elevation Equivalent | Typical Use                         |
| ------------------------- | -------------------- | ----------------------------------- |
| `surfaceContainerLowest`  | Lowest emphasis      | Page background behind cards        |
| `surfaceContainerLow`     | Low emphasis         | Side sheets, navigation rail        |
| `surfaceContainer`        | Default              | Cards, dialogs                      |
| `surfaceContainerHigh`    | Higher emphasis      | Bottom sheet, search bar on scroll  |
| `surfaceContainerHighest` | Highest emphasis     | Text field fill, active input areas |

```kotlin
// Layered surfaces for visual depth
Column {
    // Bottom layer
    Surface(color = MaterialTheme.colorScheme.surfaceContainerLowest) {
        // Middle layer card
        Card(
            colors = CardDefaults.cardColors(
                containerColor = MaterialTheme.colorScheme.surfaceContainer
            )
        ) {
            // Inner highlight
            Surface(
                color = MaterialTheme.colorScheme.surfaceContainerHighest,
                shape = RoundedCornerShape(8.dp)
            ) {
                Text(
                    "Highlighted content",
                    modifier = Modifier.padding(12.dp),
                    color = MaterialTheme.colorScheme.onSurface
                )
            }
        }
    }
}
```

### `surfaceDim` and `surfaceBright`

- **`surfaceDim`:** A dimmed surface for receding backgrounds (e.g., behind a modal).
- **`surfaceBright`:** A bright surface for maximum contrast areas.

```kotlin
// Dim background behind a bottom sheet
Box(
    modifier = Modifier
        .fillMaxSize()
        .background(MaterialTheme.colorScheme.surfaceDim)
)
```

### ❌ Common Beginner Mistakes — Surface Containers

| Mistake                              | Why It's Wrong                                                            |
| ------------------------------------ | ------------------------------------------------------------------------- |
| Using hardcoded gray tones for depth | Use surface container tokens — they adapt to light/dark mode.             |
| Ignoring the container hierarchy     | Random container levels break visual order. Go from `Lowest` → `Highest`. |

---

## 9. Scrim

### `scrim`

- **What:** A semi-transparent overlay behind dialogs, modals, and bottom sheets to dim the background.
- **Where Used:** Behind `ModalBottomSheet`, `AlertDialog`, `ModalDrawer`.

```kotlin
// Typically used via alpha
Box(
    modifier = Modifier
        .fillMaxSize()
        .background(MaterialTheme.colorScheme.scrim.copy(alpha = 0.32f))
)
```

> Usually you **don't need to apply this manually** — M3 components handle it.

---

## 10. Medium & High Contrast Variants

Your color file includes **MediumContrast** and **HighContrast** variants. These are used for **accessibility**.

| Variant               | Purpose                                                   |
| --------------------- | --------------------------------------------------------- |
| `LightMediumContrast` | Slightly higher contrast for users who need it (WCAG AA+) |
| `LightHighContrast`   | Maximum contrast for visually impaired users (WCAG AAA)   |
| `DarkMediumContrast`  | Medium contrast dark theme                                |
| `DarkHighContrast`    | Maximum contrast dark theme                               |

```kotlin
// In your Theme.kt — switch based on user preference
val colorScheme = when {
    contrastLevel == ContrastLevel.High && darkTheme -> highContrastDarkColorScheme
    contrastLevel == ContrastLevel.Medium && darkTheme -> mediumContrastDarkColorScheme
    darkTheme -> darkColorScheme
    contrastLevel == ContrastLevel.High -> highContrastLightColorScheme
    contrastLevel == ContrastLevel.Medium -> mediumContrastLightColorScheme
    else -> lightColorScheme
}

MaterialTheme(colorScheme = colorScheme) {
    // App content
}
```

---

## 📊 Summary Table

| Color Role                | Purpose                         | Real UI Usage                           | Common Mistake                                           |
| ------------------------- | ------------------------------- | --------------------------------------- | -------------------------------------------------------- |
| `primary`                 | Main brand accent               | Filled button, FAB, active nav icon     | Using it as a background for large areas                 |
| `onPrimary`               | Text/icon on primary            | Button label, FAB icon                  | Using on wrong background                                |
| `primaryContainer`        | Soft primary tint               | Tonal button, chip, card highlight      | Pairing with `onPrimary` instead of `onPrimaryContainer` |
| `onPrimaryContainer`      | Text/icon on primaryContainer   | Tonal button text, chip label           | Mixing with other `on-` colors                           |
| `secondary`               | Supporting accent               | Filter chip, toggle, small accent       | Using as main CTA                                        |
| `onSecondary`             | Text/icon on secondary          | Chip label when selected                | Wrong pairing                                            |
| `secondaryContainer`      | Soft secondary tint             | Nav drawer selected, chip background    | Ignoring it entirely                                     |
| `onSecondaryContainer`    | Text/icon on secondaryContainer | Nav item text                           | Wrong background pairing                                 |
| `tertiary`                | Third accent color              | Badge, special toggle, accent           | Overusing it                                             |
| `onTertiary`              | Text/icon on tertiary           | Badge text                              | Wrong pairing                                            |
| `tertiaryContainer`       | Soft tertiary tint              | Complementary info card                 | Confusing with secondaryContainer                        |
| `onTertiaryContainer`     | Text/icon on tertiaryContainer  | Info card text                          | Wrong pairing                                            |
| `error`                   | Error state color               | Validation error, destructive action    | Using `Color.Red` directly                               |
| `onError`                 | Text/icon on error              | Delete button text                      | Wrong pairing                                            |
| `errorContainer`          | Soft error background           | Error banner, alert card                | Using `error` for all error UI                           |
| `onErrorContainer`        | Text/icon on errorContainer     | Error banner text                       | Wrong pairing                                            |
| `background`              | Root canvas                     | App background (behind everything)      | Confusing with `surface`                                 |
| `onBackground`            | Text/icon on background         | Body text on background                 | Using `Color.Black`                                      |
| `surface`                 | Component surfaces              | Cards, sheets, dialogs, app bar         | Using `Color.White`                                      |
| `onSurface`               | Default text/icon color         | Body text, titles, icons                | Using hardcoded colors                                   |
| `surfaceVariant`          | Slightly contrasted surface     | Text field fill, search bar             | Confusing with `surface`                                 |
| `onSurfaceVariant`        | Secondary text/icon             | Placeholder, helper text, subtitle      | Using `onSurface` for all text                           |
| `outline`                 | Medium borders/dividers         | TextField border, OutlinedButton border | Using `Color.Gray`                                       |
| `outlineVariant`          | Subtle borders                  | List dividers, table lines              | Using `outline` for everything                           |
| `inverseSurface`          | Opposite-theme surface          | Snackbar, tooltip                       | Using as regular card color                              |
| `inverseOnSurface`        | Text on inverseSurface          | Snackbar text                           | Using `onSurface` in snackbar                            |
| `inversePrimary`          | Primary on inverseSurface       | Snackbar action ("Undo")                | Using `primary` in snackbar                              |
| `surfaceContainerLowest`  | Lowest emphasis surface         | Page background behind cards            | Using hardcoded grays                                    |
| `surfaceContainerLow`     | Low emphasis surface            | Side sheet, nav rail                    | Random container selection                               |
| `surfaceContainer`        | Default container               | Cards, dialogs                          | Ignoring the hierarchy                                   |
| `surfaceContainerHigh`    | Higher emphasis surface         | Bottom sheet, search bar                | Skipping levels                                          |
| `surfaceContainerHighest` | Highest emphasis surface        | Text field fill, input area             | Using for low-emphasis areas                             |
| `surfaceDim`              | Dimmed surface                  | Behind modal content                    | Confusing with `scrim`                                   |
| `surfaceBright`           | Maximum brightness surface      | High-contrast foreground area           | Overusing                                                |
| `scrim`                   | Modal overlay                   | Behind dialogs and sheets               | Applying manually (components handle it)                 |

---

## 🧠 Interview Q&A

### Q1: What is the difference between `primary` and `primaryContainer`?

**A:** `primary` is the strong brand accent used for high-emphasis elements (filled buttons, FAB). `primaryContainer` is a softer tint used for medium-emphasis elements (tonal buttons, chips). They always pair with their respective `on-` counterparts for readability.

---

### Q2: Why should we NOT use `Color.White` or `Color.Black` directly?

**A:** Hardcoded colors break dark/light theme support and accessibility contrast levels. Material 3's `onSurface`, `surface`, etc., automatically adapt to the active theme.

---

### Q3: What colors does a Snackbar use?

**A:** `inverseSurface` for background, `inverseOnSurface` for content text, and `inversePrimary` for the action button. These ensure the snackbar visually "pops" against the current theme.

---

### Q4: How does Material 3 handle elevation without shadows?

**A:** M3 uses **surface container tokens** (`surfaceContainerLowest` to `surfaceContainerHighest`) to create tonal depth. Higher containers = slightly different tone, giving a layered feel without drop shadows.

---

### Q5: What is the `scrim` color?

**A:** `scrim` is typically `Color.Black` with reduced alpha (~32%). It's used as an overlay behind modals, dialogs, and bottom sheets to dim background content.

---

### Q6: When would you use `surfaceVariant` vs `surfaceContainer`?

**A:** `surfaceVariant` has a noticeable tint from the theme's palette (used for text field fills, search bars). `surfaceContainer` tokens are neutral gray-ish tones used for depth layering (cards, sheets).

---

### Q7: What is the role of `onSurfaceVariant`?

**A:** It's for **secondary-emphasis text and icons** — placeholder text, supporting text, trailing icons, and subtitles. It's less prominent than `onSurface` but still readable on `surfaceVariant`.

---

### Q8: What are Medium and High Contrast color schemes?

**A:** They are accessibility variants with increased contrast ratios. Medium contrast targets WCAG AA+, High contrast targets WCAG AAA. Apps should offer these options for users with visual impairments.

---

### Q9: How do you correctly theme a custom component?

**A:**
```kotlin
@Composable
fun StatusCard(message: String, isError: Boolean) {
    val containerColor = if (isError)
        MaterialTheme.colorScheme.errorContainer
    else
        MaterialTheme.colorScheme.primaryContainer

    val contentColor = if (isError)
        MaterialTheme.colorScheme.onErrorContainer
    else
        MaterialTheme.colorScheme.onPrimaryContainer

    Surface(
        color = containerColor,
        shape = RoundedCornerShape(12.dp)
    ) {
        Text(
            text = message,
            color = contentColor,
            modifier = Modifier.padding(16.dp)
        )
    }
}
```

---

### Q10: What is the visual hierarchy of emphasis in M3 colors?

**A:**
```
High Emphasis:   primary / onPrimary
Medium Emphasis: primaryContainer / onPrimaryContainer
                 secondaryContainer / onSecondaryContainer
Low Emphasis:    surface / onSurface
                 surfaceVariant / onSurfaceVariant
Minimal:         outlineVariant
```

---

## 📝 Quick Revision Cheat Sheet

### 🔵 Primary Group
- `primary` → Main accent (filled buttons, FAB)
- `onPrimary` → Text on primary
- `primaryContainer` → Soft accent (tonal buttons, chips)
- `onPrimaryContainer` → Text on primaryContainer

### 🟢 Secondary Group
- `secondary` → Supporting accent (filters, toggles)
- `onSecondary` → Text on secondary
- `secondaryContainer` → Soft support (nav drawer selected)
- `onSecondaryContainer` → Text on secondaryContainer

### 🟣 Tertiary Group
- `tertiary` → Third accent (badges, extras)
- `onTertiary` → Text on tertiary
- `tertiaryContainer` → Soft third accent
- `onTertiaryContainer` → Text on tertiaryContainer

### 🔴 Error Group
- `error` → Error state (validation errors)
- `onError` → Text on error
- `errorContainer` → Soft error (error banners)
- `onErrorContainer` → Text on errorContainer

### ⬜ Surface & Background
- `background` → Root canvas (≈ same as surface in M3)
- `onBackground` → Text on background
- `surface` → Component backgrounds (cards, sheets, bars)
- `onSurface` → **Default text color**
- `surfaceVariant` → Slightly contrasted fill (text fields)
- `onSurfaceVariant` → Secondary text (placeholders, subtitles)

### 📐 Outline
- `outline` → Borders, dividers (medium emphasis)
- `outlineVariant` → Subtle borders (low emphasis)

### 🔄 Inverse
- `inverseSurface` → Snackbar/tooltip background
- `inverseOnSurface` → Snackbar text
- `inversePrimary` → Snackbar action button

### 📦 Surface Containers (Depth without Shadows)
- `surfaceContainerLowest` → Deepest background layer
- `surfaceContainerLow` → Low emphasis layer
- `surfaceContainer` → Default card/dialog layer
- `surfaceContainerHigh` → Higher emphasis layer
- `surfaceContainerHighest` → Highest emphasis layer (input areas)
- `surfaceDim` → Dimmed background
- `surfaceBright` → Bright foreground

### 🔲 Scrim
- `scrim` → Semi-transparent overlay for modals

---

## Developer Rules of Thumb

1. **Never hardcode colors** → Always use `MaterialTheme.colorScheme.xxx`
2. **Always pair fill + content** → `primary` ↔ `onPrimary`, `surface` ↔ `onSurface`
3. **One primary action per screen** → Only one filled primary button; others use tonal/outlined
4. **Use containers for soft emphasis** → `primaryContainer` > `primary` for cards and chips
5. **Let M3 components handle colors** → Only override when you have a specific design need
6. **Test in both light AND dark mode** → Colors auto-adapt, but verify the result
7. **Support contrast levels** → Include medium/high contrast schemes for accessibility
8. **Prefer `surface` over `background`** → M3 recommends `surface` for most surfaces
9. **Use `onSurfaceVariant` for secondary text** → Not `onSurface` with manual alpha
10. **Use surface containers for depth** → Not `elevation` or hardcoded gray shades

---

> **Created for:** Interview prep & real-world Android development  
> **Design System:** Material 3 (Material You)  
> **Framework:** Jetpack Compose  
