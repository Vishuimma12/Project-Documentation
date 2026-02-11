# 🧱 Mini-Project 2 — Domain Models

> **Objective:** Define all the core data models for Taskora. These models live in `domain/model/` and are **pure Kotlin** — no Android, no Room, no JSON annotations. This is the heart of Clean Architecture.

---

## 📖 Table of Contents

1. [Clean Architecture — The Big Picture](#1-clean-architecture--the-big-picture)
2. [Why a Separate Domain Layer?](#2-why-a-separate-domain-layer)
3. [Kotlin `data class` Deep Dive](#3-kotlin-data-class-deep-dive)
4. [Kotlin `enum class` Deep Dive](#4-kotlin-enum-class-deep-dive)
5. [File: Priority.kt (enum)](#5-file-prioritykt-enum)
6. [File: TaskStatus.kt (enum)](#6-file-taskstatuskt-enum)
7. [File: RecurrenceType.kt (enum)](#7-file-recurrencetypekt-enum)
8. [File: Tag.kt (data class)](#8-file-tagkt-data-class)
9. [File: Category.kt (data class)](#9-file-categorykt-data-class)
10. [File: RecurrenceRule.kt (data class)](#10-file-recurrencerulekt-data-class)
11. [File: Task.kt (data class) — The Star](#11-file-taskkt-data-class--the-star)
12. [Relationships Between Models](#12-relationships-between-models)
13. [Testing Your Models](#13-testing-your-models)
14. [java.time API & Desugaring](#14-javatime-api--desugaring)
15. [Checkpoint & Verification](#15-checkpoint--verification)
16. [Interview Questions & Answers](#16-interview-questions--answers)

---

## 1. Clean Architecture — The Big Picture

Clean Architecture divides your app into **3 layers**, each with a clear responsibility:

```
┌─────────────────────────────────────────────────┐
│              PRESENTATION LAYER                  │
│  (UI, Screens, ViewModels, Composables)          │
│  → What the USER sees and interacts with         │
├─────────────────────────────────────────────────┤
│                DOMAIN LAYER                      │
│  (Models, Use Cases, Repository Interfaces)      │
│  → The RULES of your app (pure business logic)   │
├─────────────────────────────────────────────────┤
│                 DATA LAYER                       │
│  (Room DB, Retrofit API, DataStore, Repository   │
│   Implementations)                               │
│  → WHERE data actually comes from                │
└─────────────────────────────────────────────────┘
```

### The Golden Rule: Dependencies Point INWARD

```
Presentation → depends on → Domain
Data          → depends on → Domain
Domain        → depends on → NOTHING ❌
```

The **Domain layer never imports** anything from Data or Presentation. It's the innermost circle — it's pure Kotlin.

---

## 2. Why a Separate Domain Layer?

### Without Domain Layer (❌ Common Beginner Mistake)

```kotlin
// ❌ Room Entity used directly in ViewModel and UI
@Entity(tableName = "tasks")
data class Task(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "task_title") val title: String,
    val priority: String  // stored as String in DB
)

// ❌ Now your UI knows about Room annotations
// ❌ If you change your database, UI breaks
// ❌ Can't use this in a KMM (Kotlin Multiplatform) project
```

### With Domain Layer (✅ Clean)

```kotlin
// domain/model/Task.kt — Pure Kotlin, no annotations
data class Task(
    val id: String,
    val title: String,
    val priority: Priority  // uses your enum, type-safe!
)

// data/local/entities/TaskEntity.kt — Room-specific
@Entity(tableName = "tasks")
data class TaskEntity(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "task_title") val title: String,
    val priority: String
)

// Mapper converts between the two
fun TaskEntity.toDomain() = Task(id, title, Priority.valueOf(priority))
fun Task.toEntity() = TaskEntity(id, title, priority.name)
```

### Benefits

| Benefit         | Explanation                                       |
| --------------- | ------------------------------------------------- |
| **Testability** | Domain models need no Android framework to test   |
| **Flexibility** | Change your database? Only the Data layer changes |
| **Readability** | Models are clean, no annotation clutter           |
| **Reusability** | Same models work on Android, iOS (KMM), Backend   |
| **Separation**  | Each layer has one job, one reason to change      |

---

## 3. Kotlin `data class` Deep Dive

### What is a `data class`?

A `data class` is a class whose **primary purpose is to hold data**. Kotlin auto-generates useful functions for you.

```kotlin
data class User(
    val name: String,
    val age: Int,
    val email: String
)
```

### What Kotlin Generates Automatically

```kotlin
// You get ALL of these for FREE just by writing "data class":

// 1. equals() — compares all properties
val user1 = User("John", 25, "john@email.com")
val user2 = User("John", 25, "john@email.com")
println(user1 == user2)  // true ✅ (compares ALL fields)

// 2. hashCode() — consistent with equals (needed for Sets, Maps)
println(user1.hashCode() == user2.hashCode())  // true

// 3. toString() — readable output
println(user1)  // User(name=John, age=25, email=john@email.com)

// 4. copy() — create modified copies without mutating the original
val user3 = user1.copy(age = 26)  // Only age changes
println(user3)  // User(name=John, age=26, email=john@email.com)

// 5. componentN() — destructuring
val (name, age, email) = user1
println("$name is $age years old")  // John is 25 years old
```

### `data class` Rules

| Rule                                                          | Example                        |
| ------------------------------------------------------------- | ------------------------------ |
| Must have at least **one parameter** in primary constructor   | `data class Foo(val x: Int)` ✅ |
| Parameters should be `val` or `var`                           | `val` preferred (immutable)    |
| Cannot be `abstract`, `open`, `sealed`, or `inner`            | Always `final`                 |
| Properties in constructor body are **NOT** in equals/toString | See example below              |

```kotlin
data class Task(
    val id: String,       // ✅ Included in equals(), toString(), copy()
    val title: String     // ✅ Included
) {
    var timestamp: Long = 0  // ❌ NOT included in equals/toString
}
```

### `val` vs `var` — Immutability Matters

```kotlin
// ✅ GOOD — Immutable (val). Create new instance to change.
data class Task(val title: String, val isCompleted: Boolean)

val task = Task("Buy milk", false)
val completedTask = task.copy(isCompleted = true)  // New object, original unchanged

// ❌ BAD — Mutable (var). Can be changed from anywhere.
data class Task(var title: String, var isCompleted: Boolean)

val task = Task("Buy milk", false)
task.isCompleted = true  // Mutated in place — hard to track in large apps
```

> **Rule of Thumb:** Always use `val` in domain models. Use `copy()` to create modified versions. This prevents bugs in multi-threaded/reactive environments.

---

## 4. Kotlin `enum class` Deep Dive

### What is an `enum class`?

An enum represents a **fixed set of constants**. If a value can only be one of a known set of options, use an enum.

### Basic Enum

```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}

// Usage
val dir = Direction.NORTH
println(dir.name)     // "NORTH" (String name)
println(dir.ordinal)  // 0 (position index)
```

### Enum with Properties

```kotlin
enum class Priority(
    val displayName: String,
    val level: Int,
    val colorHex: String
) {
    LOW("Low", 1, "#34A853"),
    MEDIUM("Medium", 2, "#FBBC04"),
    HIGH("High", 3, "#FA903E"),
    URGENT("Urgent", 4, "#D93025");

    // Custom function
    fun isHighPriority(): Boolean = level >= 3
}

// Usage
val p = Priority.HIGH
println(p.displayName)      // "High"
println(p.level)            // 3
println(p.isHighPriority()) // true
println(p.colorHex)         // "#FA903E"
```

### Useful Enum Functions

```kotlin
// Get all values
val allPriorities = Priority.values()  // Array of all 4 values

// Find by name (throws if not found)
val p1 = Priority.valueOf("HIGH")  // Priority.HIGH

// Safe find by name
val p2 = Priority.values().find { it.name == "HIGH" }  // Priority.HIGH or null

// Iterate
Priority.values().forEach { println("${it.displayName}: ${it.colorHex}") }

// Use in when (compiler checks all cases!)
fun getLabel(p: Priority): String = when (p) {
    Priority.LOW -> "Take your time"
    Priority.MEDIUM -> "Do it soon"
    Priority.HIGH -> "Do it now"
    Priority.URGENT -> "🔥 DO IT NOW!"
    // No 'else' needed — compiler knows all cases are covered
}
```

---

## 5. File: Priority.kt (enum)

```kotlin
// domain/model/Priority.kt
package com.yourname.taskora.domain.model

/**
 * Represents the urgency level of a task.
 *
 * @property displayName Human-readable label for UI display
 * @property level Numeric level for sorting (1=lowest, 4=highest)
 * @property colorHex Hex color string for UI indicators
 */
enum class Priority(
    val displayName: String,
    val level: Int,
    val colorHex: String
) {
    LOW(
        displayName = "Low",
        level = 1,
        colorHex = "#34A853"      // Green
    ),
    MEDIUM(
        displayName = "Medium",
        level = 2,
        colorHex = "#FBBC04"      // Yellow
    ),
    HIGH(
        displayName = "High",
        level = 3,
        colorHex = "#FA903E"      // Orange
    ),
    URGENT(
        displayName = "Urgent",
        level = 4,
        colorHex = "#D93025"      // Red
    );

    /**
     * Returns true if this priority needs immediate attention (HIGH or URGENT)
     */
    fun isHighPriority(): Boolean = level >= 3

    companion object {
        /**
         * Returns the default priority for new tasks
         */
        fun default(): Priority = MEDIUM

        /**
         * Safely finds a Priority by name, returns default if not found
         */
        fun fromName(name: String): Priority {
            return values().find { it.name == name } ?: default()
        }
    }
}
```

### Why Named Parameters in Enum Entries?

```kotlin
// ❌ Hard to read — what do these values mean?
LOW("Low", 1, "#34A853")

// ✅ Self-documenting
LOW(
    displayName = "Low",
    level = 1,
    colorHex = "#34A853"
)
```

---

## 6. File: TaskStatus.kt (enum)

```kotlin
// domain/model/TaskStatus.kt
package com.yourname.taskora.domain.model

/**
 * Represents the current state of a task in its lifecycle.
 *
 * State transitions:
 *   TODO → IN_PROGRESS → COMPLETED
 *   TODO → CANCELLED
 *   IN_PROGRESS → CANCELLED
 *   COMPLETED → TODO (reopen)
 */
enum class TaskStatus(
    val displayName: String,
    val iconEmoji: String
) {
    TODO(
        displayName = "To Do",
        iconEmoji = "📋"
    ),
    IN_PROGRESS(
        displayName = "In Progress",
        iconEmoji = "🔄"
    ),
    COMPLETED(
        displayName = "Completed",
        iconEmoji = "✅"
    ),
    CANCELLED(
        displayName = "Cancelled",
        iconEmoji = "❌"
    );

    /**
     * Check if the task is in a terminal state (no more work needed)
     */
    fun isTerminal(): Boolean = this == COMPLETED || this == CANCELLED

    /**
     * Check if the task is still active (needs attention)
     */
    fun isActive(): Boolean = this == TODO || this == IN_PROGRESS

    companion object {
        fun default(): TaskStatus = TODO
    }
}
```

### Usage Pattern

```kotlin
val status = TaskStatus.IN_PROGRESS

// Smart filtering
val activeTasks = allTasks.filter { it.status.isActive() }
val doneTasks = allTasks.filter { it.status.isTerminal() }

// Display in UI
Text(text = "${status.iconEmoji} ${status.displayName}")
// Output: 🔄 In Progress
```

---

## 7. File: RecurrenceType.kt (enum)

```kotlin
// domain/model/RecurrenceType.kt
package com.yourname.taskora.domain.model

/**
 * Defines how frequently a recurring task repeats.
 */
enum class RecurrenceType(val displayName: String) {
    DAILY("Daily"),
    WEEKLY("Weekly"),
    MONTHLY("Monthly"),
    YEARLY("Yearly");
}
```

---

## 8. File: Tag.kt (data class)

```kotlin
// domain/model/Tag.kt
package com.yourname.taskora.domain.model

/**
 * A label that can be attached to multiple tasks for flexible categorization.
 * Unlike categories (one per task), tags support many-to-many relationships.
 *
 * Example: A task "Buy birthday cake" might have tags: "shopping", "birthday"
 *
 * @property id Unique identifier (UUID format)
 * @property name Display name (e.g., "urgent", "work", "personal")
 * @property colorHex Hex color for UI badge (e.g., "#FF5722")
 */
data class Tag(
    val id: String,
    val name: String,
    val colorHex: String
)
```

### Usage

```kotlin
val workTag = Tag(
    id = "tag-001",
    name = "Work",
    colorHex = "#1A73E8"
)

val personalTag = Tag(
    id = "tag-002",
    name = "Personal",
    colorHex = "#34A853"
)

// Tags are used inside Task as a List
val myTags = listOf(workTag, personalTag)
```

---

## 9. File: Category.kt (data class)

```kotlin
// domain/model/Category.kt
package com.yourname.taskora.domain.model

/**
 * A group that a task belongs to. Each task has AT MOST one category.
 * Categories provide high-level organization.
 *
 * Examples: "Work", "Personal", "Shopping", "Health", "Study"
 *
 * @property id Unique identifier (UUID format)
 * @property name Display name
 * @property colorHex Hex color for visual indicators
 * @property iconName Material icon name (e.g., "work", "home", "shopping_cart")
 */
data class Category(
    val id: String,
    val name: String,
    val colorHex: String,
    val iconName: String
)
```

### Category vs Tag — What's the Difference?

| Feature          | Category                            | Tag                                    |
| ---------------- | ----------------------------------- | -------------------------------------- |
| **Relationship** | One-to-many (task has ONE category) | Many-to-many (task has MANY tags)      |
| **Purpose**      | Primary grouping                    | Cross-cutting labels                   |
| **Example**      | "Work" category                     | "urgent" + "meeting" tags              |
| **Analogy**      | A folder (file lives in one folder) | Stickers (file can have many stickers) |

---

## 10. File: RecurrenceRule.kt (data class)

```kotlin
// domain/model/RecurrenceRule.kt
package com.yourname.taskora.domain.model

import java.time.LocalDate

/**
 * Defines the repetition pattern for a recurring task.
 *
 * Examples:
 *   - "Every day" → RecurrenceRule(DAILY, 1, null)
 *   - "Every 2 weeks" → RecurrenceRule(WEEKLY, 2, null)
 *   - "Every month until Dec 2025" → RecurrenceRule(MONTHLY, 1, LocalDate.of(2025,12,31))
 *
 * @property type How the task repeats (daily, weekly, monthly, yearly)
 * @property interval Every N periods (1 = every time, 2 = every other, etc.)
 * @property endDate Optional: when the recurrence stops (null = repeats forever)
 */
data class RecurrenceRule(
    val type: RecurrenceType,
    val interval: Int = 1,
    val endDate: LocalDate? = null
)
```

### Usage

```kotlin
// Repeats every day forever
val dailyHabit = RecurrenceRule(type = RecurrenceType.DAILY)

// Repeats every 2 weeks until June 2026
val biweeklyMeeting = RecurrenceRule(
    type = RecurrenceType.WEEKLY,
    interval = 2,
    endDate = LocalDate.of(2026, 6, 30)
)
```

---

## 11. File: Task.kt (data class) — The Star

This is the **main model** of the entire application.

```kotlin
// domain/model/Task.kt
package com.yourname.taskora.domain.model

import java.time.LocalDateTime

/**
 * Core domain model representing a task in the Taskora app.
 * This is the SINGLE SOURCE OF TRUTH for what a "task" is in the business logic.
 *
 * NOTE: This class has ZERO Android dependencies. It's pure Kotlin.
 *
 * @property id Unique identifier (UUID string)
 * @property title Task title (required, max 100 chars)
 * @property description Optional detailed description
 * @property priority Urgency level (LOW, MEDIUM, HIGH, URGENT)
 * @property status Current lifecycle state (TODO, IN_PROGRESS, COMPLETED, CANCELLED)
 * @property categoryId ID of the category this task belongs to (nullable)
 * @property tags List of tags attached to this task
 * @property dueDate When the task is due (nullable for tasks without deadlines)
 * @property reminderTime When to send a notification (nullable)
 * @property recurrenceRule Repetition pattern (nullable for one-time tasks)
 * @property createdAt When the task was first created
 * @property updatedAt When the task was last modified
 * @property isCompleted Convenience flag derived from status
 * @property isSynced Whether this task has been synced with the remote server
 */
data class Task(
    val id: String,
    val title: String,
    val description: String? = null,
    val priority: Priority = Priority.MEDIUM,
    val status: TaskStatus = TaskStatus.TODO,
    val categoryId: String? = null,
    val tags: List<Tag> = emptyList(),
    val dueDate: LocalDateTime? = null,
    val reminderTime: LocalDateTime? = null,
    val recurrenceRule: RecurrenceRule? = null,
    val createdAt: LocalDateTime = LocalDateTime.now(),
    val updatedAt: LocalDateTime = LocalDateTime.now(),
    val isCompleted: Boolean = false,
    val isSynced: Boolean = false
) {
    /**
     * Check if the task is overdue (past due date and not completed)
     */
    fun isOverdue(): Boolean {
        return dueDate != null &&
               dueDate.isBefore(LocalDateTime.now()) &&
               !isCompleted
    }

    /**
     * Check if the task is due today
     */
    fun isDueToday(): Boolean {
        return dueDate != null &&
               dueDate.toLocalDate() == java.time.LocalDate.now()
    }

    /**
     * Check if the task has a reminder set
     */
    fun hasReminder(): Boolean = reminderTime != null

    /**
     * Check if the task is recurring
     */
    fun isRecurring(): Boolean = recurrenceRule != null
}
```

### Why Default Parameter Values?

```kotlin
// Without defaults — must provide EVERYTHING every time
val task = Task(
    id = "1",
    title = "Buy milk",
    description = null,          // tedious
    priority = Priority.MEDIUM,  // tedious
    status = TaskStatus.TODO,    // tedious
    categoryId = null,           // tedious
    tags = emptyList(),          // tedious
    dueDate = null,              // tedious
    reminderTime = null,         // tedious
    recurrenceRule = null,       // tedious
    createdAt = LocalDateTime.now(),  // tedious
    updatedAt = LocalDateTime.now(),  // tedious
    isCompleted = false,         // tedious
    isSynced = false             // tedious
)

// With defaults — only provide what you need
val task = Task(
    id = "1",
    title = "Buy milk"
)
// Everything else uses sensible defaults! Much cleaner.
```

### Using `copy()` for State Changes

```kotlin
val task = Task(id = "1", title = "Buy milk")

// Mark as completed
val completedTask = task.copy(
    status = TaskStatus.COMPLETED,
    isCompleted = true,
    updatedAt = LocalDateTime.now()
)
// Original 'task' is UNCHANGED — immutability!

// Change priority
val urgentTask = task.copy(priority = Priority.URGENT)

// Add tags
val taggedTask = task.copy(
    tags = task.tags + Tag("t1", "Shopping", "#FF5722")
)
```

---

## 12. Relationships Between Models

```
Task
 ├── has ONE Priority (enum, embedded — not a separate entity)
 ├── has ONE TaskStatus (enum, embedded)
 ├── belongs to ONE Category? (via categoryId, nullable)
 ├── has MANY Tags (List<Tag>, many-to-many in DB)
 └── may have ONE RecurrenceRule? (embedded, nullable)
```

### How Relationships Work in Code

```kotlin
// Creating a complete Task with all relationships
val workCategory = Category("cat-1", "Work", "#1A73E8", "work")
val urgentTag = Tag("tag-1", "Urgent", "#D93025")
val meetingTag = Tag("tag-2", "Meeting", "#7B1FA2")

val myTask = Task(
    id = "task-001",
    title = "Prepare quarterly report",
    description = "Q4 2025 financial summary for the board",
    priority = Priority.HIGH,
    status = TaskStatus.IN_PROGRESS,
    categoryId = workCategory.id,             // Links to category by ID
    tags = listOf(urgentTag, meetingTag),      // Multiple tags
    dueDate = LocalDateTime.of(2026, 2, 15, 17, 0),
    reminderTime = LocalDateTime.of(2026, 2, 15, 9, 0),
    recurrenceRule = null                      // One-time task
)

// Checking relationships
println(myTask.priority.displayName)    // "High"
println(myTask.status.isActive())       // true
println(myTask.tags.map { it.name })    // [Urgent, Meeting]
println(myTask.isOverdue())             // depends on current time
println(myTask.isDueToday())            // depends on current date
```

---

## 13. Testing Your Models

Even without a test framework, you can verify your models work correctly in a `main()` function:

```kotlin
// Quick verification test
fun main() {
    // Test 1: Create a minimal task
    val task1 = Task(id = "1", title = "Test task")
    println("✅ Task created: $task1")
    assert(task1.priority == Priority.MEDIUM) { "Default priority should be MEDIUM" }
    assert(task1.status == TaskStatus.TODO) { "Default status should be TODO" }
    assert(task1.tags.isEmpty()) { "Default tags should be empty" }

    // Test 2: copy() works
    val task2 = task1.copy(priority = Priority.URGENT, isCompleted = true)
    assert(task2.priority == Priority.URGENT)
    assert(task2.isCompleted)
    assert(task1.isCompleted == false) { "Original must be unchanged" }
    println("✅ copy() works correctly")

    // Test 3: equals() works
    val task3 = Task(id = "1", title = "Test task")
    assert(task1 == task3) { "Tasks with same data should be equal" }
    println("✅ equals() works correctly")

    // Test 4: Enum works
    assert(Priority.HIGH.isHighPriority())
    assert(!Priority.LOW.isHighPriority())
    assert(Priority.fromName("INVALID") == Priority.MEDIUM) // fallback
    println("✅ Priority enum works correctly")

    // Test 5: TaskStatus transitions
    assert(TaskStatus.TODO.isActive())
    assert(TaskStatus.COMPLETED.isTerminal())
    assert(!TaskStatus.IN_PROGRESS.isTerminal())
    println("✅ TaskStatus enum works correctly")

    // Test 6: Overdue check
    val overdueTask = Task(
        id = "2",
        title = "Overdue task",
        dueDate = LocalDateTime.of(2020, 1, 1, 0, 0)  // Past date
    )
    assert(overdueTask.isOverdue()) { "Task with past due date should be overdue" }
    println("✅ isOverdue() works correctly")

    println("\n🎉 All domain model tests pass!")
}
```

---

## 14. java.time API & Desugaring

### Why `java.time.LocalDateTime` instead of `java.util.Date`?

| Feature           | `java.util.Date` (Old)        | `java.time.LocalDateTime` (Modern) |
| ----------------- | ----------------------------- | ---------------------------------- |
| Mutability        | ❌ Mutable                     | ✅ Immutable                        |
| Thread-safe       | ❌ No                          | ✅ Yes                              |
| Timezone handling | ❌ Confusing                   | ✅ Clear & explicit                 |
| API design        | ❌ Awkward (months 0-indexed!) | ✅ Intuitive                        |
| Available since   | Java 1.0                      | Java 8 / Android 26                |

### The Problem on Older Devices

`java.time` requires **API 26 (Android 8.0)**. If your `minSdk < 26`, you need **desugaring** — a Gradle plugin that backports `java.time` to older devices.

### How to Enable Desugaring

```kotlin
// app/build.gradle.kts
android {
    compileOptions {
        // Enable core library desugaring
        isCoreLibraryDesugaringEnabled = true
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

> If your `minSdk = 26` (as we set in Mini-Project 1), you **don't need desugaring**. But it's good to know about it.

### Common `java.time` Usage

```kotlin
import java.time.LocalDateTime
import java.time.LocalDate
import java.time.LocalTime
import java.time.format.DateTimeFormatter

// Current date/time
val now = LocalDateTime.now()               // 2026-02-11T22:30:00
val today = LocalDate.now()                 // 2026-02-11
val currentTime = LocalTime.now()           // 22:30:00

// Creating specific dates
val date = LocalDateTime.of(2026, 3, 15, 14, 30)  // March 15, 2026 at 2:30 PM

// Formatting for display
val formatter = DateTimeFormatter.ofPattern("MMM dd, yyyy · hh:mm a")
val formatted = date.format(formatter)  // "Mar 15, 2026 · 02:30 PM"

// Comparisons
val isFuture = date.isAfter(LocalDateTime.now())    // true
val isPast = date.isBefore(LocalDateTime.now())     // false

// Adding time
val tomorrow = today.plusDays(1)
val nextWeek = today.plusWeeks(1)
val inTwoHours = now.plusHours(2)
```

---

## 15. Checkpoint & Verification

### Folder Structure After This Mini-Project

```
domain/
└── model/
    ├── Priority.kt         (enum — 4 values with properties)
    ├── TaskStatus.kt       (enum — 4 states with helpers)
    ├── RecurrenceType.kt   (enum — 4 frequency options)
    ├── Tag.kt              (data class — id, name, colorHex)
    ├── Category.kt         (data class — id, name, colorHex, iconName)
    ├── RecurrenceRule.kt   (data class — type, interval, endDate)
    └── Task.kt             (data class — 14 fields, the core model)
```

### Checklist

| Check                      | How to Verify                                      |
| -------------------------- | -------------------------------------------------- |
| ✅ All 7 files compile      | Build → Rebuild Project                            |
| ✅ No Android imports       | Grep for `import android` — should find nothing    |
| ✅ No Room/Retrofit imports | Grep for `@Entity`, `@SerializedName` — nothing    |
| ✅ Can create Task instance | Write a `main()` function and instantiate          |
| ✅ copy() works             | Modify a field and verify original is unchanged    |
| ✅ Enum operations work     | `valueOf()`, `values()`, custom functions all work |

---

## 16. Interview Questions & Answers

### Q1: What is a `data class` in Kotlin and what does the compiler generate?

**Answer:**
A `data class` is a class primarily used to hold data. The Kotlin compiler automatically generates five functions based on the properties declared in the **primary constructor**:

1. **`equals()`** — structural equality comparison of all constructor properties
2. **`hashCode()`** — consistent with equals, needed for `HashMap`/`HashSet`
3. **`toString()`** — readable string: `"ClassName(prop1=val1, prop2=val2)"`
4. **`copy()`** — creates a new instance with optionally modified properties
5. **`componentN()`** — enables destructuring declarations (`val (a, b) = obj`)

Properties declared **inside the class body** (not the constructor) are excluded from all generated functions.

---

### Q2: What is Clean Architecture and why does the Domain layer have no Android dependencies?

**Answer:**
Clean Architecture separates an app into layers with **inward-pointing dependencies**:
- **Presentation** (outermost) → knows about Domain
- **Data** (outermost) → knows about Domain
- **Domain** (innermost) → knows about **nothing**

The Domain layer has no Android dependencies because:
1. **Testability** — domain models can be unit-tested without Android emulator or instrumentation
2. **Portability** — same models work on any platform (Android, iOS via KMM, server-side)
3. **Stability** — if Android APIs change, domain logic doesn't break
4. **Single Responsibility** — domain defines WHAT your app does, not HOW (the framework handles HOW)

---

### Q3: What is the difference between `enum class` and `sealed class`? When to use which?

**Answer:**

| Feature         | `enum class`                                | `sealed class`                               |
| --------------- | ------------------------------------------- | -------------------------------------------- |
| **Instances**   | Fixed set of **singletons**                 | Subclasses can hold **different data**       |
| **State**       | All entries have the **same properties**    | Each subclass can have **unique properties** |
| **When to use** | Fixed options (Priority, Status, Direction) | Varying states (Result, UI State, Event)     |

```kotlin
// Enum — all have same shape
enum class Priority(val level: Int) {
    LOW(1), MEDIUM(2), HIGH(3)
}

// Sealed — different shapes
sealed class Result {
    data class Success(val data: List<Task>) : Result()
    data class Error(val message: String, val code: Int) : Result()
    object Loading : Result()
}
```

**Use enum** when values are constants. **Use sealed class** when each variant carries different data.

---

### Q4: Why use `String` for IDs instead of `Int`/`Long`?

**Answer:**
Using `String` (typically UUID) for IDs has several advantages over auto-increment integers:

1. **Offline generation** — UUIDs can be generated on the client without a server, essential for offline-first apps
2. **No conflicts during sync** — two devices can create tasks with different UUIDs; auto-increment IDs would collide
3. **Security** — UUIDs are not guessable (unlike sequential `/task/1`, `/task/2`)
4. **Scalability** — works across distributed systems and multiple databases

```kotlin
import java.util.UUID

val id = UUID.randomUUID().toString()
// "550e8400-e29b-41d4-a716-446655440000"
```

**Trade-off:** UUIDs use more storage (36 chars vs 4 bytes for Int), and sorting is not sequential. But for most apps, the benefits outweigh the costs.

---

### Q5: What is the difference between `null`, `emptyList()`, and `listOf()` for default values?

**Answer:**
```kotlin
data class Task(
    val description: String? = null,    // ABSENT — no description at all
    val tags: List<Tag> = emptyList()   // PRESENT but empty — has a tag list, just no items
)
```

- **`null`** means the value **doesn't exist** (e.g., no due date set)
- **`emptyList()`** means the value **exists but is empty** (e.g., task has a tag list, but no tags added yet)
- **`listOf(item1, item2)`** means the value exists with initial items

**Why this matters:** Using `null` for "no tags" forces null checks everywhere (`task.tags?.forEach`). Using `emptyList()` is safer — you can always call `task.tags.forEach` without null checks.

---

### Q6: Explain `copy()` and why it's important for immutable state in Android.

**Answer:**
`copy()` creates a new instance of a `data class` with optionally modified properties while keeping the original unchanged.

```kotlin
val original = Task(id = "1", title = "Buy milk", isCompleted = false)
val modified = original.copy(isCompleted = true)

// original.isCompleted → false (UNCHANGED)
// modified.isCompleted → true  (NEW object)
```

**Why it matters for Android/Compose:**
- Compose uses **state comparison** to decide when to recompose. If you mutate an object in place, Compose can't detect the change
- `copy()` creates a new reference, which triggers recomposition
- It enables **unidirectional data flow**: ViewModel creates new state objects, UI observes them

```kotlin
// In ViewModel
_uiState.value = _uiState.value.copy(isLoading = true)
// Compose detects the new state object and recomposes
```

---

### Q7: What is destructuring in Kotlin and how does it relate to `data class`?

**Answer:**
Destructuring lets you extract multiple values from an object into separate variables using `componentN()` functions that `data class` auto-generates.

```kotlin
data class Task(val id: String, val title: String, val priority: Priority)

// Destructuring
val task = Task("1", "Buy milk", Priority.LOW)
val (id, title, priority) = task  // Uses component1(), component2(), component3()

// Common use: in lambdas
tasks.forEach { (id, title, _) ->  // _ ignores priority
    println("$id: $title")
}

// In Maps
val map = mapOf("key1" to "value1")
for ((key, value) in map) {
    println("$key = $value")
}
```

Order matters — `component1()` corresponds to the **first** constructor parameter.

---

### Q8: What is the difference between a `data class` and a regular `class`?

**Answer:**

```kotlin
// Regular class
class TaskA(val id: String, val title: String)

val a1 = TaskA("1", "Buy milk")
val a2 = TaskA("1", "Buy milk")
println(a1 == a2)       // false ❌ (compares references)
println(a1.toString())  // "TaskA@3f99bd52" (useless)
// a1.copy() → compile error (doesn't exist)

// Data class
data class TaskB(val id: String, val title: String)

val b1 = TaskB("1", "Buy milk")
val b2 = TaskB("1", "Buy milk")
println(b1 == b2)       // true ✅ (compares values)
println(b1.toString())  // "TaskB(id=1, title=Buy milk)" (useful)
val b3 = b1.copy(title = "Buy bread")  // works!
```

**Use `data class`** when the class exists to hold data. **Use regular `class`** when the class has behavior/identity beyond its data (e.g., ViewModel, Repository).

---

### Q9: Can a `data class` have methods? What's the best practice?

**Answer:**
Yes, data classes can have methods, but keep them limited to **derived computations** from the data:

```kotlin
data class Task(val dueDate: LocalDateTime?, val isCompleted: Boolean) {
    // ✅ GOOD — Derived from existing properties, no side effects
    fun isOverdue(): Boolean = dueDate?.isBefore(LocalDateTime.now()) == true && !isCompleted

    // ❌ BAD — Side effect (saving to database). This belongs in a Repository.
    fun save() { database.insert(this) }

    // ❌ BAD — Complex business logic. This belongs in a UseCase.
    fun scheduleReminder() { alarmManager.setAlarm(this) }
}
```

**Rule:** Domain models should be **anemic** (data holders with simple derived getters). Business logic goes in Use Cases. Data operations go in Repositories.

---

### Q10: Why do we use `val` instead of `var` in domain models?

**Answer:**
Using `val` (immutable) instead of `var` (mutable) ensures:

1. **Thread safety** — multiple coroutines can read without synchronization
2. **Predictability** — once created, the object never changes unexpectedly
3. **Compose compatibility** — Compose needs new objects to detect state changes
4. **Debugging** — you always know when state changed (it's wherever `copy()` was called)
5. **Functional style** — transforms create new objects (`task.copy(status = COMPLETED)`) instead of mutating in place

```kotlin
// ❌ Mutable — dangerous
data class Task(var status: TaskStatus)
task.status = TaskStatus.COMPLETED  // Who changed it? When? Hard to trace.

// ✅ Immutable — safe
data class Task(val status: TaskStatus)
val completed = task.copy(status = TaskStatus.COMPLETED)  // Explicit, traceable.
```

---

> **✅ Mini-Project 2 Complete!** You've defined all your domain models as pure Kotlin. These are the foundation every other layer builds upon. Next up: **Mini-Project 3 — Room Database** where you'll create database entities that map to these domain models.
