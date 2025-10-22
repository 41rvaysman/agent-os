# Compose Performance & Optimization

## Quick Reference

| Scenario | Solution |
|----------|----------|
| Expensive calculation | `remember(key) { }` |
| Derive from State | `derivedStateOf { }` |
| Survive config changes | `rememberSaveable` |
| Coroutine in composable | `LaunchedEffect(key)` |
| Need cleanup | `DisposableEffect` |
| Collect Flow | `collectAsStateWithLifecycle()` |
| Large lists | Use keys + contentType |
| ViewModel background work | `runInIOThread { }` |
| ViewModel UI updates | `runInUIThread { }` |
| Service subscriptions | `subscribeToCollect()` |

## Recomposition Optimization

### Understanding Recomposition
- **Recomposition is Normal:** Compose recomposes frequently and efficiently
- **Smart Recomposition:** Only composables with changed inputs recompose
- **Read State Carefully:** Reading state triggers recomposition of that scope
- **Stability Matters:** Stable parameters skip recomposition when unchanged

### Stable vs Unstable Types
```kotlin
// ✅ Stable - Won't cause unnecessary recomposition
@Immutable
data class UserProfile(
    val name: String,
    val email: String
)

// ✅ Stable - Use immutable collections for better performance
import kotlinx.collections.immutable.ImmutableList
import kotlinx.collections.immutable.persistentListOf

data class UiState(
    val items: ImmutableList<Item> = persistentListOf()  // Compose can skip recomposition
)

// ❌ Unstable - Mutable properties or collections
data class UnstableState(
    var count: Int = 0,  // var makes it unstable
    val items: MutableList<String> = mutableListOf()  // Mutable collection
)

// ✅ Fix with @Stable annotation when needed
@Stable
class CustomViewModel {
    val state: StateFlow<UiState> = MutableStateFlow(UiState())
}
```

### Keys and ContentType in Lazy Lists
```kotlin
// ✅ Good - Use stable, unique keys + contentType for better recycling
LazyColumn {
    items(
        items = mixedItems,
        key = { it.id },
        contentType = { it.type }  // Helps Compose recycle views efficiently
    ) { item ->
        when (item.type) {
            ItemType.HEADER -> HeaderCard(item)
            ItemType.CONTENT -> ContentCard(item)
        }
    }
}

// ❌ Bad - No key (uses index, causes issues on reorder/insert)
LazyColumn {
    items(userList) { user ->
        UserCard(user)
    }
}
```

### Lazy Layout Anti-Patterns
```kotlin
// ❌ NEVER - Nested scrollable containers in same direction
LazyColumn {
    item {
        LazyColumn {  // Causes major performance issues
            items(nestedItems) { }
        }
    }
}

// ✅ Good - Flatten into single list
LazyColumn {
    items(section1Items, key = { it.id }) { }
    items(section2Items, key = { it.id }) { }
}
```

### Reading State Efficiently
```kotlin
// ✅ Good - Read state at lowest possible level
@Composable
fun UserList(users: List<User>, selectedId: String) {
    LazyColumn {
        items(users, key = { it.id }) { user ->
            // Only this item recomposes when selection changes
            UserItem(
                user = user,
                isSelected = user.id == selectedId
            )
        }
    }
}

// ❌ Bad - Reading state too high causes entire list to recompose
@Composable
fun UserList(usersState: State<List<User>>) {
    val users = usersState.value  // Entire list recomposes on any state change
    LazyColumn {
        items(users, key = { it.id }) { user ->
            UserItem(user)
        }
    }
}
```

## Remember & Derived State

### When to Use remember
```kotlin
// ✅ Good - Remember expensive calculations
@Composable
fun FilteredList(items: List<Item>, query: String) {
    val filteredItems = remember(items, query) {
        items.filter { it.name.contains(query, ignoreCase = true) }
            .sortedBy { it.name }
    }

    LazyColumn {
        items(filteredItems, key = { it.id }) { item ->
            ItemCard(item)
        }
    }
}

// ❌ Bad - Recomputes every recomposition
@Composable
fun FilteredList(items: List<Item>, query: String) {
    val filteredItems = items.filter { it.name.contains(query, ignoreCase = true) }
        .sortedBy { it.name }  // Runs on every recomposition!
}
```

