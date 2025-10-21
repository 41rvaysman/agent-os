## Repository Pattern

### Repository Structure
- **Interface Definition:** Define repository as interface
- **Implementation:** Create implementation class with `Impl` suffix
- **Hilt Module:** Provide repository via Hilt module
- **Single Responsibility:** Each repository manages one domain area

### Pattern Example
```kotlin
// Interface
interface SettingsRepository {
    suspend fun getSetting(key: String): String?
    suspend fun saveSetting(key: String, value: String)
    val settingsFlow: Flow<Map<String, String>>
}

// Implementation
class SettingsRepositoryImpl(
    private val dataStoreManager: DataStoreManager
) : SettingsRepository {
    override suspend fun getSetting(key: String): String? {
        return dataStoreManager.getString(key)
    }

    override suspend fun saveSetting(key: String, value: String) {
        dataStoreManager.putString(key, value)
    }

    override val settingsFlow: Flow<Map<String, String>> =
        dataStoreManager.getAllSettings()
}

// Hilt Module
@Module
@InstallIn(SingletonComponent::class)
object SettingsRepositoryModule {
    @Singleton
    @Provides
    fun provideSettingsRepository(
        dataStoreManager: DataStoreManager
    ): SettingsRepository = SettingsRepositoryImpl(dataStoreManager)
}
```

### DataStore Usage
- **DataStoreManager:** Use project's `DataStoreManager` for data access
- **Preferences:** Use DataStore Preferences for key-value storage
- **Type Safety:** Define preference keys with proper types
- **Flow-Based:** Expose data as Flow for reactive updates

### Repository Methods
- **Suspend Functions:** Use `suspend` for all I/O operations
- **Flow Exposure:** Expose data streams as `Flow<T>`
- **StateFlow for State:** Use `StateFlow` for stateful data
- **SharedFlow for Events:** Use `SharedFlow` for one-time events

### Best Practices
- **Keep It Simple:** Repository should be thin layer over data source
- **No Business Logic:** Business logic belongs in Use Cases or ViewModels
- **Inject DataStoreManager:** Don't create DataStore instances directly
- **Thread Safety:** DataStore is thread-safe, use it for concurrent access
- **Clear Naming:** Method names should clearly describe what they do
