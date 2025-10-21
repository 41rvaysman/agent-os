## Jetpack Compose Component Best Practices

### Component Structure
- **Stateless by Default:** Create composables that receive state and callbacks as parameters
- **Two-Function Pattern:** Separate entry point (with ViewModel) from content composable
  ```kotlin
  @Composable
  fun MyView(viewModel: MyViewModel = hiltViewModel()) {
      val state by viewModel.uiState.collectAsStateWithLifecycle()
      MyViewContent(state = state, event = viewModel::onEvent)
  }

  @Composable
  private fun MyViewContent(
      state: MyViewModel.UiState,
      event: (MyViewModel.UiEvent) -> Unit
  ) {
      // UI implementation
  }
  ```

### State Management
- **Use StateFlow:** ViewModels should expose `StateFlow<UiState>` for state
- **collectAsStateWithLifecycle:** Always use `collectAsStateWithLifecycle()` instead of `collectAsState()`
- **UiState Data Class:** Define a data class for UI state within the ViewModel
- **UiEvent Sealed Class:** Define events as sealed class/interface for user interactions
- **State Hoisting:** Hoist state to the appropriate level, keep it as local as possible

### ViewModel Integration
- **Hilt ViewModel:** Use `@HiltViewModel` annotation and inject with `hiltViewModel()`
- **Constructor Injection:** Inject dependencies through ViewModel constructor
- **Event Methods:** Pass ViewModel's `onEvent` method reference to content composable
  ```kotlin
  MyViewContent(
      state = state,
      event = viewModel::onEvent
  )
  ```

### Naming Conventions
- **View Files:** `*View.kt` for composable screens
- **ViewModel Files:** `*ViewModel.kt` for view models
- **Nested State/Events:** Define UiState and UiEvent inside ViewModel class
- **Function Names:** Use descriptive names, prefix with component type when helpful

### Composable Organization
- **Top-Level Composable:** Entry point with ViewModel
- **Private Content Composable:** Main UI implementation
- **Helper Composables:** Extract reusable UI pieces into separate composables
- **Preview Composables:** REQUIRED: Add preview functions for new components

### Layout Best Practices
- **Scaffold Pattern:** Use `Scaffold` with `TopBar` for screen-level composables
- **Padding Values:** Respect `PaddingValues` from Scaffold
- **Material 3:** Use Material 3 components and theming

### Spacing
- **Use Spacing Data Class:** Always use `MaterialTheme.spacing.*` instead of hardcoded dp values
- **Available Spacing Values:**
  - `spacing0dp` (0.dp) - No spacing
  - `spacing4dp` (4.dp) - Extra extra small
  - `spacing8dp` (8.dp) - Extra small
  - `spacing16dp` (16.dp) - Small
  - `spacing24dp` (24.dp) - Medium
  - `spacing32dp` (32.dp) - Large
  - `spacing48dp` (48.dp) - Extra large
  - `spacing64dp` (64.dp) - Extra extra large
  - `spacing80dp` (80.dp) - Specific use cases
  - `spacing192dp` (192.dp) - Specific large gaps
- **Example:**
  ```kotlin
  @Composable
  fun MyContent() {
      Column(
          modifier = Modifier.padding(MaterialTheme.spacing.spacing16dp),
          verticalArrangement = Arrangement.spacedBy(MaterialTheme.spacing.spacing8dp)
      ) {
          Text("Content")
      }
  }
  ```
- **Don't:** Use hardcoded values like `16.dp` directly
- **Do:** Use `MaterialTheme.spacing.spacing16dp` for consistency

### Modifiers
- **Order Matters:** Apply modifiers in correct order (size → padding → visual effects)
- **Reusable Modifiers:** Extract complex modifier chains into extension functions
- **Semantic Modifiers:** Use semantics modifiers for accessibility and testing
  ```kotlin
  Modifier.semantics { contentDescription = "Button description" }
  ```

### Side Effects
- **LaunchedEffect:** For launching coroutines tied to composable lifecycle
- **SideEffect:** For non-suspend side effects that should run on every recomposition
- **DisposableEffect:** For effects that need cleanup
- **rememberCoroutineScope:** For event-driven coroutine launches

### Testing Support
- **Test Tags:** Use `Modifier.testTag()` or semantic properties for UI tests
- **Content Descriptions:** Provide content descriptions for accessibility and testing
- **Stable State:** Design state to be easily mockable for tests
- **Preview Annotations:** Include previews for visual verification

### Preview Functions
- **REQUIRED:** Add `@Preview` functions for all new components
- **Multiple States:** Create previews for different states (default, loading, error, empty)
- **Theme Wrapper:** Always wrap preview content in `PalmaTheme`
- **Naming:** Prefix preview functions with `Preview` (e.g., `PreviewCustomButton`, `PreviewCustomButtonDisabled`)
- **Parameters:** Use hardcoded or mock data for preview parameters
- **Light/Dark:** Consider adding `@Preview(uiMode = UI_MODE_NIGHT_YES)` for dark mode testing when applicable
- **Device Sizes:** Add device parameter for different screen sizes when relevant
- **Example:**
  ```kotlin
  @Preview(showBackground = true)
  @Composable
  private fun PreviewCustomButton() {
      PalmaTheme {
          CustomButton(
              text = "Click Me",
              onClick = {},
              enabled = true
          )
      }
  }

  @Preview(showBackground = true)
  @Composable
  private fun PreviewCustomButtonDisabled() {
      PalmaTheme {
          CustomButton(
              text = "Disabled",
              onClick = {},
              enabled = false
          )
      }
  }
  ```

### Performance
- **Remember Values:** Use `remember` for expensive calculations
- **Derived State:** Use `derivedStateOf` for computed state
- **Immutable Data:** Use immutable data classes for state
- **Avoid Recreating Lambdas:** Extract lambdas that don't capture state

### Common Patterns
- **Scroll State:** Use `rememberScrollState()` for scrollable content
- **String Resources:** Always use `stringResource(R.string.*)` for text
- **Conditional UI:** Use standard Kotlin if/when for conditional composition
- **Lists:** Use `LazyColumn`/`LazyRow` for scrollable lists
- **Dialogs and Sheets:** Manage visibility through state