### remember vs derivedStateOf
```kotlin
// ✅ Use remember for expensive one-time calculations
@Composable
fun UserProfile(userId: String) {
    val formatter = remember {
        DateTimeFormatter.ofPattern("MMM dd, yyyy")
    }
}

// ✅ Use derivedStateOf when deriving from State
@Composable
fun SearchResults(searchQuery: State<String>, allItems: List<Item>) {
    val filteredItems by remember {
        derivedStateOf {
            if (searchQuery.value.isEmpty()) {
                allItems
            } else {
                allItems.filter { it.name.contains(searchQuery.value, ignoreCase = true) }
            }
        }
    }
    // Only recomputes when searchQuery.value actually changes
}
```

### rememberSaveable for Configuration Changes
```kotlin
// ✅ Good - Survives configuration changes and process death
@Composable
fun SearchScreen() {
    var searchQuery by rememberSaveable { mutableStateOf("") }
    var isExpanded by rememberSaveable { mutableStateOf(false) }

    SearchBar(
        query = searchQuery,
        onQueryChange = { searchQuery = it }
    )
}

// ⚠️ Limited - Lost on configuration change
@Composable
fun SearchScreen() {
    var searchQuery by remember { mutableStateOf("") }
    // Lost on rotation
}
```

## Lambda & Function References

### Avoiding Lambda Recreations
```kotlin
// ✅ Good - Lambda doesn't capture changing state
@Composable
fun ItemList(items: List<Item>, onItemClick: (Item) -> Unit) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemCard(
                item = item,
                onClick = { onItemClick(item) }  // Stable
            )
        }
    }
}

// ❌ Bad - Lambda captures changing state, recreated on every change
@Composable
fun ItemList(items: List<Item>, selectedId: String, onSelect: (String) -> Unit) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemCard(
                item = item,
                onClick = {
                    // Lambda captures selectedId - recreated on every selection change
                    if (selectedId != item.id) {
                        onSelect(item.id)
                    }
                }
            )
        }
    }
}
```

### ABApp Pattern: Method References
```kotlin
// ✅ Best - Method reference is stable and preferred
@Composable
fun PairingView(viewModel: PairingViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()

    PairingContent(
        state = state,
        event = viewModel::onUiEvent  // Stable method reference
    )
}

// Usage in content composable
@Composable
private fun PairingContent(
    state: PairingViewModel.UiState,
    event: (PairingViewModel.UiEvent) -> Unit
) {
    Button(onClick = { event(PairingViewModel.UiEvent.RetryPairing) }) {
        Text("Retry")
    }
}
```

## Side Effects & Lifecycle

### LaunchedEffect Best Practices
```kotlin
// ✅ Good - Proper key management
@Composable
fun UserProfile(userId: String, viewModel: UserViewModel) {
    LaunchedEffect(userId) {
        viewModel.loadUser(userId)
    }
    // Re-launches only when userId changes
}

// ✅ Good - Multiple keys when needed
@Composable
fun SearchResults(query: String, filters: FilterState, viewModel: SearchViewModel) {
    LaunchedEffect(query, filters) {
        viewModel.search(query, filters)
    }
    // Re-launches when either query OR filters change
}

// ❌ Bad - Using Unit or true as key
@Composable
fun UserProfile(viewModel: UserViewModel) {
    LaunchedEffect(Unit) {
        viewModel.loadUser()
    }
    // Never re-launches, even when it should
}
```

### DisposableEffect for Cleanup
```kotlin
// ✅ Good - Cleanup resources (ABApp pattern)
@Composable
fun NavigationHandler(customNavigator: CustomNavigator, navController: NavHostController) {
    DisposableEffect(customNavigator, navController) {
        val job = CoroutineScope(Dispatchers.Main).launch {
            customNavigator.navActions.collect { action ->
                customNavigator.runNavigationCommand(action, navController)
            }
        }

        onDispose {
            job.cancel()
        }
    }
}
```

## Memory Management

### ABApp BaseViewModel Pattern
All ViewModels extend `BaseViewModel` which provides automatic subscription management and threading helpers.

