## Error Handling & Logging

### Error State Representation

#### Service-Level Errors
Service errors use a standardized error event pattern with type, description, and severity.

```kotlin
// ✅ Good - Service error event structure
data class SdkServiceErrorEvent(
    val type: SdkServiceErrorType,
    val description: String,
    val severity: HiSeverity
)

enum class SdkServiceErrorType {
    // Connection & Network
    CONNECTION_ERROR,
    NO_NETWORK_ERROR,
    NOT_CONNECTED,

    // Configuration & State
    CONFIGURATION_FAILURE,
    INVALID_STATE,
    INVALID_ARGUMENT,
    INVALID_DEVICE_PASSED,

    // Device & Compatibility
    INCOMPATIBLE_DEVICES,
    INCOMPATIBLE_FIRMWARE_VERSIONS,
    DEVICE_TYPE_CI_BIMODAL_HI,
    BATTERY_LOW,

    // Service Operations
    PAIRING_FAILURE,
    DISCOVERY_FAILURE,
    REMOTE_CONTROL_FAILURE,
    REMOTE_SUPPORT_FAILURE,

    // Request & Timing
    REQUEST_TIMEOUT,
    REQUEST_CANCELED,
    OPERATION_IN_PROGRESS,

    // Authentication & Security
    RID_AUTHENTICATION_ERROR,
    MISSING_RID,

    // System
    SDK_ERROR,
    INTERNAL_ERROR,
    SERVICE_IS_NOT_RUNNING,
    UNSUPPORTED,
    UNAVAILABLE
}

enum class HiSeverity {
    INFO,
    WARNING,
    ERROR,
    CRITICAL
}
```

#### ViewModel Error States
Error information should be part of the UiState, not separate error channels.

```kotlin
// ✅ Excellent - Error as part of UiState (ABApp pattern)
@HiltViewModel
class HomeViewModel @Inject constructor() : BaseViewModel() {

    data class UiState(
        val showTurnBTOnDialog: Boolean = false,
        val showConnectionErrorHelpDialog: Boolean = false,
        val showMissingPermissionDialog: Boolean = false,
        val showSdkErrorDialog: Boolean = false,
        val sdkError: SdkServiceErrorEvent? = null,
        val monitoringServiceState: InternalServiceState,
        val remoteControlServiceState: InternalServiceState,
        val snackBarService: LocalSnackbarService = LocalSnackbarService()
    )

    private val _uiState = MutableStateFlow(UiState(
        remoteControlServiceState = ServiceRepository.shared.remoteControlService.deviceBaseService.serviceState.value,
        monitoringServiceState = ServiceRepository.shared.monitoringService.deviceBaseService.serviceState.value
    ))
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    override fun subscribe() {
        // Subscribe to service error events
        subscribeToCollect(ServiceRepository.shared.remoteControlService.deviceBaseService.onServiceError) { errorEvent ->
            handleRemoteControlServiceError(errorEvent)
        }
    }

    private fun handleRemoteControlServiceError(error: SdkServiceErrorEvent?) {
        _uiState.update {
            it.copy(
                sdkError = error,
                showSdkErrorDialog = error != null
            )
        }
    }

    fun onSdkAlertDialogConfirmClick() {
        _uiState.update { it.copy(showSdkErrorDialog = false) }
        // Handle retry or recovery based on error type
        when (uiState.value.sdkError?.type) {
            SdkServiceErrorType.NO_NETWORK_ERROR -> retryConnection()
            SdkServiceErrorType.INCOMPATIBLE_DEVICES -> navigateToSettings()
            else -> clearError()
        }
    }

    fun onSdkAlertDialogDismissClick() {
        _uiState.update { it.copy(showSdkErrorDialog = false, sdkError = null) }
    }
}

// ⚠️ Acceptable - Simple error string for basic errors
data class UiState(
    val data: List<Item> = emptyList(),
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)

// ❌ Bad - Separate error channel (harder to manage state)
sealed class UiEvent {
    data class ShowError(val message: String) : UiEvent
}

// ❌ Bad - Generic Boolean without context
data class UiState(
    val hasError: Boolean = false  // What error? How to display it?
)
```

