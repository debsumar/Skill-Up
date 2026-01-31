---
title: Local Storage
date: 2026-01-31
tags:
  - room
  - coredata
  - sqlite
  - storage
  - interview
---

# Local Storage

## Android - Room Database

### Components

```
┌─────────────────────────────────────────┐
│              Room Database              │
├─────────────────────────────────────────┤
│  Entity  │    DAO    │    Database      │
│  (Table) │  (Queries)│  (Access Point)  │
└─────────────────────────────────────────┘
```

### Entity

```kotlin
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    @ColumnInfo(name = "user_name") val name: String,
    val email: String,
    val createdAt: Long = System.currentTimeMillis()
)

// With foreign key
@Entity(
    tableName = "posts",
    foreignKeys = [ForeignKey(
        entity = UserEntity::class,
        parentColumns = ["id"],
        childColumns = ["user_id"],
        onDelete = ForeignKey.CASCADE
    )]
)
data class PostEntity(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "user_id") val userId: Int,
    val title: String
)
```

### DAO

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAll(): Flow<List<UserEntity>>
    
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getById(id: Int): UserEntity?
    
    @Query("SELECT * FROM users WHERE user_name LIKE :search")
    fun search(search: String): Flow<List<UserEntity>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(users: List<UserEntity>)
    
    @Update
    suspend fun update(user: UserEntity)
    
    @Delete
    suspend fun delete(user: UserEntity)
    
    @Query("DELETE FROM users")
    suspend fun deleteAll()
}
```

### Database

```kotlin
@Database(
    entities = [UserEntity::class, PostEntity::class],
    version = 1,
    exportSchema = true
)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    abstract fun postDao(): PostDao
}

// Type Converters
class Converters {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? = value?.let { Date(it) }
    
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? = date?.time
}
```

### Migration

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN age INTEGER DEFAULT 0")
    }
}

Room.databaseBuilder(context, AppDatabase::class.java, "app.db")
    .addMigrations(MIGRATION_1_2)
    .build()
```

## iOS - CoreData

### Stack Setup

```swift
class CoreDataStack {
    static let shared = CoreDataStack()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Model")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("CoreData error: \(error)")
            }
        }
        return container
    }()
    
    var context: NSManagedObjectContext {
        persistentContainer.viewContext
    }
    
    func saveContext() {
        if context.hasChanges {
            try? context.save()
        }
    }
}
```

### CRUD Operations

```swift
// Create
func createUser(name: String, email: String) {
    let user = User(context: context)
    user.name = name
    user.email = email
    user.createdAt = Date()
    saveContext()
}

// Read
func fetchUsers() -> [User] {
    let request: NSFetchRequest<User> = User.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
    return (try? context.fetch(request)) ?? []
}

// Read with predicate
func fetchUser(byEmail email: String) -> User? {
    let request: NSFetchRequest<User> = User.fetchRequest()
    request.predicate = NSPredicate(format: "email == %@", email)
    return try? context.fetch(request).first
}

// Update
func updateUser(_ user: User, name: String) {
    user.name = name
    saveContext()
}

// Delete
func deleteUser(_ user: User) {
    context.delete(user)
    saveContext()
}
```

### Fetch Results Controller

```swift
lazy var fetchedResultsController: NSFetchedResultsController<User> = {
    let request: NSFetchRequest<User> = User.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
    
    let controller = NSFetchedResultsController(
        fetchRequest: request,
        managedObjectContext: context,
        sectionNameKeyPath: nil,
        cacheName: nil
    )
    controller.delegate = self
    try? controller.performFetch()
    return controller
}()
```

## iOS - SQLite (Direct)

```swift
import SQLite3

class SQLiteManager {
    var db: OpaquePointer?
    
    func openDatabase() {
        let path = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
            .appendingPathComponent("app.sqlite").path
        
        if sqlite3_open(path, &db) != SQLITE_OK {
            print("Error opening database")
        }
    }
    
    func createTable() {
        let query = """
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE
            )
        """
        sqlite3_exec(db, query, nil, nil, nil)
    }
}
```

## Shared Preferences / UserDefaults

### Android - SharedPreferences

```kotlin
// Write
val prefs = context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
prefs.edit()
    .putString("user_token", token)
    .putBoolean("is_logged_in", true)
    .apply()

// Read
val token = prefs.getString("user_token", null)
val isLoggedIn = prefs.getBoolean("is_logged_in", false)

// DataStore (recommended)
val dataStore = context.createDataStore(name = "settings")
val TOKEN_KEY = stringPreferencesKey("user_token")

// Write
dataStore.edit { it[TOKEN_KEY] = token }

// Read
val tokenFlow: Flow<String?> = dataStore.data.map { it[TOKEN_KEY] }
```

