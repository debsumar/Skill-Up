---
title: Live Coding Scenarios
date: 2026-01-31
tags:
  - coding
  - interview
  - f2f
  - screen-sharing
---

# Live Coding Scenarios

> [!important] F2F Screen-Sharing Prep
> Common coding tasks asked during mobile developer interviews.

## API Call & Display Data

### Task: Fetch users from API and display in a list

**Android (Kotlin)**

```kotlin
// Data class
data class User(val id: Int, val name: String, val email: String)

// API interface
interface UserApi {
    @GET("users")
    suspend fun getUsers(): List<User>
}

// ViewModel
class UserViewModel(private val api: UserApi) : ViewModel() {
    
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                _users.value = api.getUsers()
            } catch (e: Exception) {
                // Handle error
            } finally {
                _isLoading.value = false
            }
        }
    }
}

// Composable
@Composable
fun UserList(viewModel: UserViewModel) {
    val users by viewModel.users.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()
    
    LaunchedEffect(Unit) { viewModel.loadUsers() }
    
    if (isLoading) {
        CircularProgressIndicator()
    } else {
        LazyColumn {
            items(users) { user ->
                Text("${user.name} - ${user.email}")
            }
        }
    }
}
```

**iOS (Swift)**

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

class UserViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    
    func loadUsers() async {
        isLoading = true
        defer { isLoading = false }
        
        guard let url = URL(string: "https://api.example.com/users") else { return }
        
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            users = try JSONDecoder().decode([User].self, from: data)
        } catch {
            print("Error: \(error)")
        }
    }
}

struct UserListView: View {
    @StateObject var viewModel = UserViewModel()
    