#### Error Display Patterns

```kotlin
// ✅ Good - Error dialog with SdkAlertDialog
@Composable
fun HomeView(viewModel: HomeViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()

    HomeContent(state = state, event = viewModel::onUiEvent)

    // Standardized error dialog
    SdkAlertDialog(
        showDialog = state.showSdkErrorDialog,
        sdkServiceErrorEvent = state.sdkError,
        onConfirmClick = viewModel::onSdkAlertDialogConfirmClick,
        onDismissClick = viewModel::onSdkAlertDialogDismissClick
    )
}

// SdkAlertDialog maps error types to localized messages
@Composable
fun SdkAlertDialog(
    showDialog: Boolean,
    sdkServiceErrorEvent: SdkServiceErrorEvent?,
    onConfirmClick: () -> Unit,
    onDismissClick: () -> Unit,
) {
    sdkServiceErrorEvent?.let {
        SimpleAlterDialog(
            showDialog = showDialog,
            title = getDialogTitle(sdkServiceErrorEvent),
            text = getDialogDescription(sdkServiceErrorEvent),
            confirmButtonText = getConfirmButtonText(sdkServiceErrorEvent),
            dismissButtonText = getDismissButtonText(sdkServiceErrorEvent),
            onConfirmClick = onConfirmClick,
            onDismissClick = onDismissClick
        )
    }
}

@Composable
private fun getDialogTitle(sdkServiceErrorEvent: SdkServiceErrorEvent): String =
    when (sdkServiceErrorEvent.type) {
        SdkServiceErrorType.INCOMPATIBLE_DEVICES -> stringResource(R.string.rc_incompatible_settings_title)
        SdkServiceErrorType.NO_NETWORK_ERROR -> stringResource(R.string.no_internet_connection_title)
        SdkServiceErrorType.BATTERY_LOW -> stringResource(R.string.battery_low_title)
        else -> sdkServiceErrorEvent.type.name
    }

@Composable
private fun getDialogDescription(sdkServiceErrorEvent: SdkServiceErrorEvent): String =
    when (sdkServiceErrorEvent.type) {
        SdkServiceErrorType.INCOMPATIBLE_DEVICES -> stringResource(R.string.rc_incompatible_settings_description)
        SdkServiceErrorType.NO_NETWORK_ERROR -> stringResource(R.string.rc_no_internet_connection_description)
        else -> sdkServiceErrorEvent.description
    }
```

### Exception Handling

#### Service Layer Exception Handling
Services should catch exceptions, log them, and emit error events rather than crashing.

```kotlin
// ✅ Excellent - Service exception handling (ABApp pattern)
internal class RealPairingService : BaseService(), PairingService {

    private val _onDevicePairingFailed = MutableStateFlow<PairingErrorEventData?>(null)
    override val onDevicePairingFailed = _onDevicePairingFailed.asStateFlow()

    override fun startDiscovery() {
        // Exit if we are in a pairing or discovery mode
        if (serviceState.value != PairingServiceState.IDLE) {
            return
        }

        // Reset error states
        _onDeviceDiscovered.update { null }
        _onDeviceDiscoveredWithError.update { null }

        ServiceRepository.shared.logger.info(TAG, "startDiscovery started")

        unregisterObservers()
        registerObserver(discoveryService.state.register { state ->
            _serviceState.value = if (state == ServiceState.STARTING || state == ServiceState.RUNNING) {
                PairingServiceState.IN_DISCOVERY
            } else {
                PairingServiceState.IDLE
            }
        })

        try {
            discoveryService.start(
                { onDiscoveryEvent(it) },
                { onDiscoveryResult(it) }
            )
        } catch (e: Exception) {
            // Log to Crashlytics for non-fatal tracking
            ServiceRepository.shared.crashlyticsLogger.recordException(e)
            // Log to standard logger for debugging
            ServiceRepository.shared.logger.error(TAG, e.localizedMessage ?: "Discovery start failed")
            // Emit error event for UI handling
            _onDevicePairingFailed.update {
                PairingErrorEventData(
                    errorType = PairingErrorType.DISCOVERY_ERROR,
                    message = e.localizedMessage ?: "Failed to start discovery"
                )
            }
        }
    }

    private fun onDiscoveryResult(result: AsyncResult<Unit>) {
        when (result) {
            is AsyncResult.Success -> {
                ServiceRepository.shared.logger.info(TAG, "Discovery successful")
            }
            is AsyncResult.Failure -> {
                ServiceRepository.shared.logger.error(TAG, "Discovery failed: ${result.error}")
                ServiceRepository.shared.crashlyticsLogger.recordException(
                    Exception("Discovery failed: ${result.error}")
                )
            }
        }
    }
}

// ❌ Bad - Poor exception handling without logging or user feedback
try {
    discoveryService.start()
} catch (e: Exception) {
    _uiState.update { it.copy(hasError = true) }
    // No logging, no context, no Crashlytics reporting
}
```