```kotlin
// ✅ Excellent - Use BaseViewModel.subscribeToCollect
@HiltViewModel
class PairingViewModel @Inject constructor(
    private val customNavigator: CustomNavigator
) : BaseViewModel() {

    data class UiState(
        val devices: List<Device> = emptyList(),
        val isPairing: Boolean = false
    )

    sealed interface UiEvent {
        data object RetryPairing : UiEvent
        data object PairingDone : UiEvent
    }

    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    init {
        subscribe()
    }

    override fun subscribe() {
        // subscribeToCollect automatically manages Job lifecycle
        // Subscriptions cancelled on ViewModel clear or service reinitialization
        subscribeToCollect(ServiceRepository.shared.pairingService.deviceState) { state ->
            _uiState.update { it.copy(devices = state.devices) }
        }

        subscribeToCollect(ServiceRepository.shared.pairingService.isPairing) { isPairing ->
            _uiState.update { it.copy(isPairing = isPairing) }
        }
    }

    fun onUiEvent(event: UiEvent) {
        when (event) {
            UiEvent.PairingDone -> customNavigator.navigate(AppRoute.PairingDone)
            UiEvent.RetryPairing -> retryPairing()
        }
    }

    private fun retryPairing() {
        runInIOThread {
            // Background work automatically scoped to viewModelScope
            ServiceRepository.shared.pairingService.retry()
        }
    }
}

// ⚠️ Skip First Collection When Needed
@HiltViewModel
class SettingsViewModel @Inject constructor() : BaseViewModel() {
    override fun subscribe() {
        // collectOnFirstRun = false prevents immediate execution
        // Useful for StateFlow that always emits current value
        subscribeToCollect(
            flow = ServiceRepository.shared.settingsService.volume,
            collectOnFirstRun = false  // Skip initial emission
        ) { volume ->
            _uiState.update { it.copy(volume = volume) }
        }
    }
}

// ❌ Bad - Don't use GlobalScope or manual viewModelScope.launch
class MyViewModel : BaseViewModel() {
    fun loadData() {
        GlobalScope.launch {  // NEVER - creates memory leak
            // This never cancels, keeps ViewModel in memory
        }

        viewModelScope.launch(Dispatchers.IO) {  // Don't - use runInIOThread instead
        }
    }
}
```

### Proper Flow Collection
```kotlin
// ✅ Good - collectAsStateWithLifecycle
@Composable
fun MyView(viewModel: MyViewModel = hiltViewModel()) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    // Automatically stops collection when not visible
    // Prevents wasted resources and memory
}

// ⚠️ Acceptable but less efficient - collectAsState
@Composable
fun MyView(viewModel: MyViewModel = hiltViewModel()) {
    val state by viewModel.state.collectAsState()
    // Continues collecting even when screen is in background
}
```

### Image & Bitmap Memory
```kotlin
// ✅ Good - Use Coil with proper size configuration
@Composable
fun UserAvatar(imageUrl: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(imageUrl)
            .memoryCacheKey(imageUrl)
            .diskCacheKey(imageUrl)
            .size(200)  // Scale to display size
            .build(),
        contentDescription = "User avatar",
        modifier = Modifier.size(64.dp)
    )
}

// ❌ Bad - Loading full-size images
@Composable
fun UserAvatar(imageUrl: String) {
    AsyncImage(
        model = imageUrl,  // Loads full resolution
        contentDescription = "User avatar",
        modifier = Modifier.size(64.dp)  // Displays tiny but keeps full image in memory
    )
}
```

### Avoiding Captured References
```kotlin
// ❌ Bad - LaunchedEffect captures Activity context
@Composable
fun SomeScreen() {
    val context = LocalContext.current

    LaunchedEffect(Unit) {
        delay(60000)  // 1 minute delay
        // Holds reference to Activity for 1 minute
        Toast.makeText(context, "Done", Toast.LENGTH_SHORT).show()
    }
}

// ✅ Better - Use application context for long-running operations
@Composable
fun SomeScreen() {
    val applicationContext = LocalContext.current.applicationContext

    LaunchedEffect(Unit) {
        delay(60000)
        // Only holds app context, not Activity
        Toast.makeText(applicationContext, "Done", Toast.LENGTH_SHORT).show()
    }
}

// ✅ Best - Move to ViewModel
@HiltViewModel
class MyViewModel @Inject constructor(
    @ApplicationContext private val context: Context
) : BaseViewModel() {
    fun scheduleLongRunningTask() {
        viewModelScope.launch {
            delay(60000)
            // ViewModel properly scoped, no Activity leak
        }
    }
}
```

