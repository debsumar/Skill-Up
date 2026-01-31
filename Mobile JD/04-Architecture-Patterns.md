---
title: Architecture Patterns
date: 2026-01-31
tags:
  - architecture
  - mvvm
  - mvp
  - clean-architecture
  - interview
---

# Architecture Patterns

## Why Architecture Matters

- **Testability** - Easy to write unit tests
- **Maintainability** - Easy to modify and extend
- **Scalability** - Handles growing codebase
- **Separation of Concerns** - Clear responsibilities

## MVC (Model-View-Controller)

```
┌─────────┐     Updates     ┌─────────┐
│  Model  │────────────────▶│  View   │
└────┬────┘                 └────┬────┘
     │                           │
     │      ┌──────────────┐     │
     └─────▶│  Controller  │◀────┘
            └──────────────┘
              User Actions
```

- **Model** - Data and business logic
- **View** - UI presentation
- **Controller** - Handles user input, updates model

> [!warning] Problem
> Controller becomes massive ("Massive View Controller" in iOS)

## MVP (Model-View-Presenter)

```
┌─────────┐                 ┌─────────┐
│  Model  │◀───────────────▶│Presenter│
└─────────┘                 └────┬────┘
                                 │
                            ┌────▼────┐
                            │  View   │
                            │(Passive)│
                            └─────────┘
```

- **Model** - Data and business logic
- **View** - Passive, only displays data
- **Presenter** - Mediates between Model and View

```kotlin
// View Interface
interface UserView {
    fun showUsers(users: List<User>)
    fun showError(message: String)
    fun showLoading()
}

// Presenter
class UserPresenter(
    private val view: UserView,
    private val repository: UserRepository
) {
    fun loadUsers() {
        view.showLoading()
        repository.getUsers(
            onSuccess = { view.showUsers(it) },
            onError = { view.showError(it.message) }
        )
    }
}
```

## MVVM (Model-View-ViewModel)

```
┌─────────┐                 ┌───────────┐
│  Model  │◀───────────────▶│ ViewModel │
└─────────┘                 └─────┬─────┘
                                  │
                            Data Binding
                                  │
                            ┌─────▼─────┐
                            │   View    │
                            └───────────┘
```

- **Model** - Data and business logic
- **View** - Observes ViewModel, displays data
- **ViewModel** - Exposes data streams, handles logic

```kotlin
// ViewModel
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            try {
                val users = repository.getUsers()
                _uiState.update { it.copy(users = users, isLoading = false) }
            } catch (e: Exception) {
                _uiState.update { it.copy(error = e.message, isLoading = false) }
            }
        }
    }
}

data class UserUiState(
    val users: List<User> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)
```

## Clean Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│              (Activities, Fragments, ViewModels)         │
├─────────────────────────────────────────────────────────┤
│                      Domain Layer                        │
│              (Use Cases, Entities, Repository            │
│                      Interfaces)                         │
├─────────────────────────────────────────────────────────┤
│                       Data Layer                         │
│         (Repository Impl, Data Sources, APIs)            │
└─────────────────────────────────────────────────────────┘

Dependencies flow INWARD (outer depends on inner)
```

### Layers

1. **Presentation** - UI, ViewModels
2. **Domain** - Business logic, Use Cases, Entities
3. **Data** - Repository implementations, API, Database

### Use Case Example

```kotlin
// Domain Layer - Use Case
class GetUsersUseCase(
    private val repository: UserRepository
) {
    suspend operator fun invoke(): Result<List<User>> {
        return repository.getUsers()
    }
}

// Domain Layer - Repository Interface
interface UserRepository {
    suspend fun getUsers(): Result<List<User>>
}

// Data Layer - Repository Implementation
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao
) : UserRepository {
    
    override suspend fun getUsers(): Result<List<User>> {
        return try {
            val users = api.getUsers()
            dao.insertAll(users)
            Result.success(users)
        } catch (e: Exception) {
            val cached = dao.getAll()
            if (cached.isNotEmpty()) Result.success(cached)
            else Result.failure(e)
        }
    }
}
```

## MVI (Model-View-Intent)

```
┌─────────┐  Intent   ┌───────────┐  State   ┌─────────┐
│  View   │──────────▶│  ViewModel │─────────▶│  View   │
└─────────┘           └───────────┘          └─────────┘
     ▲                                            │
     └────────────────────────────────────────────┘
                    Unidirectional
