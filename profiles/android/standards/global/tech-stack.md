## Tech Stack

This project uses native Android development with Jetpack Compose.

### Build System & Language
- **Build System:** Gradle with Kotlin DSL
- **Kotlin Version:** 2.2.20
- **Java Version:** 17 (sourceCompatibility and targetCompatibility)
- **Gradle Version:** 8.13.0

### Android Configuration
- **Min SDK:** 26 (Android 8.0 Oreo)
- **Target SDK:** 35 (Android 15)
- **Compile SDK:** 36
- **Package Manager:** Gradle with Version Catalog (libs.versions.toml)

### UI Framework
- **UI Framework:** Jetpack Compose (BOM 2025.09.01)
- **Material Design:** Material 3
- **Navigation:** Navigation Compose (2.9.5)
- **Constraint Layout:** ConstraintLayout Compose (1.1.1)
- **Icons:** Material Icons Extended
- **Animations:** Lottie Compose (6.6.9)

### Architecture & Dependency Injection
- **DI Framework:** Hilt (2.57.2)
- **Architecture Pattern:** MVVM with UiState/UiEvent
- **ViewModel:** AndroidX Lifecycle ViewModel with Hilt integration
- **Navigation Integration:** Hilt Navigation Compose (1.3.0)

### Async & State Management
- **Coroutines:** Kotlin Coroutines (1.10.2)
- **State Management:** StateFlow + collectAsStateWithLifecycle
- **Lifecycle:** AndroidX Lifecycle Runtime Compose (2.9.4)

### Data & Storage
- **Local Storage:** DataStore Preferences (1.1.7)
- **Serialization:** kotlinx.serialization (1.9.0)
- **Desugaring:** Core Library Desugaring for Java 8+ APIs

### Testing Frameworks
- **Unit Testing:** JUnit 4 (4.13.2)
- **Android Testing:** JUnit Extensions (1.1.5)
- **Mocking:** MockK (1.14.5) for unit tests, MockK Android for instrumented tests
- **Compose Testing:** Compose UI Test with JUnit4 integration
- **Coroutines Testing:** kotlinx-coroutines-test (1.10.2)
- **Test Runner:** AndroidX Test Runner (1.6.1)

### Quality & Monitoring
- **Code Style:** Official Kotlin style (kotlin.code.style=official)
- **Crash Reporting:** Firebase Crashlytics
- **Analytics:** Firebase Analytics

### Module Structure
This is a multi-module project:
- **app:** Main application module
- **theme:** Compose theming and design system
- **common:** Common utilities and helpers
- **controls:** Reusable UI components
- **service_layer:** Business logic and services
- **data:** Data layer with repositories
- **translations:** Localization resources
- **devfinder:** Device finder functionality
