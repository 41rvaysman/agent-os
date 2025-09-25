# Jetpack Compose Style Guide

## Official Documentation Reference

📖 **When in doubt, reference**: [Compose Component API Guidelines](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-component-api-guidelines.md)

## Quick Decision Tree for AI
- **Creating new composable?** → Follow Basic Pattern (line 35-53)
- **Need ViewModel integration?** → Use Hilt pattern (line 87-139)
- **Building lists?** → Use LazyColumn with keys (line 142-166)
- **Need content slots?** → Follow ActionCard pattern (line 170-192)
- **Performance concerns?** → Check Stable Parameters (line 236-250)

<conditional-block task-condition="jetpack-compose" context-check="compose-style">
IF creating/modifying Jetpack Compose UI:
  REQUIRED: Follow parameter order (required → modifier → optional → content slots)
  REQUIRED: Use theme values (MaterialTheme.colorScheme/typography)
  REQUIRED: Add preview functions for new components
  IF creating ViewModel: MUST extend BaseViewModel + use @HiltViewModel
  IF creating lists: MUST include key parameter in items()
</conditional-block>

## Core Principles

1. **Single Purpose**: Each component solves only one problem
2. **Explicit Parameters**: Make all dependencies clear in function parameters
3. **Stable Parameters**: Use immutable types to prevent unnecessary recomposition
4. **Consistent Naming**: Use PascalCase for composable functions (like React components)

## Function Structure <!-- KEYWORDS: composable, parameters, modifier, structure -->

### Basic Pattern
```kotlin
@Composable
fun ComponentName(
    // 1. Required parameters
    title: String,
    onClick: () -> Unit,
    
    // 2. Modifier (first optional parameter)
    modifier: Modifier = Modifier,
    
    // 3. Other optional parameters
    enabled: Boolean = true,
    
    // 4. Content slots (always last)
    content: @Composable () -> Unit = {}
) {
    // Implementation
}
```

### Real Example
```kotlin
@Composable
fun DeviceCard(
    title: String,
    status: String,
    modifier: Modifier = Modifier,
    statusColor: Color = MaterialTheme.colorScheme.primary,
    onCardClick: (() -> Unit)? = null
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .let { if (onCardClick != null) it.clickable { onCardClick() } else it }
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = title,
                style = MaterialTheme.typography.headlineSmall
            )
            Text(
                text = status,
                style = MaterialTheme.typography.bodyMedium,
                color = statusColor
            )
        }
    }
}
```

## Common Patterns <!-- KEYWORDS: viewmodel, hilt, stateful, stateless, lists -->

### Connect Composables with ViewModels using Hilt
```kotlin
// ViewModel with Hilt
@HiltViewModel
class DeviceViewModel @Inject constructor(
    private val deviceService: DeviceService,
    private val eventHelper: EventHelper
) : BaseViewModel() {
    
    private val _uiState = MutableStateFlow(DeviceUiState.Loading)
    val uiState: StateFlow<DeviceUiState> = _uiState.asStateFlow()
    
    override fun subscribe() {
        subscribeToCollect(deviceService.deviceState) { state ->
            _uiState.value = DeviceUiState.Success(state.devices)
        }
    }
    
    fun handleAction(action: DeviceAction) {
        when (action) {
            is DeviceAction.ConnectDevice -> connectDevice(action.deviceId)
            is DeviceAction.DisconnectDevice -> disconnectDevice(action.deviceId)
        }
    }
}

// Screen (stateful) - Hilt injection point
@Composable
fun DeviceScreen(
    viewModel: DeviceViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    
    DeviceContent(
        uiState = uiState,
        onAction = viewModel::handleAction
    )
}

// Content (stateless) - easily testable
@Composable
fun DeviceContent(
    uiState: DeviceUiState,
    onAction: (DeviceAction) -> Unit,
    modifier: Modifier = Modifier
) {
    when (uiState) {
        is DeviceUiState.Loading -> CircularProgressIndicator()
        is DeviceUiState.Success -> DeviceList(uiState.devices, onAction)
        is DeviceUiState.Error -> ErrorMessage(uiState.message)
    }
}
```

### State Management Quick Reference
```kotlin
// 1. Screen State (in ViewModel)
private val _uiState = MutableStateFlow(InitialState)
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

// 2. Local Component State
@Composable
fun Component() {
    var isExpanded by remember { mutableStateOf(false) }
    // Use isExpanded
}

// 3. Derived State
@Composable
fun Component(items: List<Item>) {
    val isEmpty by remember(items) { derivedStateOf { items.isEmpty() } }
    // Use isEmpty
}
```

### Lists with Keys
```kotlin
@Composable
fun DeviceList(
    devices: List<Device>,
    onDeviceClick: (Device) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyColumn(
        modifier = modifier,
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(
            items = devices,
            key = { device -> device.id } // Important for performance
        ) { device ->
            DeviceCard(
                title = device.name,
                status = device.status,
                onCardClick = { onDeviceClick(device) }
            )
        }
    }
}
```