#### ViewModel Exception Handling

```kotlin
// ✅ Good - ViewModel handles errors with logging and state updates
@HiltViewModel
class SettingsViewModel @Inject constructor(
    private val saveSettingsUseCase: SaveSettingsUseCase
) : BaseViewModel() {

    data class UiState(
        val isSaving: Boolean = false,
        val saveError: String? = null,
        val saveSuccess: Boolean = false
    )

    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    fun saveSettings(settings: Settings) {
        runInIOThread {
            _uiState.update { it.copy(isSaving = true, saveError = null) }

            try {
                saveSettingsUseCase(settings)
                _uiState.update { it.copy(isSaving = false, saveSuccess = true) }
                ServiceRepository.shared.logger.info(TAG, "Settings saved successfully")
            } catch (e: Exception) {
                ServiceRepository.shared.crashlyticsLogger.recordException(e)
                ServiceRepository.shared.logger.error(TAG, "Failed to save settings: ${e.localizedMessage}")
                _uiState.update {
                    it.copy(
                        isSaving = false,
                        saveError = e.localizedMessage ?: "Failed to save settings"
                    )
                }
            }
        }
    }

    // ✅ Good - Handle specific exception types with different recovery strategies
    fun loadData() {
        runInIOThread {
            try {
                val data = repository.loadData()
                _uiState.update { it.copy(data = data, isLoading = false) }
            } catch (e: IOException) {
                ServiceRepository.shared.logger.error(TAG, "Network error: ${e.localizedMessage}")
                _uiState.update {
                    it.copy(
                        isLoading = false,
                        errorType = ErrorType.NETWORK,
                        errorMessage = "Check your internet connection"
                    )
                }
            } catch (e: Exception) {
                ServiceRepository.shared.crashlyticsLogger.recordException(e)
                _uiState.update {
                    it.copy(
                        isLoading = false,
                        errorType = ErrorType.UNKNOWN,
                        errorMessage = "An unexpected error occurred"
                    )
                }
            }
        }
    }
}
```

### Logging

#### Logger Usage (ABApp Pattern)
The project uses a dual logging approach: standard logging for development and Crashlytics for production monitoring.

