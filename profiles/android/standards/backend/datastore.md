## DataStore Best Practices

### DataStoreManager
- **Central Manager:** Use project's `DataStoreManager` class for all DataStore operations
- **Singleton:** DataStoreManager is provided as singleton via Hilt
- **Named Stores:** Each feature can have its own named DataStore

### Preference Keys
- **Type Safety:** Use typed preference keys
- **Constants:** Define keys as constants
- **Namespacing:** Prefix keys with feature name to avoid conflicts
```kotlin
object PreferenceKeys {
    const val USER_NAME = "user_name"
    const val IS_FIRST_LAUNCH = "is_first_launch"
    const val SETTINGS_VERSION = "settings_version"
}
```

### Reading Data
```kotlin
// Suspend function
suspend fun getString(key: String): String? {
    return dataStoreManager.getString(key)
}

// Flow-based
fun observeString(key: String): Flow<String?> {
    return dataStoreManager.getStringFlow(key)
}
```

### Writing Data
```kotlin
// Suspend function
suspend fun saveString(key: String, value: String) {
    dataStoreManager.putString(key, value)
}
```

### Serialization
- **kotlinx.serialization:** Use for complex objects
- **JSON Format:** Store serialized objects as JSON strings
- **Type Safety:** Define serializable data classes
```kotlin
@Serializable
data class UserSettings(
    val theme: String,
    val notificationsEnabled: Boolean,
    val fontSize: Int
)

suspend fun saveUserSettings(settings: UserSettings) {
    val json = Json.encodeToString(settings)
    dataStoreManager.putString("user_settings", json)
}
```

### Encryption
- **Sensitive Data:** Encrypt sensitive data before storing
- **StringEncryptor:** Use project's `StringEncryptor` utility
- **Keystore Integration:** Encryption uses Android Keystore

### Best Practices
- **Don't Block Main Thread:** Always use suspend functions or Flows
- **Transaction Safety:** Use edit { } block for multiple related updates
- **Avoid Over-Writing:** Only write when value actually changes
- **Error Handling:** Catch and handle serialization/deserialization errors
- **No SharedPreferences:** Migrate from SharedPreferences to DataStore
