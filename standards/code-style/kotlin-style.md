# Kotlin Style Guide

<!-- KEYWORDS: viewmodel, baseviewmodel, hilt, dependency injection, mvvm, real-simulated-service -->
<!-- PATTERNS: flow, coroutines, absolute-imports, service-pattern, viewmodel-pattern -->
<!-- RULES: 500-line-limit, 120-char-limit, no-i-prefix, baseviewmodel-required -->

## AI Quick Reference

### Critical Rules (Check First)
- **All ViewModels MUST extend BaseViewModel** ([see line 64-87](#viewmodel-requirements-critical))
- **Use Real/Simulated service pattern** ([see line 89-126](#service-implementation-pattern))
- **Max file length: 500 lines** ([see line 155](#file-structure))
- **Max line length: 120 characters** ([see line 20](#indentation-and-spacing))
- **No 'I' prefix for interfaces** ([see line 93](#service-implementation-pattern))
- **Use absolute imports only** ([see line 160](#import-organization))

### ✅ Always Do
- Extend BaseViewModel for all ViewModels
- Use @HiltViewModel annotation
- Implement subscribe() and unsubscribe()
- Use absolute imports
- Keep files under 500 lines
- Follow Real/Simulated service pattern

### ❌ Never Do
- Create ViewModel without BaseViewModel
- Use 'I' prefix for interfaces
- Use relative imports
- Exceed 500 lines per file
- Mix style changes with logic

### Common Patterns Quick Reference
- **ViewModel**: `@HiltViewModel + BaseViewModel + subscribe/unsubscribe`
- **Service**: `Interface + RealImpl + SimulatedImpl`
- **DI**: `@Inject constructor + @Singleton/@HiltViewModel`

## Context

Kotlin code formatting and visual styling rules following official Kotlin conventions for the Palma Android Application.

<conditional-block task-condition="kotlin" context-check="kotlin-style">
IF current task involves writing or updating Kotlin code:
  IF kotlin-style.md already in context:
    SKIP: Re-reading this file
    NOTE: "Using Kotlin style guide already in context"
  ELSE:
    READ: The following Kotlin formatting and styling rules
</conditional-block>

## AI Decision Tree

**Creating ViewModel?**
→ Must extend BaseViewModel ([see example](#viewmodel-requirements-critical))
→ Must use @HiltViewModel annotation
→ Must implement subscribe/unsubscribe methods

**Creating Service?**
→ Interface without 'I' prefix ([see example](#service-implementation-pattern))
→ Real + Simulated implementations required
→ Use @Singleton annotation

**File getting large?**
→ Check line count vs 500 limit ([see rule](#file-structure))
→ Refactor if approaching limit

**Adding imports?**
→ Use absolute imports only ([see rule](#import-organization))
→ Never use relative imports

## Table of Contents
- [AI Quick Reference](#ai-quick-reference) ⭐ **READ FIRST**
- [AI Decision Tree](#ai-decision-tree) ⭐ **CRITICAL PATTERNS**
- [ViewModel Requirements](#viewmodel-requirements-critical) ⭐ **MUST FOLLOW**
- [Service Pattern](#service-implementation-pattern) ⭐ **MUST FOLLOW**
- [General Formatting](#general-formatting)
- [Code Organization](#code-organization)
- [Documentation Standards](#documentation-standards)
- [Common Anti-Patterns](#common-anti-patterns-to-avoid) ❌ **AVOID THESE**

## General Formatting

### Indentation and Spacing
- Use 4 spaces for indentation (never tabs)
- Maximum line length: 120 characters
- Use single blank lines to separate logical groups of code
- Place opening braces at the end of the line

```kotlin
sealed class ServiceResult<T, E> {
    
    data class Success<T, E>(val result: T) : ServiceResult<T, E>()
    
    data class Failure<T, E>(val error: E) : ServiceResult<T, E>()
}

inline fun <T, E> ServiceResult<T, E>.fold(
    onSuccess: (value: T) -> Unit,
    onFailure: (error: E) -> Unit
) {
    when (this) {
        is ServiceResult.Failure -> onFailure(error)
        is ServiceResult.Success -> onSuccess(result)
    }
}
```

### Naming Conventions
- **Classes and Interfaces**: PascalCase (e.g., `DeviceViewModel`, `PairingService`)
- **Functions and Properties**: camelCase (e.g., `startPairing`, `deviceState`)
- **Constants**: SCREAMING_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`)
- **Backing Properties**: Prefix with underscore (e.g., `_deviceState`)

```kotlin
class DeviceRepository {
    private val _deviceState = mutableStateOf<DeviceState>(DeviceState.Disconnected)
    val deviceState: State<DeviceState> = _deviceState
    
    companion object {
        const val MAX_CONNECTION_ATTEMPTS = 3
        const val CONNECTION_TIMEOUT_MS = 5000L
    }
}
```

## Project-Specific Architecture Patterns

### ViewModel Requirements (CRITICAL)
All ViewModels MUST extend `BaseViewModel` and implement required methods:

```kotlin
@HiltViewModel
class StartViewModel @Inject constructor(
    private val deviceService: DeviceService,
    private val eventHelper: EventHelper
) : BaseViewModel() {
    
    override fun subscribe() {
        subscribeToCollect(deviceService.connectionState) { state ->
            updateConnectionState(state)
        }
    }
    
    override fun unsubscribe() {
        // Cleanup handled by BaseViewModel
    }
    
    private fun updateConnectionState(state: ConnectionState) {
        // State update logic
    }
}
```

### Service Implementation Pattern
Follow Real/Simulated pattern for service implementations:

```kotlin
// Interface (no 'I' prefix)
interface DeviceService {
    fun connectToDevice(deviceId: String): Flow<ConnectionResult>
    suspend fun getDeviceInfo(): DeviceInfo?
}

// Real implementation
@Singleton
class RealDeviceService @Inject constructor(
    private val sonovaSdk: SonovaMobileSDK
) : DeviceService {
    
    override fun connectToDevice(deviceId: String): Flow<ConnectionResult> = flow {
        emit(ConnectionResult.Connecting)
        try {
            val result = sonovaSdk.connect(deviceId)
            emit(ConnectionResult.Success(result))
        } catch (e: Exception) {
            emit(ConnectionResult.Error(e))
        }
    }
}

// Simulated implementation
@Singleton
class SimulatedDeviceService @Inject constructor() : DeviceService {
    
    override fun connectToDevice(deviceId: String): Flow<ConnectionResult> = flow {
        emit(ConnectionResult.Connecting)
        delay(2000) // Simulate connection time
        emit(ConnectionResult.Success(mockDeviceInfo))
    }
}
```

### Dependency Injection with Hilt
- Use `@HiltAndroidApp` on application class
- Use `@AndroidEntryPoint` on activities/fragments
- Use `@HiltViewModel` on ViewModels
- Use `@Inject` for constructor injection

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object ServiceModule {
    
    @Provides
    @Singleton
    fun provideDeviceService(
        @ApplicationContext context: Context,
        sonovaSdk: SonovaMobileSDK
    ): DeviceService = if (BuildConfig.USE_REAL_SERVICES) {
        RealDeviceService(sonovaSdk)
    } else {
        SimulatedDeviceService()
    }
}
```

## Code Organization

### File Structure
- Maximum file length: 500 lines
- Group related functionality in the same file
- Split large files by feature or responsibility

### Import Organization
- Use absolute imports, never relative
- Group imports: Android framework, third-party libraries, project modules
- Order alphabetically within each group

```kotlin
import android.content.Context
import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.Flow

import com.advancedbionics.palma.models.ServiceResult
import com.advancedbionics.palma.services.DeviceService
```

### Package Structure
Follow the established module structure:
- `app/` - UI, ViewModels, Activities
- `common/` - Shared utilities and extensions
- `controls/` - Reusable UI components
- `data/` - Data persistence layer
- `service_layer/` - Business logic and services

## Documentation Standards

### KDoc Documentation
All public APIs must include KDoc documentation using official Kotlin style:

```kotlin
/**
 * Manages device connection and communication for hearing devices.
 * 
 * This service handles the complete lifecycle of device connectivity,
 * including discovery, pairing, and data exchange.
 */
interface DeviceService {
    
    /**
     * Initiates connection to a specific hearing device.
     * 
     * The [deviceId] should be a unique identifier for the target device.
     * Returns a Flow emitting connection progress and final result.
     */
    fun connectToDevice(deviceId: String): Flow<ConnectionResult>
}
```

### Inline Comments
Add comments for complex business logic:

```kotlin
private fun calculateBatteryPercentage(voltage: Float): Int {
    // Convert voltage to percentage using device-specific curve
    // Voltage range: 3.0V (0%) to 4.2V (100%)
    return ((voltage - 3.0f) / 1.2f * 100).coerceIn(0f, 100f).toInt()
}
```

## Kotlin-Specific Patterns

### Data Classes
Use data classes for simple data containers:

```kotlin
data class DeviceInfo(
    val id: String,
    val name: String,
    val batteryLevel: Int,
    val isConnected: Boolean
)
```

### Extension Functions
Place extensions in appropriate files, prefer member functions when possible:

```kotlin
fun Context.hasPermission(permission: String): Boolean {
    return ContextCompat.checkSelfPermission(this, permission) == PackageManager.PERMISSION_GRANTED
}
```

### Coroutines and Flow
Use structured concurrency and proper error handling:

```kotlin
class DeviceRepository @Inject constructor(
    private val deviceService: DeviceService,
    private val scope: CoroutineScope
) {
    
    fun observeDeviceState(): Flow<DeviceState> = deviceService.deviceUpdates
        .catch { exception ->
            Timber.e(exception, "Error observing device state")
            emit(DeviceState.Error(exception.message))
        }
        .flowOn(Dispatchers.IO)
}
```

### Null Safety
Leverage Kotlin's null safety features:

```kotlin
fun processDevice(device: Device?) {
    device?.let { nonNullDevice ->
        updateDeviceInfo(nonNullDevice)
    } ?: run {
        showNoDeviceMessage()
    }
}
```

## Error Handling

### Exception Handling
Use specific exception types and proper logging:

```kotlin
sealed class DeviceError : Exception() {
    object NotFound : DeviceError()
    object ConnectionFailed : DeviceError()
    data class UnknownError(override val message: String) : DeviceError()
}

suspend fun connectDevice(id: String): Result<Device> = try {
    val device = deviceService.connect(id)
    Result.success(device)
} catch (e: DeviceError.NotFound) {
    Timber.w("Device not found: $id")
    Result.failure(e)
} catch (e: Exception) {
    Timber.e(e, "Unexpected error connecting to device: $id")
    Result.failure(DeviceError.UnknownError(e.message ?: "Unknown error"))
}
```

## Performance Considerations

### Memory Management
- Properly clean up resources in ViewModels
- Use lazy initialization for heavy objects
- Avoid memory leaks with proper lifecycle management

### Concurrency
- Use appropriate dispatchers for different workloads
- Prefer Flow over LiveData for reactive streams
- Use `viewModelScope` for ViewModel-bound operations

```kotlin
@HiltViewModel
class DeviceListViewModel @Inject constructor(
    private val deviceRepository: DeviceRepository
) : BaseViewModel() {
    
    private val _devices = MutableStateFlow<List<Device>>(emptyList())
    val devices: StateFlow<List<Device>> = _devices.asStateFlow()
    
    override fun subscribe() {
        subscribeToCollect(deviceRepository.observeDevices()) { deviceList ->
            _devices.value = deviceList
        }
    }
}
```

## AI Pattern Matching Examples

### ✅ Correct ViewModel Pattern (AI should recognize and replicate)
```kotlin
@HiltViewModel
class DeviceViewModel @Inject constructor(
    private val deviceService: DeviceService
) : BaseViewModel() {
    
    override fun subscribe() {
        subscribeToCollect(deviceService.connectionState) { state ->
            // Handle state updates
        }
    }
    
    override fun unsubscribe() {
        // Cleanup handled by BaseViewModel
    }
}
```

### ❌ Incorrect ViewModel Pattern (AI should never generate)
```kotlin
class DeviceViewModel : ViewModel() { // WRONG: Missing BaseViewModel and @HiltViewModel
    // WRONG: Missing subscribe/unsubscribe methods
}
```

### ✅ Correct Service Pattern (AI should recognize and replicate)
```kotlin
// Interface (no 'I' prefix)
interface DeviceService {
    fun connectToDevice(): Flow<ConnectionResult>
}

// Real implementation
@Singleton
class RealDeviceService @Inject constructor(
    private val sdk: SonovaMobileSDK
) : DeviceService {
    override fun connectToDevice() = flow { /* real implementation */ }
}

// Simulated implementation
@Singleton
class SimulatedDeviceService @Inject constructor() : DeviceService {
    override fun connectToDevice() = flow { /* simulated implementation */ }
}
```

### ❌ Incorrect Service Pattern (AI should never generate)
```kotlin
// WRONG: 'I' prefix
interface IDeviceService { ... }

// WRONG: Missing Real/Simulated pattern
class DeviceService { ... }
```

## AI Validation Checklist

Before generating/modifying Kotlin code, verify:
- [ ] ViewModel extends BaseViewModel? ([see pattern](#ai-pattern-matching-examples))
- [ ] Service follows Real/Simulated pattern? ([see pattern](#ai-pattern-matching-examples))
- [ ] Using absolute imports? ([see rule](#import-organization))
- [ ] File under 500 lines? ([see rule](#file-structure))
- [ ] Lines under 120 characters? ([see rule](#indentation-and-spacing))
- [ ] No 'I' prefix on interfaces? ([see pattern](#ai-pattern-matching-examples))
- [ ] Proper Hilt annotations? (@HiltViewModel, @Singleton)

## Common AI Mistakes to Prevent

### ❌ Critical Error: Creating ViewModel without BaseViewModel
```kotlin
// NEVER generate this pattern
class MyViewModel : ViewModel() {
    // Missing BaseViewModel, @HiltViewModel, subscribe/unsubscribe
}
```
**Fix:** Always use BaseViewModel pattern shown above

### ❌ Critical Error: Service without Real/Simulated pattern
```kotlin
// NEVER generate this pattern
interface IDeviceService { ... } // Wrong: 'I' prefix
class DeviceService { ... }      // Wrong: Missing Real/Simulated
```
**Fix:** Always use Real/Simulated service pattern shown above

### ❌ Critical Error: Relative imports
```kotlin
// NEVER generate this
import ../models/Device // Wrong: relative import
```
**Fix:** Always use absolute imports: `import com.advancedbionics.palma.models.Device`

## Unit Testing with Mocks <!-- KEYWORDS: testing, mock, unit, viewmodel, coroutines, stateflow -->

### Test Class Setup with MockK and Coroutines
```kotlin
@ExperimentalCoroutinesApi
class ListeningCheckLandingViewModelTest : BaseTest() {

    @MockK
    private lateinit var customNavigator: CustomNavigator

    @MockK
    private lateinit var mockGroup: Group

    @MockK(relaxed = true)
    private lateinit var mockLeftDevice: InternalDevice

    @MockK(relaxed = true)
    private lateinit var mockRightDevice: InternalDevice

    private lateinit var viewModel: ListeningCheckLandingViewModel
    private lateinit var mockDeviceBaseService: DeviceBaseService

    // StateFlow mocks for reactive testing
    private val leftDeviceConnectionError = MutableStateFlow(false)
    private val rightDeviceConnectionError = MutableStateFlow(false)
    private val deviceStateMap = MutableStateFlow<Map<HiSide, InternalDeviceState?>>(emptyMap())
    private val serviceState = MutableStateFlow(InternalServiceState.STOPPED)

    @Before
    fun setup() {
        MockKAnnotations.init(this)
    }

    @After
    fun tearDown() {
        try {
            // Un-mock objects and static methods
            unmockkObject(ServiceRepository)
            unmockkStatic("com.advancedbionics.palma.models.devices.InternalDeviceImplKt")

            testScope.advanceUntilIdle()
            Dispatchers.resetMain()
        } catch (_: UninitializedPropertyAccessException) {
            // Test scope was not initialized, ignore
        }
    }
}
```

### Complex Service Mocking with Dependencies
```kotlin
private fun initViewModel(
    testScheduler: TestCoroutineScheduler,
    leftDevice: InternalDevice? = null,
    rightDevice: InternalDevice? = null,
    leftDeviceSupportsListeningCheck: Boolean = true,
    rightDeviceSupportsListeningCheck: Boolean = true
) {
    // Initialize test dispatcher and scope
    testDispatcher = StandardTestDispatcher(testScheduler)
    testScope = TestScope(testDispatcher)
    Dispatchers.setMain(testDispatcher)

    // Mock extension functions
    mockkStatic("com.advancedbionics.palma.models.devices.InternalDeviceImplKt")

    // Create mock services
    mockDeviceBaseService = mockk<DeviceBaseService>(relaxed = true)
    val mockConfigurationService = mockk<ConfigurationService>(relaxed = true)
    val mockDeviceGroupService = mockk<DeviceGroupService>(relaxed = true)

    // Setup device base service with state flows
    every { mockDeviceBaseService.deviceState } returns deviceStateMap
    every { mockDeviceBaseService.serviceState } returns serviceState
    every { mockConfigurationService.deviceBaseService } returns mockDeviceBaseService

    // Mock startService since it's called when supported devices are present
    coEvery { mockConfigurationService.startService() } returns null

    // Setup mock devices with extension functions
    leftDevice?.let { device ->
        every { device.hasConnectionError } returns leftDeviceConnectionError
        every { device.isListeningCheckSupported() } returns leftDeviceSupportsListeningCheck
    }

    // Setup mock group with devices
    val leftDeviceFlow = MutableStateFlow(leftDevice)
    val rightDeviceFlow = MutableStateFlow(rightDevice)

    every { mockGroup.leftDevice } returns leftDeviceFlow
    every { mockGroup.rightDevice } returns rightDeviceFlow

    // Setup device group service with active group
    val activeGroupFlow = MutableStateFlow(mockGroup)
    every { mockDeviceGroupService.activeGroup } returns activeGroupFlow

    // Mock the ServiceRepository singleton
    mockkObject(ServiceRepository)
    every { ServiceRepository.shared.configurationService } returns mockConfigurationService
    every { ServiceRepository.shared.deviceGroupService } returns mockDeviceGroupService
    every { ServiceRepository.shared.onServicesReinitialized } returns MutableStateFlow(Unit)

    // Setup navigator
    coEvery { customNavigator.goBack() } just Runs
    coEvery { customNavigator.navigate(any<AppRoute>()) } just Runs

    // Create real use case instance
    val isListeningCheckDeviceReadyUseCase = IsListeningCheckDeviceReadyUseCase()

    // Initialize the ViewModel with real use case
    viewModel = ListeningCheckLandingViewModel(customNavigator, isListeningCheckDeviceReadyUseCase)
}
```

### Testing StateFlow Updates and Reactive Behavior
```kotlin
@Test
fun `when both devices connected then no connection message shown`() = runTest {
    initViewModel(
        testScheduler,
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice
    )

    val (currentState, collectionJob) = collectStateFlow(viewModel.uiState, testScope)

    // Wait for initial state to settle
    testDispatcher.scheduler.runCurrent()

    // Set devices as connected by updating StateFlow
    updateDeviceStates(
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice,
        leftConnected = true,
        rightConnected = true
    )

    // Process any queued coroutines
    testDispatcher.scheduler.runCurrent()

    waitUntilCondition(testDispatcher) {
        currentState.value.leftDevice != null && currentState.value.rightDevice != null
    }

    // Verify state changes
    assertTrue(currentState.value.isLeftSupported)
    assertTrue(currentState.value.isRightSupported)
    assertFalse(currentState.value.showDeviceConnectingMessage)

    collectionJob.cancel()
}
```

### Helper Functions for Mock State Management
```kotlin
private fun updateDeviceStates(
    leftDevice: InternalDevice?,
    rightDevice: InternalDevice?,
    leftConnected: Boolean,
    rightConnected: Boolean
) {
    val newStateMap = mutableMapOf<HiSide, InternalDeviceState?>()

    // Update device states based on connection status
    if (leftDevice != null) {
        newStateMap[HiSide.LEFT] = if (leftConnected)
            InternalDeviceState.RUNNING
        else
            InternalDeviceState.CONNECTION_ERROR
    }

    if (rightDevice != null) {
        newStateMap[HiSide.RIGHT] = if (rightConnected)
            InternalDeviceState.RUNNING
        else
            InternalDeviceState.CONNECTION_ERROR
    }

    // Force emission by clearing first, then setting new value
    deviceStateMap.value = emptyMap()
    deviceStateMap.value = newStateMap
    serviceState.value = InternalServiceState.RUNNING

    // Mock service method based on device states
    val allDevicesConnected = when {
        leftDevice != null && rightDevice != null -> leftConnected && rightConnected
        leftDevice != null && rightDevice == null -> leftConnected
        leftDevice == null && rightDevice != null -> rightConnected
        else -> false
    }

    every { mockDeviceBaseService.areAllDevicesInActiveGroupRunning() } returns allDevicesConnected
}
```

### Testing Dynamic State Changes
```kotlin
@Test
fun `when device reconnects after disconnection then buttons enabled and message hidden`() = runTest {
    initViewModel(
        testScheduler,
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice
    )

    val (currentState, collectionJob) = collectStateFlow(viewModel.uiState, testScope)

    // Wait for initial state to settle
    testDispatcher.scheduler.runCurrent()

    // Start with device disconnected
    updateDeviceStates(
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice,
        leftConnected = false,
        rightConnected = true
    )

    testDispatcher.scheduler.runCurrent()

    // Verify disconnected state
    assertTrue(currentState.value.showDeviceConnectingMessage)
    assertFalse(currentState.value.isLeftButtonEnabled)

    // Reconnect the device
    updateDeviceStates(
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice,
        leftConnected = true,
        rightConnected = true
    )

    testDispatcher.scheduler.runCurrent()

    // Verify reconnected state
    assertFalse(currentState.value.showDeviceConnectingMessage)
    assertTrue(currentState.value.isLeftButtonEnabled)
    assertTrue(currentState.value.isRightButtonEnabled)

    collectionJob.cancel()
}
```

### Testing Edge Cases and Error Scenarios
```kotlin
@Test
fun `when connection fails with specific error then error description available`() = runTest {
    initViewModel(
        testScheduler,
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice
    )

    val (currentState, collectionJob) = collectStateFlow(viewModel.uiState, testScope)

    testDispatcher.scheduler.runCurrent()

    val errorMessage = "Bluetooth connection timeout"

    // Set specific connection error through StateFlow
    leftDeviceConnectionError.value = true
    leftDeviceConnectionErrorDescription.value = errorMessage

    // Update device states to reflect connection error
    updateDeviceStates(
        leftDevice = mockLeftDevice,
        rightDevice = mockRightDevice,
        leftConnected = false,
        rightConnected = true
    )

    testDispatcher.scheduler.runCurrent()

    // Verify error handling
    assertTrue(currentState.value.showDeviceConnectingMessage)
    assertFalse(currentState.value.isLeftButtonEnabled)

    collectionJob.cancel()
}
```

### Key Unit Testing Patterns
- **Extend BaseTest**: Provides common test infrastructure and utilities
- **Use @MockK Annotations**: Simplifies mock creation with automatic initialization
- **Mock StateFlow/Flow**: Use MutableStateFlow for reactive state testing
- **Mock Singletons**: Use `mockkObject` for ServiceRepository and similar singletons
- **Mock Extension Functions**: Use `mockkStatic` for extension function mocking
- **Test Coroutines**: Use `runTest`, `TestScope`, and `StandardTestDispatcher`
- **Helper Functions**: Create reusable setup functions for complex test scenarios
- **Proper Cleanup**: Always unmock objects and reset dispatchers in tearDown
- **State Collection**: Use helper functions to collect and verify StateFlow emissions
- **Async Testing**: Use `waitUntilCondition` and `runCurrent()` for timing control

## Common Anti-Patterns to Avoid

❌ **Don't create ViewModels without extending BaseViewModel**
❌ **Don't use relative imports**
❌ **Don't use 'I' prefix for interfaces**
❌ **Don't create files > 500 lines**
❌ **Don't ignore null safety**
❌ **Don't mix style changes with business logic**