```kotlin
// ✅ Excellent - Access loggers via ServiceRepository
internal class RealMonitoringService : BaseService(), MonitoringService {

    companion object {
        private const val TAG = "RealMonitoringService"
    }

    fun connectToDevice(device: InternalDevice) {
        // Info logging for normal flow
        ServiceRepository.shared.logger.info(TAG, "Connecting to device: ${device.serialNumber}")

        try {
            val result = sdkService.connect(device.sdkDevice)

            when (result) {
                is AsyncResult.Success -> {
                    ServiceRepository.shared.logger.info(TAG, "Connected successfully")
                }
                is AsyncResult.Failure -> {
                    // Error logging for failures
                    ServiceRepository.shared.logger.error(TAG, "Connection failed: ${result.error}")
                }
            }
        } catch (e: Exception) {
            // Exception logging with Crashlytics
            ServiceRepository.shared.crashlyticsLogger.recordException(e)
            ServiceRepository.shared.logger.error(TAG, "Exception during connection: ${e.localizedMessage}")
        }
    }
}

// ✅ Good - Severity-based logging
fun processDeviceUpdate(update: DeviceUpdate) {
    when {
        update.batteryLevel < 10 -> {
            // Warning for concerning but non-critical events
            ServiceRepository.shared.logger.warning(TAG, "Low battery: ${update.batteryLevel}%")
        }
        update.isConnected -> {
            // Info for normal operations
            ServiceRepository.shared.logger.info(TAG, "Device connected")
        }
        update.hasError -> {
            // Error for failures
            ServiceRepository.shared.logger.error(TAG, "Device error: ${update.errorMessage}")
        }
    }
}

// ✅ Good - Debug logging for development
fun debugDeviceState(device: InternalDevice) {
    if (BuildConfig.DEBUG) {
        ServiceRepository.shared.logger.debug(
            TAG,
            "Device state - Connected: ${device.isConnected}, Battery: ${device.batteryLevel}%"
        )
    }
}

// ❌ Bad - Common logging mistakes
Log.d("MyTag", "Some message")  // Use ServiceRepository.shared.logger, not Android Log

ServiceRepository.shared.logger.info("RealPairingService", "Message")  // Use TAG constant

try {
    riskyOperation()
} catch (e: Exception) {
    // Missing Crashlytics reporting and error logging
}
```

#### Crashlytics Integration

```kotlin
// ✅ Excellent - Crashlytics for non-fatal exceptions
fun syncData() {
    runInIOThread {
        try {
            val data = networkService.fetchData()
            saveToDatabase(data)
        } catch (e: Exception) {
            // Record non-fatal exception for monitoring
            ServiceRepository.shared.crashlyticsLogger.recordException(e)

            // Add context with custom keys
            ServiceRepository.shared.crashlyticsLogger.setCustomKey("operation", "sync_data")
            ServiceRepository.shared.crashlyticsLogger.setCustomKey("timestamp", System.currentTimeMillis())

            // Also log locally for debugging
            ServiceRepository.shared.logger.error(TAG, "Data sync failed: ${e.localizedMessage}")

            // Update UI state
            _uiState.update { it.copy(syncError = e.localizedMessage) }
        }
    }
}

// ✅ Good - Crashlytics breadcrumbs and custom keys for context
fun performComplexOperation(device: InternalDiscoveredDevice) {
    // Add custom context
    ServiceRepository.shared.crashlyticsLogger.setCustomKey("device_serial", device.serialNumber)
    ServiceRepository.shared.crashlyticsLogger.setCustomKey("operation", "complex_operation")

    ServiceRepository.shared.crashlyticsLogger.log("Starting complex operation")

    try {
        step1()
        ServiceRepository.shared.crashlyticsLogger.log("Step 1 completed")

        step2()
        ServiceRepository.shared.crashlyticsLogger.log("Step 2 completed")
    } catch (e: Exception) {
        // Crashlytics will include breadcrumb trail and custom keys
        ServiceRepository.shared.crashlyticsLogger.recordException(e)
        ServiceRepository.shared.logger.error(TAG, "Operation failed: ${e.localizedMessage}")
    }
}
```

#### Logger Interface

```kotlin
// Project's logger interface with severity levels
enum class MessageSeverity {
    VERBOSE, DEBUG, INFO, WARNING, ERROR
}

// Access via ServiceRepository.shared.logger
// Available methods: verbose(), debug(), info(), warning(), error()
```

### User-Facing Error Messages

#### Snackbar for Transient Errors