```

- **Intent** - User actions
- **Model** - Immutable state
- **View** - Renders state

```kotlin
// State
data class UserState(
    val users: List<User> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

// Intent
sealed class UserIntent {
    object LoadUsers : UserIntent()
    data class DeleteUser(val id: Int) : UserIntent()
}

// ViewModel
class UserViewModel : ViewModel() {
    private val _state = MutableStateFlow(UserState())
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    fun processIntent(intent: UserIntent) {
        when (intent) {
            is UserIntent.LoadUsers -> loadUsers()
            is UserIntent.DeleteUser -> deleteUser(intent.id)
        }
    }
}
```

## Comparison

| Pattern | Testability | Complexity | Best For |
|---------|-------------|------------|----------|
| MVC | Low | Low | Simple apps |
| MVP | High | Medium | Legacy projects |
| MVVM | High | Medium | Modern Android/iOS |
| Clean | Very High | High | Large, complex apps |
| MVI | Very High | High | Complex state management |

## Questions & Answers

> [!question]- Q1: What is the main difference between MVP and MVVM?
> **Answer:** 
> - **MVP**: Presenter holds reference to View interface, directly calls view methods
> - **MVVM**: ViewModel doesn't know about View, exposes data streams that View observes
> 
> MVVM is more loosely coupled and easier to test.

> [!question]- Q2: Explain the Dependency Rule in Clean Architecture.
> **Answer:** 
> Dependencies must point inward. Outer layers can depend on inner layers, but inner layers cannot depend on outer layers.
> 
> - Presentation → Domain ✓
> - Domain → Data ✗
> - Data implements Domain interfaces

> [!question]- Q3: What is a Use Case in Clean Architecture?
> **Answer:** 
> A Use Case (or Interactor) encapsulates a single business action. It contains application-specific business rules.
> 
> Benefits:
> - Single responsibility
> - Easy to test
> - Reusable across different UI
> - Documents what the app does

> [!question]- Q4: Why use MVVM over MVC?
> **Answer:** 
> MVVM advantages:
> - Better separation of concerns
> - ViewModel is easier to unit test (no Android dependencies)
> - Data binding reduces boilerplate
> - Survives configuration changes
> - Avoids "Massive View Controller" problem

> [!question]- Q5: What is unidirectional data flow?
> **Answer:** 
> Data flows in one direction: State → View → Intent → State
> 
> Benefits:
> - Predictable state changes
> - Easier debugging
> - Time-travel debugging possible
> - Clear data flow

> [!question]- Q6: How do you handle navigation in MVVM?
> **Answer:** 
> Options:
> 1. **Events/Commands** - ViewModel emits navigation events
> 2. **Navigator interface** - Inject navigator into ViewModel
> 3. **Shared ViewModel** - For fragment communication
> 4. **Navigation Component** - Let View handle navigation based on state

> [!question]- Q7: What is the Repository pattern?
> **Answer:** 
> Repository abstracts data sources from the rest of the app. It decides whether to fetch from network, cache, or database.
> 
> Benefits:
> - Single source of truth
> - Decouples data sources from business logic
> - Easy to swap implementations
> - Simplifies testing

> [!question]- Q8: How do you test a ViewModel?
> **Answer:** 
> ```kotlin
> @Test
> fun `loadUsers updates state with users`() = runTest {
>     val repository = FakeUserRepository(listOf(testUser))
>     val viewModel = UserViewModel(repository)
>     
>     viewModel.loadUsers()
>     
>     assertEquals(listOf(testUser), viewModel.uiState.value.users)
> }
> ```
> 
> Use fake repositories and test coroutines with `runTest`.

> [!question]- Q9: What is the difference between Entity and Model?
> **Answer:** 
> - **Entity** (Domain): Core business object, no framework dependencies
> - **Model** (Data): Data transfer object, may have serialization annotations
> 
> Map between them at layer boundaries.

> [!question]- Q10: When would you NOT use Clean Architecture?
> **Answer:** 
> Skip Clean Architecture for:
> - Small, simple apps
> - Prototypes or MVPs
> - Apps with short lifespan
> - Solo projects with tight deadlines
> 
> The overhead isn't worth it for simple apps.
