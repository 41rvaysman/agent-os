## Project Structure

### Multi-Module Architecture
This project uses a multi-module architecture with the following modules:

#### App Module (`app/`)
- Main application module
- Contains UI screens, ViewModels, and navigation
- Depends on all other modules
- Package structure: `com.advancedbionics.palma.ui.[feature]/`

#### Theme Module (`theme/`)
- Material 3 theming and design system
- Custom color schemes, typography, spacing
- Composition locals for spacing and dimensions
- No dependencies on other feature modules

#### Common Module (`common/`)
- Shared utilities and helpers
- Common extensions
- Base classes used across modules

#### Controls Module (`controls/`)
- Reusable UI components
- Custom composables used in multiple screens
- Design system components

#### Service Layer Module (`service_layer/`)
- Business logic and services
- Service interfaces and implementations
- SDK integrations

#### Data Module (`data/`)
- Data layer with repositories
- DataStore management
- Data models and serialization

### Package Organization
```
com.advancedbionics.palma/
├── ui/
│   ├── [feature]/
│   │   ├── [FeatureName]View.kt
│   │   ├── viewmodel/
│   │   │   └── [FeatureName]ViewModel.kt
│   │   └── subview/
│   ├── theme/
│   └── viewmodel/
│       └── BaseViewModel.kt
├── di/
│   └── Modules.kt
├── navigation/
├── helpers/
├── models/
├── repositories/
├── services/
└── usecases/
```

### Feature Organization
- **By Feature:** UI organized by feature (pairing, settings, devicehealth, etc.)
- **View Files:** Composable screens in feature root
- **ViewModel Package:** ViewModels in `viewmodel/` subpackage
- **Sub-Views:** Complex features have `subview/` directory
- **Shared Components:** Common UI in `controls` module

### File Naming Conventions
- **Views:** `*View.kt` (e.g., `SettingsView.kt`)
- **ViewModels:** `*ViewModel.kt` (e.g., `SettingsViewModel.kt`)
- **Base Classes:** `Base*.kt` (e.g., `BaseViewModel.kt`)
- **Repositories:** `*Repository.kt` and `*RepositoryImpl.kt`
- **Use Cases:** `*UseCase.kt`

### Best Practices
- **Module Independence:** Each module should be independently compilable
- **Clear Boundaries:** Respect module boundaries
- **Shared Resources:** Put truly shared code in `common`, not duplicated
- **Feature Isolation:** Features should not directly depend on other features
- **Testing:** Each module has its own tests
