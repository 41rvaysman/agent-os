## Conventions

### File Organization
- **Copyright Headers:** All files must include copyright notice
```kotlin
/*
 * Copyright (c) 2025. Advanced Bionics
 */
```

### Naming Standards
- **View Files:** Named with `View` suffix (e.g., `SettingsView.kt`)
- **ViewModel Files:** Named with `ViewModel` suffix (e.g., `SettingsViewModel.kt`)
- **Repository Interfaces:** Named with `Repository` suffix (e.g., `SettingsRepository`)
- **Repository Implementations:** Named with `RepositoryImpl` suffix (e.g., `SettingsRepositoryImpl`)
- **Use Cases:** Named with `UseCase` suffix (e.g., `LoadDataUseCase`)

### Package Structure
- **Feature-Based:** Organize by feature under `ui/[feature]/`
- **ViewModels:** Place in `viewmodel/` subpackage
- **Sub-Views:** Complex features use `subview/` directory
- **Shared Code:** Place in appropriate module (`common`, `controls`, etc.)

### Dependency Direction
- App module depends on all feature modules
- Feature modules depend on `common` and `theme`
- Data module is standalone
- No circular dependencies between modules

### Code Organization
- **Properties First:** Declare properties before functions
- **Public Before Private:** Public members before private members
- **Related Code Together:** Group related functionality

### State Management
- **Single StateFlow:** One StateFlow per ViewModel for UI state
- **Data Classes:** Use data classes for state
- **Immutable State:** State updates via `.copy()`
- **Default Values:** All state properties have sensible defaults

### Event Handling
- **Sealed Classes:** Events defined as sealed interface/class
- **Single Handler:** One `onEvent()` method handles all events
- **Navigation via Events:** All navigation triggered through events

### Error Handling
- **Try-Catch:** Wrap I/O and network operations
- **User-Facing Messages:** Show meaningful error messages to users
- **Logging:** Log errors for debugging

### Testing Requirements
- **Unit Tests:** All ViewModels must have unit tests
- **UI Tests:** Critical user flows must have UI tests
- **Test Naming:** Use descriptive backtick names
- **Given-When-Then:** Structure tests with clear sections