### iOS - UserDefaults

```swift
// Write
UserDefaults.standard.set("token123", forKey: "userToken")
UserDefaults.standard.set(true, forKey: "isLoggedIn")

// Read
let token = UserDefaults.standard.string(forKey: "userToken")
let isLoggedIn = UserDefaults.standard.bool(forKey: "isLoggedIn")

// Remove
UserDefaults.standard.removeObject(forKey: "userToken")
```

## Secure Storage

### Android - EncryptedSharedPreferences

```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val securePrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

securePrefs.edit().putString("api_key", apiKey).apply()
```

### iOS - Keychain

```swift
func saveToKeychain(key: String, value: String) {
    let data = value.data(using: .utf8)!
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data
    ]
    SecItemDelete(query as CFDictionary)
    SecItemAdd(query as CFDictionary, nil)
}

func readFromKeychain(key: String) -> String? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecReturnData as String: true
    ]
    var result: AnyObject?
    SecItemCopyMatching(query as CFDictionary, &result)
    guard let data = result as? Data else { return nil }
    return String(data: data, encoding: .utf8)
}
```

## Questions & Answers

> [!question]- Q1: What are the main components of Room?
> **Answer:** 
> - **Entity**: Represents a table (annotated data class)
> - **DAO**: Data Access Object with query methods
> - **Database**: Abstract class holding database instance
> 
> Room provides compile-time SQL verification.

> [!question]- Q2: How do you handle database migrations in Room?
> **Answer:** 
> Create Migration objects specifying version changes:
> ```kotlin
> val MIGRATION_1_2 = object : Migration(1, 2) {
>     override fun migrate(db: SupportSQLiteDatabase) {
>         db.execSQL("ALTER TABLE users ADD COLUMN age INTEGER")
>     }
> }
> ```
> Add migrations when building database.

> [!question]- Q3: What is the difference between CoreData and SQLite?
> **Answer:** 
> | CoreData | SQLite |
> |----------|--------|
> | Object graph management | Relational database |
> | Higher-level abstraction | Low-level SQL |
> | Automatic change tracking | Manual queries |
> | Built-in caching | Manual caching |
> 
> CoreData uses SQLite as one of its storage options.

> [!question]- Q4: When should you use SharedPreferences vs Room?
> **Answer:** 
> **SharedPreferences**: Simple key-value pairs (settings, tokens, flags)
> 
> **Room**: Structured data, relationships, complex queries
> 
> Rule: If you need to query or have relationships, use Room.

> [!question]- Q5: How do you store sensitive data securely?
> **Answer:** 
> - **Android**: EncryptedSharedPreferences, Android Keystore
> - **iOS**: Keychain Services
> 
> Never store sensitive data in plain SharedPreferences/UserDefaults.

> [!question]- Q6: What is NSFetchedResultsController?
> **Answer:** 
> A controller that efficiently manages results from CoreData fetch requests and provides updates to UI.
> 
> Benefits:
> - Automatic batching
> - Change notifications
> - Section support
> - Memory efficient

> [!question]- Q7: How does Room handle threading?
> **Answer:** 
> Room enforces main-thread safety:
> - Queries must run on background thread (or use suspend functions)
> - Returns Flow/LiveData for reactive updates
> - Use `allowMainThreadQueries()` only for debugging

> [!question]- Q8: What is DataStore and why use it over SharedPreferences?
> **Answer:** 
> DataStore is Jetpack's modern data storage solution.
> 
> Advantages:
> - Async API (Flow-based)
> - Type safety with Proto DataStore
> - No runtime exceptions
> - Transactional updates

> [!question]- Q9: How do you implement relationships in Room?
> **Answer:** 
> Use `@Relation` annotation:
> ```kotlin
> data class UserWithPosts(
>     @Embedded val user: User,
>     @Relation(parentColumn = "id", entityColumn = "userId")
>     val posts: List<Post>
> )
> ```
> Query with `@Transaction` for consistency.

> [!question]- Q10: What is the difference between `apply()` and `commit()` in SharedPreferences?
> **Answer:** 
> - **apply()**: Asynchronous, returns immediately, writes in background
> - **commit()**: Synchronous, blocks until write completes, returns boolean
> 
> Use `apply()` unless you need immediate confirmation.
