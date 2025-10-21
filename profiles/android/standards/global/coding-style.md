## Coding Style

### Kotlin Style Guide
- **Official Style:** Follow official Kotlin coding conventions (`kotlin.code.style=official`)
- **Formatting:** Use default IntelliJ/Android Studio formatting
- **Line Length:** Keep lines under 120 characters when reasonable
- **Indentation:** 4 spaces for indentation (no tabs)

### Naming Conventions
- **Classes:** PascalCase (e.g., `UserViewModel`, `SettingsRepository`)
- **Functions:** camelCase (e.g., `getUserData`, `onButtonClick`)
- **Properties:** camelCase (e.g., `userName`, `isConnected`)
- **Constants:** UPPER_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`)
- **Private Members:** camelCase with underscore prefix for backing properties (e.g., `_uiState`)
- **Composables:** PascalCase like classes (e.g., `SettingsView`, `CustomButton`)

### Package Naming
- **Lowercase:** All lowercase, no underscores (e.g., `com.advancedbionics.palma.ui.settings`)
- **Descriptive:** Use descriptive names (e.g., `viewmodel`, `subview`, not `vm`, `sub`)

### File Organization
- **One Class Per File:** Each class in its own file (except sealed classes and small helper classes)
- **File Name:** File name must match the primary class name
- **Extensions:** Group related extension functions in `*Extensions.kt` files

### Import Organization
- **Wildcard Imports:** Avoid wildcard imports except for Compose
- **Unused Imports:** Remove unused imports
- **Order:** Android, AndroidX, Third-party, Java, Kotlin, Project

### Composable Functions
- **PascalCase:** Composable functions use PascalCase like classes
- **No Return Value:** Composables should not return values (return Unit)
- **Modifier Parameter:** Always include `modifier: Modifier = Modifier` parameter
- **Parameter Order:** REQUIRED: Follow parameter order (required → modifier → optional → content slots)
  - Required parameters first (no default values)
  - Modifier parameter next
  - Optional parameters with defaults
  - Content lambdas (trailing lambda syntax) last

```kotlin
@Composable
fun CustomButton(
    text: String,                        // Required
    onClick: () -> Unit,                 // Required
    modifier: Modifier = Modifier,       // Modifier
    enabled: Boolean = true,             // Optional
    content: @Composable () -> Unit = {} // Content slot
) {
    // Implementation
}
```

### Function Structure
- **Small Functions:** Keep functions focused and small
- **Single Responsibility:** Each function should do one thing
- **Max Parameters:** Limit to 5 parameters; use data classes for more
- **Default Values:** Use default values instead of overloads

### Null Safety
- **Avoid !!:** Use safe calls (?.) and elvis operator (?:) instead
- **Early Returns:** Use early returns for null checks
- **let/run/apply:** Use scope functions appropriately
```kotlin
// Prefer
val result = value?.let { processValue(it) } ?: defaultValue

// Over
val result = if (value != null) processValue(value) else defaultValue
```

### Comments
- **Copyright Headers:** Include copyright header at top of each file
```kotlin
/*
 * Copyright (c) 2025. Advanced Bionics
 */
```
- **KDoc:** Use KDoc for public APIs and complex logic
- **Inline Comments:** Use sparingly, explain "why" not "what"

### String Handling
- **String Templates:** Use string templates over concatenation
```kotlin
// Prefer
val message = "Hello, $userName!"

// Over
val message = "Hello, " + userName + "!"
```
- **Resources:** Always use string resources for user-facing text

### Collections
- **Immutability:** Prefer immutable collections (List, Map, Set)
- **Collection Functions:** Use Kotlin collection functions (map, filter, fold, etc.)

### Type Inference
- **Let Compiler Infer:** Let compiler infer obvious types
- **Explicit for Clarity:** Be explicit when type isn't obvious

### Best Practices
- **DRY Principle:** Don't repeat yourself
- **Meaningful Names:** Use descriptive, meaningful names
- **No Magic Numbers:** Use named constants instead
- **Early Returns:** Use guard clauses and early returns
- **Immutability:** Prefer val over var
- **Null Safety:** Leverage Kotlin's null safety features