    var body: some View {
        Group {
            if viewModel.isLoading {
                ProgressView()
            } else {
                List(viewModel.users, id: \.id) { user in
                    Text("\(user.name) - \(user.email)")
                }
            }
        }
        .task { await viewModel.loadUsers() }
    }
}
```

## Login Form with Validation

### Task: Create login form with email/password validation

**Android (Kotlin)**

```kotlin
@Composable
fun LoginScreen(onLogin: (String, String) -> Unit) {
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    var emailError by remember { mutableStateOf<String?>(null) }
    var passwordError by remember { mutableStateOf<String?>(null) }
    
    fun validate(): Boolean {
        emailError = when {
            email.isBlank() -> "Email required"
            !email.contains("@") -> "Invalid email"
            else -> null
        }
        passwordError = when {
            password.isBlank() -> "Password required"
            password.length < 6 -> "Min 6 characters"
            else -> null
        }
        return emailError == null && passwordError == null
    }
    
    Column(modifier = Modifier.padding(16.dp)) {
        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            isError = emailError != null,
            supportingText = { emailError?.let { Text(it) } }
        )
        
        Spacer(modifier = Modifier.height(8.dp))
        
        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Password") },
            visualTransformation = PasswordVisualTransformation(),
            isError = passwordError != null,
            supportingText = { passwordError?.let { Text(it) } }
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Button(onClick = { if (validate()) onLogin(email, password) }) {
            Text("Login")
        }
    }
}
```

**iOS (Swift)**

```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var emailError: String?
    @State private var passwordError: String?
    
    var onLogin: (String, String) -> Void
    
    func validate() -> Bool {
        emailError = email.isEmpty ? "Email required" :
                     !email.contains("@") ? "Invalid email" : nil
        passwordError = password.isEmpty ? "Password required" :
                        password.count < 6 ? "Min 6 characters" : nil
        return emailError == nil && passwordError == nil
    }
    
    var body: some View {
        VStack(spacing: 16) {
            TextField("Email", text: $email)
                .textFieldStyle(.roundedBorder)
            if let error = emailError {
                Text(error).foregroundColor(.red).font(.caption)
            }
            
            SecureField("Password", text: $password)
                .textFieldStyle(.roundedBorder)
            if let error = passwordError {
                Text(error).foregroundColor(.red).font(.caption)
            }
            
            Button("Login") {
                if validate() { onLogin(email, password) }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

## Search with Debounce

### Task: Implement search that waits for user to stop typing

**Android (Kotlin)**

```kotlin
class SearchViewModel : ViewModel() {
    private val _query = MutableStateFlow("")
    val query: StateFlow<String> = _query.asStateFlow()
    
    private val _results = MutableStateFlow<List<String>>(emptyList())
    val results: StateFlow<List<String>> = _results.asStateFlow()
    
    init {
        viewModelScope.launch {
            _query
                .debounce(300) // Wait 300ms after last keystroke
                .filter { it.length >= 2 }
                .distinctUntilChanged()
                .collect { query ->
                    _results.value = search(query)
                }
        }
    }
    
    fun onQueryChange(query: String) {
        _query.value = query
    }
    
    private suspend fun search(query: String): List<String> {
        // API call or local search
        delay(100) // Simulate network
        return listOf("Result 1", "Result 2", "Result 3")
    }
}
```

## RecyclerView/List with Click Handler

### Task: Display list with item click navigation

**Android (Kotlin)**

```kotlin
@Composable
fun ProductList(
    products: List<Product>,
    onProductClick: (Product) -> Unit
) {
    LazyColumn {
        items(products, key = { it.id }) { product ->
            ProductItem(
                product = product,
                onClick = { onProductClick(product) }
            )
        }
    }
}

@Composable
fun ProductItem(product: Product, onClick: () -> Unit) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .clickable(onClick = onClick)
    ) {
        Row(modifier = Modifier.padding(16.dp)) {
            AsyncImage(
                model = product.imageUrl,
                contentDescription = null,
                modifier = Modifier.size(60.dp)
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column {
                Text(product.name, fontWeight = FontWeight.Bold)
                Text("$${product.price}")
            }
        }
    }
}
```

## CRUD Operations with Room

### Task: Implement add/edit/delete for notes

**Android (Kotlin)**

```kotlin
@Entity
data class Note(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val content: String,
    val createdAt: Long = System.currentTimeMillis()
)

@Dao
interface NoteDao {
    @Query("SELECT * FROM note ORDER BY createdAt DESC")
    fun getAll(): Flow<List<Note>>
    
    @Insert
    suspend fun insert(note: Note)
    
    @Update
    suspend fun update(note: Note)
    
    @Delete
    suspend fun delete(note: Note)
}

class NoteViewModel(private val dao: NoteDao) : ViewModel() {
    val notes = dao.getAll().stateIn(
        viewModelScope,
        SharingStarted.WhileSubscribed(5000),
        emptyList()
    )
    
    fun addNote(title: String, content: String) {
        viewModelScope.launch {
            dao.insert(Note(title = title, content = content))
        }
    }
    
    fun updateNote(note: Note) {
        viewModelScope.launch { dao.update(note) }
    }
    
    fun deleteNote(note: Note) {
        viewModelScope.launch { dao.delete(note) }
    }
}
```

## Common Algorithm Questions

### Reverse a String

```kotlin
fun reverseString(s: String): String = s.reversed()

// Without built-in
fun reverseString(s: String): String {
    val chars = s.toCharArray()
    var left = 0
    var right = chars.size - 1
    while (left < right) {
        val temp = chars[left]
        chars[left] = chars[right]
        chars[right] = temp
        left++
        right--
    }
    return String(chars)
}
```

### Find Duplicates in Array

```kotlin
fun findDuplicates(nums: List<Int>): List<Int> {
    return nums.groupBy { it }
        .filter { it.value.size > 1 }
        .keys.toList()
}
```

### Check Palindrome

```kotlin
fun isPalindrome(s: String): Boolean {
    val cleaned = s.lowercase().filter { it.isLetterOrDigit() }
    return cleaned == cleaned.reversed()
}
```

### FizzBuzz

```kotlin
fun fizzBuzz(n: Int): List<String> {
    return (1..n).map { i ->
        when {
            i % 15 == 0 -> "FizzBuzz"
            i % 3 == 0 -> "Fizz"
            i % 5 == 0 -> "Buzz"
            else -> i.toString()
        }
    }
}
```

## Tips for Live Coding

> [!tip] During Screen Share
> 1. **Think aloud** - Explain your approach before coding
> 2. **Start simple** - Get basic version working first
> 3. **Ask clarifying questions** - Don't assume
> 4. **Handle edge cases** - Null, empty, errors
> 5. **Test your code** - Walk through with example
> 6. **Refactor if time** - Clean up after it works

> [!warning] Common Mistakes
> - Jumping into code without planning
> - Not handling null/empty cases
> - Forgetting error handling
> - Over-engineering the solution
> - Not testing with examples

## Questions & Answers

> [!question]- Q1: How do you handle configuration changes in Android?
> **Answer:**
> - Use ViewModel to store UI state (survives rotation)
> - Use `rememberSaveable` in Compose for primitive state
> - For complex objects, use `onSaveInstanceState`
> 
> ViewModel is the primary solution.

> [!question]- Q2: How do you prevent memory leaks in Android?
> **Answer:**
> - Don't hold Activity/Context references in long-lived objects
> - Use `viewModelScope` for coroutines
> - Unregister listeners in `onDestroy`
> - Use weak references when necessary
> - Use LeakCanary to detect leaks

> [!question]- Q3: How do you implement pull-to-refresh?
> **Answer:**
> ```kotlin
> // Compose
> val refreshState = rememberPullRefreshState(
>     refreshing = isRefreshing,
>     onRefresh = { viewModel.refresh() }
> )
> Box(Modifier.pullRefresh(refreshState)) {
>     LazyColumn { ... }
>     PullRefreshIndicator(isRefreshing, refreshState)
> }
> ```

> [!question]- Q4: How do you handle back press in Compose?
> **Answer:**
> ```kotlin
> BackHandler(enabled = true) {
>     // Handle back press
>     if (showDialog) {
>         showDialog = false
>     } else {
>         navController.popBackStack()
>     }
> }
> ```

> [!question]- Q5: How do you pass data between screens?
> **Answer:**
> Options:
> - Navigation arguments (simple data)
> - Shared ViewModel (complex data)
> - SavedStateHandle
> - Global state (Redux-like)
> 
> For simple data, use navigation arguments.

> [!question]- Q6: How do you implement swipe-to-delete?
> **Answer:**
> ```kotlin
> // Compose
> SwipeToDismiss(
>     state = dismissState,
>     background = { DeleteBackground() },
>     dismissContent = { ItemContent() }
> )
> 
> LaunchedEffect(dismissState.currentValue) {
>     if (dismissState.currentValue == DismissValue.DismissedToStart) {
>         viewModel.deleteItem(item)
>     }
> }
> ```

> [!question]- Q7: How do you show a bottom sheet?
> **Answer:**
> ```kotlin
> // Compose Material 3
> val sheetState = rememberModalBottomSheetState()
> 
> if (showSheet) {
>     ModalBottomSheet(
>         onDismissRequest = { showSheet = false },
>         sheetState = sheetState
>     ) {
>         // Sheet content
>     }
> }
> ```

> [!question]- Q8: How do you implement dark mode?
> **Answer:**
> ```kotlin
> // Follow system
> val darkTheme = isSystemInDarkTheme()
> 
> MaterialTheme(
>     colorScheme = if (darkTheme) darkColorScheme() else lightColorScheme()
> ) {
>     // App content
> }
> ```
> Store user preference in DataStore.

> [!question]- Q9: How do you handle keyboard visibility?
> **Answer:**
> ```kotlin
> // Compose
> val keyboardController = LocalSoftwareKeyboardController.current
> 
> // Hide keyboard
> keyboardController?.hide()
> 
> // Adjust for keyboard
> Modifier.imePadding()
> ```

> [!question]- Q10: How do you implement biometric authentication?
> **Answer:**
> ```kotlin
> val biometricPrompt = BiometricPrompt(activity, executor, callback)
> val promptInfo = BiometricPrompt.PromptInfo.Builder()
>     .setTitle("Login")
>     .setNegativeButtonText("Cancel")
>     .build()
> 
> biometricPrompt.authenticate(promptInfo)
> ```
