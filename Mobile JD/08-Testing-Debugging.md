---
title: Testing & Debugging
date: 2026-01-31
tags:
  - testing
  - debugging
  - unit-test
  - interview
---

# Testing & Debugging

## Testing Pyramid

```
          ┌─────────┐
          │   E2E   │  ← Few, slow, expensive
          ├─────────┤
          │Integration│  ← Some
          ├───────────┤
          │  Unit     │  ← Many, fast, cheap
          └───────────┘
```

## Unit Testing

### Android (JUnit + Mockito)

```kotlin
class UserViewModelTest {
    
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()
    
    private lateinit var viewModel: UserViewModel
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        repository = mock()
        viewModel = UserViewModel(repository)
    }
    
    @Test
    fun `loadUsers updates state with users`() = runTest {
        // Given
        val users = listOf(User(1, "John"))
        whenever(repository.getUsers()).thenReturn(users)
        
        // When
        viewModel.loadUsers()
        
        // Then
        assertEquals(users, viewModel.uiState.value.users)
        assertFalse(viewModel.uiState.value.isLoading)
    }
    
    @Test
    fun `loadUsers shows error on failure`() = runTest {
        // Given
        whenever(repository.getUsers()).thenThrow(RuntimeException("Network error"))
        
        // When
        viewModel.loadUsers()
        
        // Then
        assertNotNull(viewModel.uiState.value.error)
    }
}
```

### iOS (XCTest)

```swift
class UserViewModelTests: XCTestCase {
    
    var viewModel: UserViewModel!
    var mockRepository: MockUserRepository!
    
    override func setUp() {
        super.setUp()
        mockRepository = MockUserRepository()
        viewModel = UserViewModel(repository: mockRepository)
    }
    
    func testLoadUsersSuccess() async {
        // Given
        let expectedUsers = [User(id: 1, name: "John")]
        mockRepository.usersToReturn = expectedUsers
        
        // When
        await viewModel.loadUsers()
        
        // Then
        XCTAssertEqual(viewModel.users, expectedUsers)
        XCTAssertFalse(viewModel.isLoading)
    }
    
    func testLoadUsersFailure() async {
        // Given
        mockRepository.errorToThrow = NetworkError.serverError
        
        // When
        await viewModel.loadUsers()
        
        // Then
        XCTAssertNotNil(viewModel.error)
    }
}
```

## Integration Testing

### Android (Room Database)

```kotlin
@RunWith(AndroidJUnit4::class)
class UserDaoTest {
    
    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao
    
    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase::class.java
        ).build()
        userDao = database.userDao()
    }
    
    @After
    fun teardown() {
        database.close()
    }
    
    @Test
    fun insertAndRetrieveUser() = runTest {
        val user = UserEntity(id = 1, name = "John", email = "john@test.com")
        
        userDao.insert(user)
        val retrieved = userDao.getById(1)
        
        assertEquals(user, retrieved)
    }
}
```

## UI Testing

### Android (Espresso)

```kotlin
@RunWith(AndroidJUnit4::class)
class LoginActivityTest {
    
    @get:Rule
    val activityRule = ActivityScenarioRule(LoginActivity::class.java)
    
    @Test
    fun loginWithValidCredentials() {
        // Enter email
        onView(withId(R.id.emailInput))
            .perform(typeText("test@example.com"), closeSoftKeyboard())
        
        // Enter password
        onView(withId(R.id.passwordInput))
            .perform(typeText("password123"), closeSoftKeyboard())
        
        // Click login button
        onView(withId(R.id.loginButton))
            .perform(click())
        
        // Verify navigation to home
        onView(withId(R.id.homeScreen))
            .check(matches(isDisplayed()))
    }
    
    @Test
    fun showsErrorForEmptyEmail() {
        onView(withId(R.id.loginButton))
            .perform(click())
        
        onView(withId(R.id.emailError))
            .check(matches(withText("Email is required")))
    }
}
```

### iOS (XCUITest)

```swift
class LoginUITests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }
    
    func testLoginWithValidCredentials() {
        let emailField = app.textFields["emailTextField"]
        let passwordField = app.secureTextFields["passwordTextField"]
        let loginButton = app.buttons["loginButton"]
        
        emailField.tap()
        emailField.typeText("test@example.com")
        
        passwordField.tap()
        passwordField.typeText("password123")
        
        loginButton.tap()
        
        XCTAssertTrue(app.otherElements["homeScreen"].waitForExistence(timeout: 5))
    }
}
```

## Debugging Tools

### Android

| Tool | Purpose |
|------|---------|
| Logcat | View app logs |
| ADB | Command-line debugging |
| Android Profiler | CPU, Memory, Network |
| Layout Inspector | View hierarchy |
| Database Inspector | View Room data |