## Animation Performance

### Use graphicsLayer for Animations
```kotlin
// ✅ Good - Use graphicsLayer for animations (no recomposition)
@Composable
fun AnimatedBox(scale: Float, alpha: Float) {
    Box(
        modifier = Modifier
            .graphicsLayer {
                scaleX = scale
                scaleY = scale
                this.alpha = alpha
            }
    )
    // Animates without triggering recomposition
}

// ❌ Bad - Triggers recomposition on every frame
@Composable
fun AnimatedBox(scale: Float, alpha: Float) {
    Box(
        modifier = Modifier
            .scale(scale)  // Recomposes every frame
            .alpha(alpha)   // Recomposes every frame
    )
}
```

### Modifier Order Matters
```kotlin
// ✅ Good - Layout modifiers before drawing modifiers
Modifier
    .size(100.dp)           // Layout
    .padding(16.dp)         // Layout
    .background(Color.Red)  // Drawing
    .clickable { }          // Pointer input

// ❌ Bad - Forces extra measure/layout passes
Modifier
    .background(Color.Red)  // Drawing
    .padding(16.dp)         // Layout - forces remeasure
    .size(100.dp)           // Layout
```

## Common Anti-Patterns

### Anti-Pattern: Heavy Work in Composition
```kotlin
// ❌ Bad - Heavy computation in composition
@Composable
fun UserStats(transactions: List<Transaction>) {
    val totalAmount = transactions.sumOf { it.amount }
    val avgAmount = transactions.map { it.amount }.average()
    // Computed on every recomposition!

    StatsDisplay(total = totalAmount, average = avgAmount)
}

// ✅ Good - Move to ViewModel
data class UiState(
    val transactions: List<Transaction> = emptyList(),
    val totalAmount: Double = 0.0,
    val avgAmount: Double = 0.0
)

@HiltViewModel
class StatsViewModel @Inject constructor() : BaseViewModel() {
    private val _state = MutableStateFlow(UiState())
    val state: StateFlow<UiState> = _state.asStateFlow()

    fun updateTransactions(transactions: List<Transaction>) {
        _state.update {
            it.copy(
                transactions = transactions,
                totalAmount = transactions.sumOf { tx -> tx.amount },
                avgAmount = transactions.map { tx -> tx.amount }.average()
            )
        }
    }
}
```

### Anti-Pattern: Recreating Objects in Composition
```kotlin
// ❌ Bad - Creates new list every recomposition
@Composable
fun FilteredList(items: List<Item>, showCompleted: Boolean) {
    val filteredItems = if (showCompleted) {
        items
    } else {
        items.filter { !it.completed }  // New list every time
    }
}

// ✅ Good - Remember filtered result
@Composable
fun FilteredList(items: List<Item>, showCompleted: Boolean) {
    val filteredItems = remember(items, showCompleted) {
        if (showCompleted) items else items.filter { !it.completed }
    }
}
```

## Performance Monitoring

### Recomposition Debugging
```kotlin
// ✅ Good - Track recompositions in debug builds
@Composable
fun PerformanceTracker(tag: String, content: @Composable () -> Unit) {
    if (BuildConfig.DEBUG) {
        val recompositions = remember { mutableStateOf(0) }
        SideEffect {
            recompositions.value++
            Log.d("Recomposition", "$tag recomposed ${recompositions.value} times")
        }
    }
    content()
}

// Usage
@Composable
fun MyExpensiveComponent() {
    PerformanceTracker("MyExpensiveComponent") {
        // Component content
    }
}
```

### Enable Compose Compiler Metrics
```kotlin
// build.gradle.kts - Check stability reports for unstable classes
kotlinOptions {
    freeCompilerArgs += listOf(
        "-P",
        "plugin:androidx.compose.compiler.plugins.kotlin:reportsDestination=" +
            project.buildDir.absolutePath + "/compose_metrics"
    )
}

// Use Layout Inspector in Android Studio:
// - Enable "Show Recomposition Counts"
// - Red highlights indicate frequent recompositions
```

## ABApp-Specific Patterns

