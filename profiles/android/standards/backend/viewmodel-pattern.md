## ViewModel Architecture Pattern

### Base ViewModel
- **Extend BaseViewModel:** All ViewModels should extend the project's `BaseViewModel` class
- **Subscribe Pattern:** Use `subscribe()` and `unsubscribe()` lifecycle methods for Flow collection
- **Flow Collection Helper:** Use `subscribeToCollect()` for automatic cleanup of Flow subscriptions

### ViewModel Structure
```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val navigator: CustomNavigator,
    private val useCase: MyUseCase
) : BaseViewModel() {

    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    data class UiState(
        val property1: String = "",
        val property2: Boolean = false,
        val isLoading: Boolean = false
    )

    sealed interface UiEvent {
        data object GoBack : UiEvent
        data class OnAction(val param: String) : UiEvent
    }

    init {
        subscribe()
    }

    override fun subscribe() {
        subscribeToCollect(someFlow) { value ->
            _uiState.update { it.copy(property1 = value) }
        }
    }

    fun onEvent(event: UiEvent) {
        when (event) {
            is UiEvent.GoBack -> navigator.goBack()
            is UiEvent.OnAction -> handleAction(event.param)
        }
    }
}
```

### State Management
- **Private MutableStateFlow:** Expose private `_uiState` as `MutableStateFlow`
- **Public StateFlow:** Expose public `uiState` using `asStateFlow()`
- **Data Class for State:** Define `UiState` as data class with default values
- **Immutable Updates:** Use `.update { it.copy(...) }` to update state
- **Nested State Classes:** Define UiState inside ViewModel for encapsulation

### Event Handling
- **Sealed Interface:** Define `UiEvent` as sealed interface or sealed class
- **Single Event Method:** Implement `onEvent(event: UiEvent)` method
- **When Expression:** Use exhaustive when to handle all event types
- **Event Objects:** Use `data object` for parameter-less events, `data class` for events with parameters

### Dependency Injection
- **@HiltViewModel:** Annotate ViewModel with `@HiltViewModel`
- **@Inject Constructor:** Use constructor injection for dependencies
- **Navigator Injection:** Inject `CustomNavigator` for navigation
- **Use Case Injection:** Inject use cases for business logic
- **Repository Injection:** Inject repositories for data access

### Threading
- **runInUIThread:** Use for UI-related coroutines (runs on Main dispatcher)
- **runInIOThread:** Use for IO operations (runs on IO dispatcher)
- **viewModelScope:** Automatically available, use when dispatcher doesn't matter
- **Avoid Blocking:** Never block the main thread

### Flow Subscription
- **subscribeToCollect:** Preferred method for collecting StateFlow/SharedFlow
- **collectOnFirstRun Parameter:** Set to `false` to skip initial collection (StateFlow default behavior)
- **Automatic Cleanup:** Subscriptions are automatically canceled on ViewModel clear
- **Manual Cleanup:** Call `unsubscribe()` when needed (e.g., service reinitialization)

### Navigation
- **CustomNavigator:** Use injected `CustomNavigator` for all navigation
- **AppRoute:** Navigate using sealed class `AppRoute` instances
- **Go Back:** Use `navigator.goBack()` for back navigation
- **Navigation in Events:** Handle navigation in `onEvent()` method

### Best Practices
- **Single Responsibility:** Each ViewModel should manage one screen/feature
- **No Android Framework:** Avoid direct Android framework dependencies (except androidx.lifecycle)
- **Testable:** Design ViewModels to be easily unit testable
- **Clear State:** State should clearly represent all possible UI states
- **Side Effects in Events:** Handle side effects (navigation, snackbars) in event handlers
- **Initialize in init:** Start subscriptions and load initial data in `init` block
- **Observable Everything:** Expose all UI-relevant data through StateFlow