```kotlin
// ✅ Excellent - Snackbar service for non-blocking notifications
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val displayConnectionSuccessSnackbarUseCase: DisplayConnectionSuccessSnackbarUseCase,
    private val displayConnectionTakesTimeSnackbarUseCase: DisplayConnectionTakesTimeSnackbarUseCase
) : BaseViewModel() {

    data class UiState(
        val snackBarService: LocalSnackbarService = LocalSnackbarService()
    )

    override fun subscribe() {
        subscribeToCollect(ServiceRepository.shared.remoteControlService.deviceBaseService.onAllDevicesConnected) {
            displayConnectionSuccessSnackbarUseCase(uiState.value.snackBarService)
        }

        subscribeToCollect(ServiceRepository.shared.remoteControlService.deviceBaseService.connectionTakesTime) { takesTime ->
            if (takesTime) {
                displayConnectionTakesTimeSnackbarUseCase(uiState.value.snackBarService) {
                    _uiState.update { it.copy(showConnectionErrorHelpDialog = true) }
                }
            }
        }
    }
}

// Use cases encapsulate snackbar display logic
class DisplayConnectionSuccessSnackbarUseCase @Inject constructor() {
    operator fun invoke(snackBarService: LocalSnackbarService) {
        snackBarService.show(
            SnackbarItem(
                type = SnackBarItemType.SUCCESS,
                message = "Connected successfully",
                duration = 3000
            )
        )
    }
}

// Snackbar with action button
class DisplayConnectionTakesTimeSnackbarUseCase @Inject constructor() {
    operator fun invoke(snackBarService: LocalSnackbarService, onActionClick: () -> Unit) {
        snackBarService.show(
            SnackbarItem(
                type = SnackBarItemType.WARNING,
                message = "Connection taking longer than expected",
                actionText = "Help",
                onActionClick = onActionClick,
                duration = 5000
            )
        )
    }
}
```

#### Dialog for Critical Errors

```kotlin
// ✅ Good - Dialog for errors requiring user acknowledgment
data class UiState(
    val showPermissionDeniedDialog: Boolean = false,
    val showBluetoothOffDialog: Boolean = false,
    val showSdkErrorDialog: Boolean = false,
    val sdkError: SdkServiceErrorEvent? = null
)

// In composable
if (state.showBluetoothOffDialog) {
    SimpleAlterDialog(
        showDialog = true,
        title = stringResource(R.string.bluetooth_off_title),
        text = stringResource(R.string.bluetooth_off_message),
        confirmButtonText = stringResource(R.string.turn_on),
        dismissButtonText = stringResource(R.string.cancel),
        onConfirmClick = viewModel::onTurnOnBluetoothClick,
        onDismissClick = viewModel::onDismissBluetoothDialog
    )
}
```

### Error Recovery Patterns

#### Retry Mechanisms

```kotlin
// ✅ Good - Retry with exponential backoff
@HiltViewModel
class ConnectionViewModel @Inject constructor() : BaseViewModel() {

    data class UiState(
        val isConnecting: Boolean = false,
        val connectionError: String? = null,
        val retryCount: Int = 0,
        val maxRetries: Int = 3
    )

    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    fun connect() {
        if (uiState.value.retryCount >= uiState.value.maxRetries) {
            _uiState.update {
                it.copy(
                    connectionError = "Maximum retry attempts reached. Please check your connection."
                )
            }
            return
        }

        runInIOThread {
            _uiState.update { it.copy(isConnecting = true, connectionError = null) }

            try {
                ServiceRepository.shared.deviceService.connect()
                _uiState.update { it.copy(isConnecting = false, retryCount = 0) }
                ServiceRepository.shared.logger.info(TAG, "Connection successful")
            } catch (e: Exception) {
                val newRetryCount = uiState.value.retryCount + 1
                val backoffDelay = calculateBackoffDelay(newRetryCount)

                ServiceRepository.shared.logger.warning(
                    TAG,
                    "Connection failed (attempt $newRetryCount), retrying in ${backoffDelay}ms"
                )

                delay(backoffDelay)
                _uiState.update { it.copy(retryCount = newRetryCount, isConnecting = false) }

                // Retry
                connect()
            }
        }
    }

    private fun calculateBackoffDelay(retryCount: Int): Long {
        return (1000 * (2.0.pow(retryCount - 1))).toLong()  // 1s, 2s, 4s
    }

    fun resetRetry() {
        _uiState.update { it.copy(retryCount = 0, connectionError = null) }
    }
}
```