### BaseViewModel Pattern
```kotlin
// ✅ Always extend BaseViewModel
@HiltViewModel
class MyViewModel @Inject constructor(
    private val customNavigator: CustomNavigator
) : BaseViewModel() {

    data class UiState(/* state properties */)
    sealed interface UiEvent {/* event types */}

    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    init {
        subscribe()  // Start subscriptions
    }

    override fun subscribe() {
        // Use subscribeToCollect for all Flow subscriptions
        // Auto-cancels on ViewModel.onCleared()
        // Auto-cancels on service reinitialization
        subscribeToCollect(ServiceRepository.shared.service.data) { data ->
            _uiState.update { it.copy(data = data) }
        }
    }

    fun onUiEvent(event: UiEvent) {
        when (event) {
            is UiEvent.Navigate -> customNavigator.navigate(AppRoute.Target)
            is UiEvent.Action -> handleAction()
        }
    }
}
```

### Threading Helpers
```kotlin
// ✅ Use BaseViewModel threading helpers
runInUIThread {
    // UI updates on Main dispatcher
}

runInIOThread {
    // Background work on IO dispatcher
}

// ❌ Don't use viewModelScope.launch directly
viewModelScope.launch(Dispatchers.IO) {
    // Use runInIOThread instead
}
```

### Navigation Pattern
```kotlin
// ✅ Use CustomNavigator (injected singleton)
customNavigator.navigate(AppRoute.Home)
customNavigator.navigateAndClear(AppRoute.Welcome)
customNavigator.goBack()
customNavigator.goBackTo(AppRoute.Settings)
```

### Service Access Pattern
```kotlin
// ✅ Access services via ServiceRepository.shared
ServiceRepository.shared.pairingService.startPairing()
ServiceRepository.shared.remoteControlService.connect()
ServiceRepository.shared.monitoringService.deviceState

// Subscribe to service flows in ViewModel.subscribe()
override fun subscribe() {
    subscribeToCollect(ServiceRepository.shared.pairingService.state) { state ->
        _uiState.update { it.copy(pairingState = state) }
    }
}
```

### Spacing Components
```kotlin
// ✅ Use SpacerH for vertical spacing
Column {
    Text("First")
    SpacerH(MaterialTheme.spacing.spacing16dp)
    Text("Second")
}

// ✅ Use SpacerW for horizontal spacing
Row {
    Text("First")
    SpacerW(MaterialTheme.spacing.spacing8dp)
    Text("Second")
}

// ❌ Don't create inline Spacers
Spacer(modifier = Modifier.height(16.dp))  // Use SpacerH instead
```

### UiState & UiEvent Pattern
```kotlin
// ✅ Define nested in ViewModel
@HiltViewModel
class MyViewModel @Inject constructor() : BaseViewModel() {

    data class UiState(
        val isLoading: Boolean = false,
        val data: List<Item> = emptyList(),
        val error: String? = null
    )

    sealed interface UiEvent {
        data object Refresh : UiEvent
        data class ItemClick(val id: String) : UiEvent
        data object GoBack : UiEvent
    }

    fun onUiEvent(event: UiEvent) {
        when (event) {
            UiEvent.Refresh -> refresh()
            is UiEvent.ItemClick -> handleClick(event.id)
            UiEvent.GoBack -> customNavigator.goBack()
        }
    }
}
```

## Best Practices Summary

### Recomposition
- Use stable, immutable data classes for state
- Provide keys + contentType for LazyColumn/LazyRow items
- Read state at the lowest possible scope
- Use `@Stable` and `@Immutable` annotations when appropriate

### Remember & State
- Use `remember` for expensive calculations and object creation
- Use `derivedStateOf` when deriving from other State
- Use `rememberSaveable` for UI state that should survive configuration changes
- Remember the key parameter - only recompute when keys change

### Side Effects
- Use `LaunchedEffect` for coroutines tied to composition lifecycle
- Use `DisposableEffect` when cleanup is needed
- Always specify proper keys for effects
- Don't capture Activity context in long-running effects

### Memory Management
- Always use `viewModelScope` in ViewModels, never GlobalScope
- Use `collectAsStateWithLifecycle()` for Flow collection
- Scale images to display size, not full resolution
- Clean up resources in `onDispose` blocks
- Use Application context for long-running operations

### Performance
- Avoid heavy computation in composition - do it in ViewModel
- Don't create new objects/lists in composition body
- Use `graphicsLayer` for animations
- Order modifiers correctly (layout before drawing)
- Monitor recompositions in debug builds