```kotlin
// Logging
Log.d("TAG", "Debug message")
Log.e("TAG", "Error message", exception)

// Timber (recommended)
Timber.d("Debug message")
Timber.e(exception, "Error message")
```

### iOS

| Tool | Purpose |
|------|---------|
| Xcode Console | View logs |
| Instruments | Performance profiling |
| Memory Graph | Find retain cycles |
| View Debugger | 3D view hierarchy |
| Network Link Conditioner | Simulate network |

```swift
// Logging
print("Debug message")
debugPrint(object)

// os_log (recommended)
import os
let logger = Logger(subsystem: "com.app", category: "network")
logger.debug("Request started")
logger.error("Request failed: \(error)")
```

## Performance Testing

### Android

```kotlin
// Benchmark
@RunWith(AndroidJUnit4::class)
class MyBenchmark {
    @get:Rule
    val benchmarkRule = BenchmarkRule()
    
    @Test
    fun benchmarkSomeWork() {
        benchmarkRule.measureRepeated {
            // Code to benchmark
        }
    }
}
```

### Memory Leak Detection

```kotlin
// LeakCanary (Android)
// Just add dependency - automatic detection
debugImplementation("com.squareup.leakcanary:leakcanary-android:2.12")
```

## Test Doubles

| Type | Description |
|------|-------------|
| **Stub** | Returns predefined values |
| **Mock** | Verifies interactions |
| **Fake** | Working implementation (simplified) |
| **Spy** | Wraps real object, tracks calls |

```kotlin
// Fake Repository
class FakeUserRepository : UserRepository {
    var users = mutableListOf<User>()
    var shouldFail = false
    
    override suspend fun getUsers(): List<User> {
        if (shouldFail) throw Exception("Failed")
        return users
    }
}
```

## Questions & Answers

> [!question]- Q1: What is the testing pyramid?
> **Answer:** 
> A strategy for test distribution:
> - **Unit tests** (base): Many, fast, test individual components
> - **Integration tests** (middle): Some, test component interactions
> - **E2E/UI tests** (top): Few, slow, test full user flows
> 
> More unit tests, fewer E2E tests.

> [!question]- Q2: What is the difference between mock and stub?
> **Answer:** 
> - **Stub**: Provides canned responses, doesn't verify behavior
> - **Mock**: Verifies that certain methods were called with expected arguments
> 
> Use stubs for state verification, mocks for behavior verification.

> [!question]- Q3: How do you test coroutines in Android?
> **Answer:** 
> Use `kotlinx-coroutines-test`:
> ```kotlin
> @Test
> fun test() = runTest {
>     // Test code with coroutines
> }
> ```
> Use `MainDispatcherRule` to replace Main dispatcher in tests.

> [!question]- Q4: What is Espresso used for?
> **Answer:** 
> Espresso is Android's UI testing framework. It:
> - Finds views using matchers
> - Performs actions (click, type)
> - Verifies view states
> - Automatically waits for UI to be idle

> [!question]- Q5: How do you detect memory leaks?
> **Answer:** 
> - **Android**: LeakCanary (automatic detection)
> - **iOS**: Instruments (Leaks tool), Memory Graph Debugger
> 
> Common causes: Static references, unregistered listeners, retain cycles in closures.

> [!question]- Q6: What is XCTest?
> **Answer:** 
> Apple's testing framework for iOS/macOS. Includes:
> - `XCTestCase` for unit tests
> - `XCUITest` for UI tests
> - Assertions like `XCTAssertEqual`, `XCTAssertTrue`
> - Performance testing with `measure {}`

> [!question]- Q7: How do you test Room database?
> **Answer:** 
> Use in-memory database:
> ```kotlin
> Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java).build()
> ```
> Benefits: Fast, isolated, no cleanup needed.

> [!question]- Q8: What is code coverage and why does it matter?
> **Answer:** 
> Code coverage measures percentage of code executed by tests.
> 
> Types: Line, branch, function coverage
> 
> Important but not sufficient - high coverage doesn't guarantee quality tests.

> [!question]- Q9: How do you debug network issues in mobile apps?
> **Answer:** 
> Tools:
> - **Android**: OkHttp Logging Interceptor, Charles Proxy
> - **iOS**: Network Link Conditioner, Charles Proxy
> 
> Check: Request/response bodies, headers, status codes, timing.

> [!question]- Q10: What is TDD (Test-Driven Development)?
> **Answer:** 
> Development approach:
> 1. Write failing test first
> 2. Write minimal code to pass
> 3. Refactor
> 
> Benefits: Better design, high coverage, documentation through tests.
