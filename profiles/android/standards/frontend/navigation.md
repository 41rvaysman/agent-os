## Navigation Best Practices

### Navigation Setup
- **Custom Navigator:** Use project's `CustomNavigator` abstraction
- **Injection:** Inject `CustomNavigator` into ViewModels via Hilt
- **Sealed Class Routes:** Define navigation destinations as sealed class `AppRoute`

### AppRoute Definition
```kotlin
sealed class AppRoute {
    data object Home : AppRoute()
    data object Settings : AppRoute()
    data class DeviceDetails(val deviceId: String) : AppRoute()
}
```

### Navigation in ViewModel
```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val navigator: CustomNavigator
) : BaseViewModel() {

    fun onEvent(event: UiEvent) {
        when (event) {
            is UiEvent.NavigateToSettings -> {
                navigator.navigate(AppRoute.Settings)
            }
            is UiEvent.NavigateToDevice -> {
                navigator.navigate(AppRoute.DeviceDetails(event.deviceId))
            }
            is UiEvent.GoBack -> {
                navigator.goBack()
            }
        }
    }
}
```

### Navigation in Composables
```kotlin
@Composable
fun MyView(viewModel: MyViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()

    MyViewContent(
        state = state,
        event = viewModel::onEvent
    )
}

@Composable
private fun MyViewContent(
    state: MyViewModel.UiState,
    event: (MyViewModel.UiEvent) -> Unit
) {
    Button(onClick = { event(UiEvent.NavigateToSettings) }) {
        Text("Go to Settings")
    }
}
```

### Back Navigation
- **Go Back:** Use `navigator.goBack()` for back navigation
- **TopBar Integration:** Connect back button to GoBack event
```kotlin
Scaffold(topBar = {
    TopBar(title = "Screen Title") {
        event(UiEvent.GoBack)
    }
}) { padding ->
    // Content
}
```

### Navigation with Arguments
```kotlin
// Define route with parameters
data class DeviceDetails(
    val deviceId: String,
    val showSettings: Boolean = false
) : AppRoute()

// Navigate
navigator.navigate(
    AppRoute.DeviceDetails(
        deviceId = device.id,
        showSettings = true
    )
)
```

### Best Practices
- **Navigation in ViewModel:** Always handle navigation in ViewModel, not in Composables
- **Events for Navigation:** Use UiEvents to trigger navigation
- **Type-Safe Routes:** Use sealed classes for type-safe routes
- **No Direct Access:** Don't pass NavController to Composables
- **Back Stack:** Consider back stack behavior when designing navigation
- **Single Source:** Navigator is single source for navigation logic