#### Fallback Strategies

```kotlin
// ✅ Good - Graceful degradation
fun loadUserData(userId: String) {
    runInIOThread {
        try {
            // Try network first
            val data = networkRepository.getUserData(userId)
            cacheRepository.saveUserData(data)
            _uiState.update { it.copy(userData = data, isFromCache = false) }
        } catch (e: IOException) {
            ServiceRepository.shared.logger.warning(TAG, "Network unavailable, loading from cache")

            try {
                // Fallback to cache
                val cachedData = cacheRepository.getUserData(userId)
                _uiState.update {
                    it.copy(
                        userData = cachedData,
                        isFromCache = true,
                        warning = "Showing cached data. Check your connection."
                    )
                }
            } catch (e: Exception) {
                ServiceRepository.shared.logger.error(TAG, "Both network and cache failed")
                _uiState.update {
                    it.copy(
                        error = "Unable to load data. Please try again later."
                    )
                }
            }
        }
    }
}
```

### Best Practices Summary

#### Error States
- Include error information in UiState (not separate channels)
- Use typed error objects (SdkServiceErrorEvent) instead of strings
- Provide context: error type, message, severity
- Include recovery options in error state (retry count, suggested actions)

#### Exception Handling
- Always catch exceptions in services and ViewModels
- Log exceptions to both standard logger and Crashlytics
- Never swallow exceptions silently
- Provide user-facing error messages
- Emit error events for UI handling

#### Logging
- Access loggers via `ServiceRepository.shared.logger` and `ServiceRepository.shared.crashlyticsLogger`
- Use TAG constants for consistency
- Choose appropriate severity levels (info, warning, error)
- Add Crashlytics breadcrumbs for complex operations
- Use custom keys for additional context

#### User Communication
- Use snackbars for transient, non-critical notifications
- Use dialogs for errors requiring user acknowledgment
- Provide actionable error messages (not just "Error occurred")
- Map technical errors to user-friendly messages
- Offer recovery actions (retry, open settings, etc.)

#### Recovery
- Implement retry with exponential backoff
- Provide fallback strategies (cache, default values)
- Track retry attempts to prevent infinite loops
- Clear error state on successful recovery
- Log recovery attempts for monitoring

### ABApp-Specific Patterns

#### Service Error Flow
```kotlin
// Service emits error event
private val _onServiceError = MutableStateFlow<SdkServiceErrorEvent?>(null)
override val onServiceError = _onServiceError.asStateFlow()

// ViewModel subscribes to error
override fun subscribe() {
    subscribeToCollect(ServiceRepository.shared.service.onServiceError) { error ->
        handleServiceError(error)
    }
}

// ViewModel updates UI state
private fun handleServiceError(error: SdkServiceErrorEvent?) {
    _uiState.update {
        it.copy(sdkError = error, showSdkErrorDialog = error != null)
    }
}

// UI displays error dialog
SdkAlertDialog(
    showDialog = state.showSdkErrorDialog,
    sdkServiceErrorEvent = state.sdkError,
    onConfirmClick = viewModel::onErrorConfirm,
    onDismissClick = viewModel::onErrorDismiss
)
```

#### Permission & System State Errors
```kotlin
// Check for missing permissions
if (!hasPermission) {
    _uiState.update { it.copy(showMissingPermissionDialog = true) }
    return
}

// Check for Bluetooth off
if (!isBluetoothOn) {
    _uiState.update { it.copy(showTurnBTOnDialog = true) }
    return
}

// Provide system settings access
fun openAppSettings(context: Context) {
    _uiState.update { it.copy(showMissingPermissionDialog = false) }
    openAppSettingsUseCase {
        // Re-check permissions after settings
    }
}
```
