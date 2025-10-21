## Compose UI Testing Standards

### Testing Framework
- **Framework:** Compose UI Test with JUnit4
- **Annotation:** `@HiltAndroidTest` for Hilt integration
- **Base Class:** Extend `BaseUITest` for common setup
- **Rule:** Use `createAndroidComposeRule` or `createComposeRule`

### Test Class Structure
```kotlin
@OptIn(ExperimentalTestApi::class)
@HiltAndroidTest
class MyViewTest : BaseUITest() {

    private lateinit var mockNavigator: CustomNavigator
    private lateinit var mockViewModel: MyViewModel
    private lateinit var uiState: MutableStateFlow<MyViewModel.UiState>

    @Before
    fun setup() {
        hiltRule.inject()

        mockNavigator = mockk<CustomNavigator>()
        every { mockNavigator.navigate(any<AppRoute>()) } returns Unit
        every { mockNavigator.goBack() } returns Unit

        uiState = MutableStateFlow(MyViewModel.UiState())

        mockViewModel = mockk<MyViewModel>()
        every { mockViewModel.uiState } returns uiState
        every { mockViewModel.onEvent(any()) } returns Unit
    }

    @After
    fun cleanup() {
        clearAllMocks()
    }
}
```

### Setting Content
```kotlin
private fun setupComposableContent(state: MyViewModel.UiState) {
    uiState.value = state

    composeTestRule.setContent {
        PalmaTheme {
            Surface {
                MyView(mockViewModel)
            }
        }
    }
}
```

### Test Naming
- **Descriptive Names:** Clear, readable test names describing the scenario
- **Pattern:** `[scenario]_[expectedBehavior]` or `when[Condition]_then[Behavior]`
- **Examples:**
  ```kotlin
  @Test
  fun userTapsBackButton_navigatesBack()

  @Test
  fun whenDeviceDisconnected_thenConnectionMessageDisplays()
  ```

### Finding UI Elements
- **Text:** `onNodeWithText(getString(R.string.text_id))`
- **Content Description:** `onNodeWithContentDescription("description")`
- **Test Tag:** `onNodeWithTag("tag")`

### String Resources
```kotlin
private fun getString(resId: Int): String =
    InstrumentationRegistry.getInstrumentation().targetContext.getString(resId)
```

### Assertions
- **Is Displayed:** `assertIsDisplayed()`
- **Does Not Exist:** `assertDoesNotExist()`
- **Is Enabled:** `assertIsEnabled()`
- **Is Not Enabled:** `assertIsNotEnabled()`

### User Interactions
- **Click:** `performClick()`
- **Text Input:** `performTextInput("text")`
- **Scroll:** `performScrollTo()`

### Wait and Synchronization
- **Wait for Idle:** `composeTestRule.waitForIdle()`
- **State Updates:** Update state then call `waitForIdle()`

### Given-When-Then Structure
```kotlin
@Test
fun userClicksButton_navigatesToNextScreen() {
    // Given: Initial state
    val state = MyViewModel.UiState(isReady = true)
    setupComposableContent(state)

    // When: User interaction
    composeTestRule.onNodeWithText("Next").performClick()

    // Then: Verify behavior
    verify { mockViewModel.onEvent(MyViewModel.UiEvent.NavigateNext) }
}
```

### Test Organization
- **Regions:** Use region comments to group related tests
  ```kotlin
  // region State Management Tests
  @Test
  fun stateTest1() { }

  @Test
  fun stateTest2() { }
  // endregion

  // region User Interaction Tests
  @Test
  fun interactionTest1() { }
  // endregion
  ```

### Mock Factory Patterns
- **Complex Objects:** Create factory functions for reusable mocks
  ```kotlin
  private fun createMockDevice(
      side: HiSide,
      deviceTypeName: String,
      isSupported: Boolean = true
  ): InternalDevice = mockk {
      every { this@mockk.side } returns side
      every { this@mockk.deviceTypeName } returns deviceTypeName
      every { this@mockk.isSupported() } returns isSupported
  }
  ```

### Initialization Helpers
- **Demo Setup:** Extract initialization into helper when needed
  ```kotlin
  private fun setupComposableContent(state: MyViewModel.UiState) {
      uiState.value = state

      composeTestRule.setContent {
          initDemo() // Optional: Initialize demo data
          PalmaTheme {
              MyView(mockViewModel)
          }
      }
  }
  ```

### Best Practices
- **Test User Flows:** Test complete user interactions, not just individual elements
- **Accessibility:** Verify content descriptions for accessibility
- **Error States:** Test error messages and alerts
- **Loading States:** Test loading indicators and progress states
- **Theme Wrapper:** Always wrap content in theme (e.g., `PalmaTheme`)
- **State Isolation:** Each test should set up its own state
- **Cleanup:** Clean up mocks and test state after each test
