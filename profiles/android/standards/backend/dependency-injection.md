## Dependency Injection with Hilt

### Module Structure
- **One Module per Dependency Type:** Create separate `@Module` objects for each dependency type
- **Descriptive Names:** Name modules clearly: `*RepositoryModule`, `*ServiceModule`, `*UseCaseModule`
- **SingletonComponent:** Use `@InstallIn(SingletonComponent::class)` for app-level dependencies
- **Module Objects:** Define modules as Kotlin `object`

### Module Pattern
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object MyRepositoryModule {
    @Singleton
    @Provides
    fun provideMyRepository(
        dataStoreManager: DataStoreManager
    ): MyRepository = MyRepositoryImpl(dataStoreManager)
}
```

### Scoping
- **@Singleton:** Use for app-level single instances
- **Scope Appropriately:** Match scope to component lifecycle
- **ViewModels:** Use `@HiltViewModel` annotation, automatically scoped

### Constructor Injection
- **Prefer Constructor Injection:** Use constructor injection over field injection
- **@Inject Constructor:** Annotate constructors with `@Inject` for auto-wiring
- **Interface Binding:** Provide interfaces via @Provides methods

### Context Injection
- **@ApplicationContext:** Use `@ApplicationContext` qualifier for Application context
- **Avoid Activity Context:** Use Application context unless specifically needed
```kotlin
fun provideRepository(
    @ApplicationContext context: Context
): MyRepository
```

### ViewModel Injection
- **@HiltViewModel:** Annotate ViewModels with `@HiltViewModel`
- **@Inject Constructor:** Use constructor injection for ViewModel dependencies
- **hiltViewModel():** Retrieve in Composables using `hiltViewModel()`
```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: MyRepository,
    private val navigator: CustomNavigator
) : BaseViewModel()
```

### Best Practices
- **Avoid Static:** Don't use singleton objects, use Hilt-provided singletons
- **Module Organization:** Group related providers in same module
- **Minimize @Provides:** Use constructor injection when possible
- **Clear Dependencies:** Make dependency graph obvious and documented
- **No Dagger Modules:** Stick to Hilt modules and components