### Content Slots
```kotlin
@Composable
fun ActionCard(
    title: String,
    modifier: Modifier = Modifier,
    actions: @Composable () -> Unit = {}
) {
    Card(modifier = modifier) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(title, style = MaterialTheme.typography.headlineSmall)
            Spacer(modifier = Modifier.height(8.dp))
            actions()
        }
    }
}

// Usage
ActionCard(title = "Settings") {
    Row {
        TextButton(onClick = {}) { Text("Cancel") }
        Button(onClick = {}) { Text("Save") }
    }
}
```

### Preview Functions
```kotlin
@Preview
@Preview(uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
private fun DeviceCardPreview() {
    PalmaTheme {
        DeviceCard(
            title = "Advanced Bionics Device",
            status = "Connected"
        )
    }
}
```

## Theme Integration - Required Patterns

### ✅ Always Use Theme Values
```kotlin
// Colors
MaterialTheme.colorScheme.primary      // Not Color.Blue
MaterialTheme.colorScheme.surface      // Not Color.White
MaterialTheme.colorScheme.onSurface    // Not Color.Black

// Typography  
MaterialTheme.typography.headlineSmall // Not fontSize = 18.sp
MaterialTheme.typography.bodyLarge     // Not fontSize = 16.sp

// Spacing (if defined in theme)
PalmaTheme.spacing.medium              // Not 16.dp directly
```

### ❌ Never Hardcode
```kotlin
// Wrong
Text(text = "Title", fontSize = 18.sp, color = Color.Blue)

// Correct  
Text(
    text = "Title", 
    style = MaterialTheme.typography.headlineSmall,
    color = MaterialTheme.colorScheme.primary
)
```

## Performance Tips

### Stable Parameters
```kotlin
// ✅ Good: Stable parameters
@Composable
fun DeviceStatus(
    deviceName: String,           // Stable
    isConnected: Boolean,         // Stable
    onToggle: () -> Unit          // Stable
) { /* ... */ }

// ❌ Avoid: Unstable parameters
@Composable  
fun DeviceStatus(
    device: MutableDevice         // Unstable - causes excessive recomposition
) { /* ... */ }
```

### Remember Expensive Operations
```kotlin
@Composable
fun ChartComponent(
    dataPoints: List<DataPoint>,
    modifier: Modifier = Modifier
) {
    val processedData = remember(dataPoints) {
        // Expensive computation only runs when dataPoints change
        processChartData(dataPoints)
    }
    
    // Chart implementation using processedData
}
```

## Common Mistakes to Avoid

### ❌ Incorrect Parameter Order
```kotlin
// Wrong - modifier should come before optional parameters
@Composable
fun BadComponent(
    title: String,
    enabled: Boolean = true,
    modifier: Modifier = Modifier  // Should be earlier
) { }

// Correct
@Composable 
fun GoodComponent(
    title: String,
    modifier: Modifier = Modifier,
    enabled: Boolean = true
) { }
```

### ❌ Missing Keys in Lists
```kotlin
// Wrong - will cause performance issues
LazyColumn {
    items(devices) { device ->
        DeviceCard(device)
    }
}

// Correct
LazyColumn {
    items(devices, key = { it.id }) { device ->
        DeviceCard(device)
    }
}
```

## Quick Reference

### ✅ Do
- Use PascalCase for composable function names
- Put `modifier` as first optional parameter
- Provide stable, explicit parameters
- Include meaningful content descriptions
- Use theme colors and typography
- Add preview functions

### ❌ Don't
- Place modifier last when content lambdas exist
- Pass entire objects when specific properties suffice
- Forget keys in LazyColumn items
- Hardcode colors or dimensions
- Create multi-purpose components
- Skip accessibility support

## Accessibility Example
```kotlin
@Composable
fun AccessibleButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true
) {
    Button(
        onClick = onClick,
        enabled = enabled,
        modifier = modifier.semantics {
            contentDescription = if (enabled) {
                "Button: $text. Tap to activate."
            } else {
                "Button: $text. Currently disabled."
            }
        }
    ) {
        Text(text)
    }
}
```

## Instrumented Testing with Mocks <!-- KEYWORDS: testing, mock, instrumented, hilt, compose -->

### Test Class Setup with Hilt
```kotlin
@OptIn(ExperimentalTestApi::class)
@HiltAndroidTest
class ListeningCheckLandingViewTest : BaseUITest() {

    private lateinit var mockNavigator: CustomNavigator
    private lateinit var mockViewModel: ListeningCheckLandingViewModel
    private lateinit var uiState: MutableStateFlow<ListeningCheckLandingViewModel.UiState>

    @Before
    fun setup() {
        hiltRule.inject()

        // Create mock navigator
        mockNavigator = mockk<CustomNavigator>()
        every { mockNavigator.navigate(any<AppRoute>()) } returns Unit
        every { mockNavigator.goBack() } returns Unit

        // Create initial UI state
        uiState = MutableStateFlow(ListeningCheckLandingViewModel.UiState())

        // Create mock ViewModel
        mockViewModel = mockk<ListeningCheckLandingViewModel>()
        every { mockViewModel.uiState } returns uiState
        every { mockViewModel.onEvent(any()) } returns Unit
    }

    @After
    fun cleanup() {
        runTest {
            try {
                ServiceRepository.shared.deviceGroupService.removeAllDevices()
                ServiceRepository.shared.configurationService.stopService()
            } catch (_: Exception) {
                // ServiceRepository not initialized - ignore
            }
        }
        clearAllMocks()
    }
}
```

