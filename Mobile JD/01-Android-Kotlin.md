---
title: Android Development with Kotlin
date: 2026-01-31
tags:
  - android
  - kotlin
  - interview
---

# Android Development with Kotlin

## Kotlin Fundamentals

### Null Safety

```kotlin
// Nullable vs Non-nullable
var name: String = "John"      // Cannot be null
var nickname: String? = null   // Can be null

// Safe call operator
val length = nickname?.length  // Returns null if nickname is null

// Elvis operator
val len = nickname?.length ?: 0  // Default value if null

// Not-null assertion (use sparingly)
val len = nickname!!.length  // Throws NPE if null
```

### Data Classes

```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
)
// Auto-generates: equals(), hashCode(), toString(), copy()
```

### Coroutines

```kotlin
// Launch coroutine
viewModelScope.launch {
    val result = withContext(Dispatchers.IO) {
        repository.fetchData()
    }
    updateUI(result)
}

// Dispatchers
// Dispatchers.Main - UI operations
// Dispatchers.IO - Network/disk operations
// Dispatchers.Default - CPU-intensive work
```

### Extension Functions

```kotlin
fun String.addExclamation(): String = "$this!"
val greeting = "Hello".addExclamation() // "Hello!"
```

### Sealed Classes

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
    object Loading : Result<Nothing>()
}
```

## Android Components

### Activity Lifecycle

```
┌─────────────┐
│  onCreate() │ ─── Activity created
└──────┬──────┘
       ▼
┌─────────────┐
│  onStart()  │ ─── Visible to user
└──────┬──────┘
       ▼
┌─────────────┐
│ onResume()  │ ─── Interactive (foreground)
└──────┬──────┘
       │ User navigates away
       ▼
┌─────────────┐
│  onPause()  │ ─── Partially visible
└──────┬──────┘
       ▼
┌─────────────┐
│  onStop()   │ ─── Not visible
└──────┬──────┘
       │ App killed or finish()
       ▼
┌─────────────┐
│ onDestroy() │ ─── Activity destroyed
└─────────────┘
```

### Fragment Lifecycle

Additional callbacks:
- `onAttach()` - Fragment attached to activity
- `onCreateView()` - Create fragment UI
- `onViewCreated()` - View is ready
- `onDestroyView()` - View destroyed
- `onDetach()` - Fragment detached

### ViewModel

```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    fun loadUsers() {
        viewModelScope.launch {
            _users.value = repository.getUsers()
        }
    }
}
```

### LiveData vs StateFlow

| LiveData | StateFlow |
|----------|-----------|
| Lifecycle-aware | Requires manual lifecycle handling |
| Android-specific | Kotlin standard |
| No initial value required | Requires initial value |
| Single observer pattern | Hot flow, multiple collectors |

```kotlin
// StateFlow
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState.asStateFlow()
```

## Jetpack Components

### Navigation Component

```kotlin
// Navigate with arguments
findNavController().navigate(
    R.id.action_home_to_detail,
    bundleOf("userId" to userId)
)

// Safe Args (recommended)
val action = HomeFragmentDirections.actionHomeToDetail(userId)
findNavController().navigate(action)
```

### Room Database

```kotlin
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "user_name") val name: String
)

@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAll(): Flow<List<UserEntity>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)
    
    @Delete
    suspend fun delete(user: UserEntity)
}

@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

### WorkManager

```kotlin
class SyncWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            syncData()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}

// Schedule work
val request = OneTimeWorkRequestBuilder<SyncWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(request)
```

## Dependency Injection

### Hilt

```kotlin
@HiltAndroidApp
class MyApplication : Application()

@AndroidEntryPoint
class MainActivity : AppCompatActivity()

@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
}
```

## Debugging Tools

- **Logcat** - View logs with filters
- **ADB** - Android Debug Bridge commands
- **Android Profiler** - CPU, Memory, Network analysis
- **Layout Inspector** - Debug UI hierarchy
- **Database Inspector** - View Room database

```bash
# Common ADB commands
adb devices                    # List connected devices
adb logcat                     # View logs
adb install app.apk            # Install APK
adb shell pm clear com.app     # Clear app data
```

## Questions & Answers

> [!question]- Q1: What is the difference between `val` and `var` in Kotlin?
> **Answer:** 
> `val` declares an immutable (read-only) reference - once assigned, it cannot be reassigned.
> `var` declares a mutable reference that can be reassigned.
> 
> Note: `val` doesn't mean the object is immutable, just the reference.

> [!question]- Q2: Explain the Android Activity lifecycle.
> **Answer:** 
> Activities go through states: Created → Started → Resumed → Paused → Stopped → Destroyed.
> 
> Key callbacks:
> - `onCreate()` - Initialize activity, set content view
> - `onStart()` - Activity becomes visible
> - `onResume()` - Activity is interactive
> - `onPause()` - Another activity coming to foreground
> - `onStop()` - Activity no longer visible
> - `onDestroy()` - Activity being destroyed

> [!question]- Q3: What are Kotlin Coroutines and why use them?
> **Answer:** 
> Coroutines are Kotlin's solution for asynchronous programming. They allow writing async code sequentially without callbacks.
> 
> Benefits:
> - Lightweight (thousands can run on single thread)
> - Built-in cancellation support
> - Structured concurrency
> - Easy to switch between threads using Dispatchers

> [!question]- Q4: What is the purpose of ViewModel in Android?
> **Answer:** 
> ViewModel stores and manages UI-related data in a lifecycle-conscious way. It survives configuration changes (like screen rotation).
> 
> Key points:
> - Separates UI data from UI controllers
> - Data survives configuration changes
> - Should not hold references to Views or Context

> [!question]- Q5: Explain Room Database components.
> **Answer:** 
> Room has three main components:
> - **Entity** - Represents a table in the database
> - **DAO** - Contains methods to access the database
> - **Database** - Abstract class that holds the database and serves as main access point
> 
> Room provides compile-time verification of SQL queries.

> [!question]- Q6: What is Hilt and why use it?
> **Answer:** 
> Hilt is Android's recommended dependency injection library built on top of Dagger.
> 
> Benefits:
> - Reduces boilerplate compared to Dagger
> - Provides standard components for Android classes
> - Compile-time dependency graph validation
> - Integration with Jetpack libraries

> [!question]- Q7: Difference between LiveData and StateFlow?
> **Answer:** 
> | LiveData | StateFlow |
> |----------|-----------|
> | Android-specific | Kotlin standard library |
> | Lifecycle-aware automatically | Needs manual lifecycle handling |
> | No initial value required | Requires initial value |
> | Emits only to active observers | Hot flow, always active |

> [!question]- Q8: What is the Elvis operator in Kotlin?
> **Answer:** 
> The Elvis operator `?:` returns the left operand if it's not null, otherwise returns the right operand.
> 
> ```kotlin
> val name = nullableName ?: "Default"
> ```
> 
> Useful for providing default values for nullable expressions.

> [!question]- Q9: How do you handle configuration changes in Android?
> **Answer:** 
> Options:
> 1. **ViewModel** - Store data that survives config changes
> 2. **onSaveInstanceState()** - Save small amounts of data
> 3. **android:configChanges** - Handle changes manually (not recommended)
> 
> ViewModel is the preferred approach for UI data.

> [!question]- Q10: What is WorkManager used for?
> **Answer:** 
> WorkManager handles deferrable, guaranteed background work that needs to run even if the app exits.
> 
> Use cases:
> - Syncing data with server
> - Uploading logs
> - Processing images
> 
> Features: Constraints, chaining, periodic work, guaranteed execution.
