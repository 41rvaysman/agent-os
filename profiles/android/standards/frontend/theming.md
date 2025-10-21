## Theming and Design System

### Theme Setup
- **Material 3:** Use Material 3 components and theming
- **PalmaTheme:** Wrap all UI in project's `PalmaTheme` composable
- **Light Theme Only:** Currently supports light theme only

### Theme Wrapper
```kotlin
@Composable
fun MyScreen() {
    PalmaTheme {
        Surface {
            // Screen content
        }
    }
}
```

### Color System
- **MaterialTheme.colorScheme:** Access colors via Material theme
- **Primary:** `MaterialTheme.colorScheme.primary`
- **Background:** `MaterialTheme.colorScheme.background`
- **OnPrimary:** `MaterialTheme.colorScheme.onPrimary`
- **Custom Colors:** Define in `Color.kt` (e.g., `neutral02`, `neutral04`, `alert`)

### Custom Color Usage
```kotlin
import com.advancedbionics.palma.ui.theme.neutral02
import com.advancedbionics.palma.ui.theme.alert

@Composable
fun MyComponent() {
    Box(
        modifier = Modifier.background(neutral02)
    )

    if (hasError) {
        Text(
            text = "Error",
            color = alert
        )
    }
}
```

### Typography
- **MaterialTheme.typography:** Access text styles via Material theme
- **Display:** `MaterialTheme.typography.displayLarge/Medium/Small`
- **Headline:** `MaterialTheme.typography.headlineLarge/Medium/Small`
- **Title:** `MaterialTheme.typography.titleLarge/Medium/Small`
- **Body:** `MaterialTheme.typography.bodyLarge/Medium/Small`
- **Label:** `MaterialTheme.typography.labelLarge/Medium/Small`

### Spacing System
- **MaterialTheme.spacing:** Access spacing via composition local
- **Predefined Spacings:** `spacing4dp`, `spacing8dp`, `spacing16dp`, `spacing24dp`, `spacing32dp`, etc.

### Spacing Usage
```kotlin
@Composable
fun SpacingExample() {
    Column(
        modifier = Modifier.padding(MaterialTheme.spacing.spacing16dp)
    ) {
        Text("Item 1")
        SpacerH(MaterialTheme.spacing.spacing8dp)
        Text("Item 2")
    }
}
```

### Custom Spacer Components
- **SpacerH:** Vertical spacer (height)
- **SpacerW:** Horizontal spacer (width)
```kotlin
SpacerH(MaterialTheme.spacing.spacing16dp) // Vertical space
SpacerW(MaterialTheme.spacing.spacing8dp)  // Horizontal space
```

### Best Practices
- **Always Use Theme:** Never hardcode colors, sizes, or fonts
- **Semantic Colors:** Use semantic color names from Material (primary, error) not literal (blue, red)
- **Typography Scale:** Use predefined typography styles, don't create custom TextStyles inline
- **Spacing Consistency:** Use spacing system for consistent margins and padding
- **Surface Container:** Wrap content in `Surface` for proper Material theming
- **Preview Theme:** Always wrap previews in `PalmaTheme`