### Creating Complex Mocks with Extension Functions
```kotlin
private fun createMockDevice(
    side: HiSide,
    deviceTypeName: String,
    isListeningCheckSupported: Boolean,
    serialNumber: String = "SN001",
    hasCrosIdentification: Boolean = false
): InternalDevice = mockk {
    every { this@mockk.side } returns side
    every { this@mockk.serialNumber } returns serialNumber
    every { this@mockk.deviceTypeName } returns deviceTypeName

    // Mock extension functions - these would normally be static functions
    every { this@mockk.isListeningCheckSupported() } returns isListeningCheckSupported
    every { this@mockk.hasCrosIdentification() } returns hasCrosIdentification
}
```

### Setting Up Composable Content for Testing
```kotlin
private fun setupComposableContent(state: ListeningCheckLandingViewModel.UiState) {
    uiState.value = state

    composeTestRule.setContent {
        initDemo() // Initialize demo mode if needed

        PalmaTheme {
            Surface {
                Column(modifier = Modifier.fillMaxSize()) {
                    ListeningCheckLandingView(mockViewModel)
                }
            }
        }
    }
}
```

### Testing User Interactions with Mocks
```kotlin
@Test
fun userClicksEnabledLeftDeviceButton_navigatesToListeningCheckPreparationForLeftSide() {
    // Given: State with enabled left device
    val leftDevice = createMockDevice(
        side = HiSide.LEFT,
        deviceTypeName = "Claro L",
        isListeningCheckSupported = true,
        serialNumber = "SL001"
    )
    val state = ListeningCheckLandingViewModel.UiState(
        leftDevice = leftDevice,
        isLeftSupported = true,
        isLeftButtonEnabled = true
    )
    setupComposableContent(state)

    // When: User clicks left device button
    composeTestRule.onNodeWithText("Claro L").performClick()

    // Then: ViewModel should receive TestDevice event for LEFT side
    verify { mockViewModel.onEvent(ListeningCheckLandingViewModel.UiEvent.TestDevice(HiSide.LEFT)) }
}
```

### Mocking StateFlow for Dynamic UI Updates
```kotlin
@Test
fun deviceGroupChangesToIncludePreviouslyMissingDevice_UIUpdatesToShowNewDeviceButtonWithAppropriateEnabledState() {
    // Given: Initial state with missing device
    val leftDevice = createMockDevice(
        side = HiSide.LEFT,
        deviceTypeName = "Claro L",
        isListeningCheckSupported = true,
        serialNumber = "SL001"
    )
    val initialState = ListeningCheckLandingViewModel.UiState(
        leftDevice = leftDevice,
        isLeftSupported = true,
        isRightDeviceMissing = true,
        isRightCros = false,
        isLeftButtonEnabled = false
    )
    setupComposableContent(initialState)

    // Verify initial state
    composeTestRule.onNodeWithText(getString(R.string.txt_not_paired)).assertIsDisplayed().assertIsNotEnabled()

    // When: Device group changes to include previously missing device
    val rightDevice = createMockDevice(
        side = HiSide.RIGHT,
        deviceTypeName = "Claro R",
        isListeningCheckSupported = true,
        serialNumber = "SR002"
    )
    val updatedState = ListeningCheckLandingViewModel.UiState(
        leftDevice = leftDevice,
        rightDevice = rightDevice,
        isLeftSupported = true,
        isRightSupported = true,
        isRightDeviceMissing = false,
        isLeftButtonEnabled = true,
        isRightButtonEnabled = true
    )

    // Update the state
    uiState.value = updatedState

    // Then: UI should update to show new device button
    composeTestRule.waitForIdle()
    composeTestRule.onNodeWithText("Claro R").assertIsDisplayed().assertIsEnabled()
}
```

### Helper Functions for Testing
```kotlin
private fun getString(resId: Int): String =
    InstrumentationRegistry.getInstrumentation().targetContext.getString(resId)
```

### Key Testing Patterns
- **Extend BaseUITest**: Provides common test infrastructure
- **Use @HiltAndroidTest**: Required for Hilt dependency injection in tests
- **Mock ViewModels**: Create mock ViewModels to control behavior and verify interactions
- **Mock Complex Objects**: Use factory functions for creating test data objects
- **StateFlow Testing**: Use MutableStateFlow to simulate state changes
- **Verify Interactions**: Use MockK's `verify` to ensure proper event handling
- **UI State Testing**: Test both initial states and dynamic state changes

---

💡 **For comprehensive details, edge cases, and advanced patterns, reference the [Official Compose Component API Guidelines](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-component-api-guidelines.md)**