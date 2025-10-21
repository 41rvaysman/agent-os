## Unit Testing Standards

### Testing Framework
- **Framework:** JUnit 4
- **Mocking:** MockK for Kotlin
- **Coroutines:** kotlinx-coroutines-test
- **Base Class:** Extend `BaseTest` for common test setup

### Test Class Structure
```kotlin
@ExperimentalCoroutinesApi
class MyViewModelTest : BaseTest() {

    @MockK
    private lateinit var mockNavigator: CustomNavigator

    @MockK
    private lateinit var mockRepository: MyRepository

    private lateinit var viewModel: MyViewModel

    @Before
    fun setup() {
        MockKAnnotations.init(this)
        every { mockRepository.getData() } returns flowOf(testData)

        testDispatcher = StandardTestDispatcher(testScheduler)
        testScope = TestScope(testDispatcher)
        Dispatchers.setMain(testDispatcher)

        viewModel = MyViewModel(mockNavigator, mockRepository)
    }

    @After
    fun tearDown() {
        testScope.advanceUntilIdle()
        Dispatchers.resetMain()
    }
}
```

### Test Naming
- **Descriptive Names:** Use backticks for readable test names
- **Pattern:** `when [condition] then [expected behavior]`
- **Examples:**
  ```kotlin
  @Test
  fun `when user clicks button then navigation occurs`()

  @Test
  fun `when data loads successfully then state updates`()
  ```

### Test Structure (Given-When-Then)
```kotlin
@Test
fun `when device disconnects then connection message shown`() = runTest {
    // Given: Initial setup and state
    val initialState = MyViewModel.UiState(isConnected = true)

    // When: Action occurs
    viewModel.onEvent(UiEvent.DeviceDisconnected)
    testDispatcher.scheduler.runCurrent()

    // Then: Verify expected behavior
    assertTrue(viewModel.uiState.value.showConnectionMessage)
    assertFalse(viewModel.uiState.value.isConnected)
}
```

### MockK Usage
- **Annotations:** Use `@MockK` for mock dependencies
- **Relaxed Mocks:** Use `@MockK(relaxed = true)` when you don't care about all interactions
- **Mock Initialization:** Call `MockKAnnotations.init(this)` in `@Before`
- **Stubbing:** Use `every { ... } returns ...` for method stubbing
- **Verification:** Use `verify { ... }` to verify method calls
- **Coroutine Mocking:** Use `coEvery { ... } returns ...` for suspend functions

### Coroutine Testing
- **runTest:** Wrap test body in `runTest { }` for coroutine tests
- **TestDispatcher:** Use `StandardTestDispatcher` with `TestCoroutineScheduler`
- **Test Scope:** Create `TestScope` with test dispatcher
- **Set Main Dispatcher:** Use `Dispatchers.setMain(testDispatcher)`
- **Advance Time:** Use `testDispatcher.scheduler.runCurrent()` or `advanceUntilIdle()`
- **Reset:** Always call `Dispatchers.resetMain()` in `@After`

### StateFlow Testing
- **Collect State:** Use custom `collectStateFlow()` helper
- **Initial State:** Wait for initial state to settle
- **State Updates:** Use `runCurrent()` to process state updates
- **Assertions:** Assert on `currentState.value.property`

### Advanced Mocking
- **Object Mocking:** Use `mockkObject()` for singleton objects
  ```kotlin
  mockkObject(ServiceRepository)
  every { ServiceRepository.shared.getService() } returns mockService
  unmockkObject(ServiceRepository) // Cleanup in @After
  ```
- **Static Functions:** Use `mockkStatic()` for extension functions
  ```kotlin
  mockkStatic("com.package.ExtensionsKt")
  every { device.isSupported() } returns true
  unmockkStatic("com.package.ExtensionsKt") // Cleanup
  ```

### Test Helper Patterns
- **Setup Helpers:** Extract complex initialization into helper methods
  ```kotlin
  private fun initViewModel(
      device: Device? = null,
      isConnected: Boolean = true
  ) {
      // Complex setup logic
  }
  ```
- **Custom Wait Utilities:** For complex state conditions
  ```kotlin
  private fun waitUntilCondition(
      dispatcher: TestDispatcher,
      condition: () -> Boolean
  ) {
      while (!condition()) {
          dispatcher.scheduler.runCurrent()
      }
  }
  ```

### Best Practices
- **Test One Thing:** Each test should verify one specific behavior
- **Readable Tests:** Use descriptive names and clear structure
- **Independent Tests:** Tests should not depend on each other
- **Fast Tests:** Keep unit tests fast, avoid Thread.sleep
- **Mock External:** Mock all external dependencies
- **Test Edge Cases:** Include tests for error states, empty data, null values
- **Cleanup:** Always clean up mocks and test state
