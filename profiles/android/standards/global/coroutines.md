## Kotlin Coroutines

### Coroutine Scope
- **ViewModel:** Use `viewModelScope` for ViewModel coroutines (automatically canceled)
- **Lifecycle:** Use `lifecycleScope` for lifecycle-aware coroutines
- **Never GlobalScope:** Avoid `GlobalScope`, it's not lifecycle-aware

### BaseViewModel Helpers
- **runInUIThread:** Use for UI-related coroutines (Main dispatcher)
```kotlin
runInUIThread {
    // UI operations on Main dispatcher
}
```
- **runInIOThread:** Use for I/O operations (IO dispatcher)
```kotlin
runInIOThread {
    // Network or disk I/O on IO dispatcher
}
```

### Flow Collection
- **subscribeToCollect:** Use BaseViewModel's helper method
```kotlin
override fun subscribe() {
    subscribeToCollect(repository.dataFlow) { data ->
        _uiState.update { it.copy(data = data) }
    }
}
```
- **collectOnFirstRun:** Control whether to collect initial StateFlow value
```kotlin
subscribeToCollect(flow, collectOnFirstRun = false) { value ->
    // Handle value changes, skip initial
}
```

### Dispatchers
- **Main:** UI updates, lightweight work
- **IO:** Network requests, database, file operations
- **Default:** CPU-intensive work (parsing, computation)

### StateFlow Pattern
```kotlin
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

fun updateState(newValue: String) {
    _uiState.update { it.copy(value = newValue) }
}
```

### SharedFlow for Events
```kotlin
private val _events = MutableSharedFlow<UiEvent>()
val events: SharedFlow<UiEvent> = _events.asSharedFlow()

fun emitEvent(event: UiEvent) {
    viewModelScope.launch {
        _events.emit(event)
    }
}
```

### Suspend Functions
- **Suspend Modifier:** Use for long-running or async operations
- **No Blocking:** Never block in suspend functions
- **Return Results:** Return results directly, not through callbacks
```kotlin
suspend fun loadData(): Result<Data> {
    return withContext(Dispatchers.IO) {
        try {
            Result.success(repository.getData())
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Error Handling
- **Try-Catch:** Wrap suspend calls in try-catch
- **CoroutineExceptionHandler:** Use for uncaught exceptions

### Cancellation
- **Cooperative:** Make long-running work cancellable
- **isActive:** Check in loops
```kotlin
while (isActive) {
    // Work that can be canceled
}
```

### Flow Operators
- **map:** Transform emitted values
- **filter:** Filter emitted values
- **combine:** Combine multiple flows
- **stateIn:** Convert to StateFlow
```kotlin
val combinedState = combine(flow1, flow2) { a, b ->
    CombinedState(a, b)
}.stateIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(5000),
    initialValue = CombinedState()
)
```

### Best Practices
- **Update State Safely:** Use `.update { }` for thread-safe StateFlow updates
- **Don't Return Flows:** Expose flows as properties, not return from methods
- **Single Source of Truth:** Use single StateFlow for UI state
- **Lifecycle Awareness:** Always use lifecycle-aware scopes in Android
- **No runBlocking:** Never use `runBlocking` in production code (tests only)
- **Collect in UI:** Collect flows in Composables with `collectAsStateWithLifecycle()`
