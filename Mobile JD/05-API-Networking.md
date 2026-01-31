---
title: API & Networking
date: 2026-01-31
tags:
  - api
  - rest
  - networking
  - interview
---

# API & Networking

## REST API Fundamentals

### HTTP Methods

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET | Retrieve data | Yes |
| POST | Create resource | No |
| PUT | Update/Replace | Yes |
| PATCH | Partial update | No |
| DELETE | Remove resource | Yes |

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

### REST Best Practices

```
GET    /users          → List users
GET    /users/123      → Get user 123
POST   /users          → Create user
PUT    /users/123      → Update user 123
DELETE /users/123      → Delete user 123
GET    /users/123/posts → Get posts by user 123
```

## Android Networking (Retrofit)

### Setup

```kotlin
// Retrofit instance
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .client(okHttpClient)
    .build()

// API interface
interface UserApi {
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Int): User
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") id: Int, @Body user: User): User
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int)
}
```

### OkHttp Interceptors

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    })
    .addInterceptor { chain ->
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        chain.proceed(request)
    }
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .build()
```

### Error Handling

```kotlin
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val code: Int, val message: String) : NetworkResult<Nothing>()
    object Loading : NetworkResult<Nothing>()
}

suspend fun <T> safeApiCall(apiCall: suspend () -> T): NetworkResult<T> {
    return try {
        NetworkResult.Success(apiCall())
    } catch (e: HttpException) {
        NetworkResult.Error(e.code(), e.message())
    } catch (e: IOException) {
        NetworkResult.Error(-1, "Network error")
    }
}
```

## iOS Networking (URLSession)

### Basic Request

```swift
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }
    
    return try JSONDecoder().decode([User].self, from: data)
}
```

### Network Layer

```swift
class NetworkManager {
    static let shared = NetworkManager()
    private let session = URLSession.shared
    
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var request = URLRequest(url: endpoint.url)
        request.httpMethod = endpoint.method.rawValue
        request.allHTTPHeaderFields = endpoint.headers
        
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            return try JSONDecoder().decode(T.self, from: data)
        case 401:
            throw NetworkError.unauthorized
        case 404:
            throw NetworkError.notFound
        default:
            throw NetworkError.serverError(httpResponse.statusCode)
        }
    }
}
```

## JSON Parsing

### Android (Gson/Moshi)

```kotlin
// Gson
data class User(
    @SerializedName("user_id") val id: Int,
    @SerializedName("user_name") val name: String,
    val email: String
)

// Moshi
@JsonClass(generateAdapter = true)
data class User(
    @Json(name = "user_id") val id: Int,
    @Json(name = "user_name") val name: String,
    val email: String
)
```

### iOS (Codable)

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
    
    enum CodingKeys: String, CodingKey {
        case id = "user_id"
        case name = "user_name"
        case email
    }
}

// Custom date decoding
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
```

## Authentication

### Token-Based Auth

```
┌────────┐  1. Login    ┌────────┐
│ Client │─────────────▶│ Server │
└────────┘              └───┬────┘
     ▲                      │
     │  2. JWT Token        │
     └──────────────────────┘

┌────────┐  3. Request + Token  ┌────────┐
│ Client │─────────────────────▶│ Server │
└────────┘                      └───┬────┘
     ▲                              │
     │  4. Protected Resource       │
     └──────────────────────────────┘
```

### JWT Structure

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### OAuth 2.0 Flow

```
┌────────┐  1. Auth Request   ┌────────────┐
│  App   │───────────────────▶│ Auth Server│
└────────┘                    └─────┬──────┘
     │                              │
     │  2. User Login (Browser)     │
     │◀─────────────────────────────┘
     │
     │  3. Auth Code
     ▼
┌────────┐  4. Exchange Code  ┌────────────┐
│  App   │───────────────────▶│ Auth Server│
└────────┘                    └─────┬──────┘
     │                              │
     │  5. Access + Refresh Token   │
     │◀─────────────────────────────┘
```

### Google Sign-In (Android)

```kotlin
// Setup
val gso = GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
    .requestIdToken(getString(R.string.web_client_id))
    .requestEmail()
    .build()

val googleSignInClient = GoogleSignIn.getClient(this, gso)

// Launch sign-in
val signInIntent = googleSignInClient.signInIntent
startActivityForResult(signInIntent, RC_SIGN_IN)

// Handle result
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == RC_SIGN_IN) {
        val task = GoogleSignIn.getSignedInAccountFromIntent(data)
        val account = task.getResult(ApiException::class.java)
        val idToken = account.idToken // Send to your server
    }
}
```

### Apple Sign-In (iOS)

```swift
import AuthenticationServices

class LoginViewController: UIViewController, ASAuthorizationControllerDelegate {
    
    func signInWithApple() {
        let request = ASAuthorizationAppleIDProvider().createRequest()
        request.requestedScopes = [.fullName, .email]
        
        let controller = ASAuthorizationController(authorizationRequests: [request])
        controller.delegate = self
        controller.performRequests()
    }
    
    func authorizationController(controller: ASAuthorizationController, 
                                 didCompleteWithAuthorization authorization: ASAuthorization) {
        if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
            let idToken = credential.identityToken
            // Send to your server
        }
    }
}
```

### Biometric Authentication

**Android**

```kotlin
class BiometricHelper(private val activity: FragmentActivity) {
    
    private val executor = ContextCompat.getMainExecutor(activity)
    
    fun authenticate(onSuccess: () -> Unit, onError: (String) -> Unit) {
        val biometricPrompt = BiometricPrompt(activity, executor,
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                    onSuccess()
                }
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                    onError(errString.toString())
                }
            })
        
        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle("Biometric Login")
            .setSubtitle("Use fingerprint or face to login")
            .setNegativeButtonText("Use password")
            .build()
        
        biometricPrompt.authenticate(promptInfo)
    }
    
    fun canAuthenticate(): Boolean {
        val manager = BiometricManager.from(activity)
        return manager.canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_STRONG) == 
               BiometricManager.BIOMETRIC_SUCCESS
    }
}
```

**iOS**

```swift
import LocalAuthentication

class BiometricHelper {
    
    func authenticate(completion: @escaping (Bool, Error?) -> Void) {
        let context = LAContext()
        var error: NSError?
        
        if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
            context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                                   localizedReason: "Login with biometrics") { success, error in
                DispatchQueue.main.async {
                    completion(success, error)
                }
            }
        } else {
            completion(false, error)
        }
    }
    
    var biometricType: LABiometryType {
        let context = LAContext()
        _ = context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: nil)
        return context.biometryType // .faceID, .touchID, or .none
    }
}
```

### Refresh Token Flow

```kotlin
class AuthInterceptor(
    private val tokenManager: TokenManager
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer ${tokenManager.accessToken}")
            .build()
        
        val response = chain.proceed(request)
        
        if (response.code == 401) {
            synchronized(this) {
                val newToken = tokenManager.refreshToken()
                val newRequest = request.newBuilder()
                    .header("Authorization", "Bearer $newToken")
                    .build()
                return chain.proceed(newRequest)
            }
        }
        
        return response
    }
}
```

## Caching Strategies

### Cache-Control Headers

```
Cache-Control: max-age=3600        // Cache for 1 hour
Cache-Control: no-cache            // Validate before using
Cache-Control: no-store            // Don't cache
ETag: "abc123"                     // Version identifier
```

### Offline-First Strategy

```kotlin
class UserRepository(
    private val api: UserApi,
    private val dao: UserDao
) {
    fun getUsers(): Flow<List<User>> = flow {
        // Emit cached data first
        emit(dao.getAll())
        
        // Fetch fresh data
        try {
            val fresh = api.getUsers()
            dao.insertAll(fresh)
            emit(fresh)
        } catch (e: Exception) {
            // Already emitted cached data
        }
    }
}
```

## Questions & Answers

> [!question]- Q1: What is the difference between PUT and PATCH?
> **Answer:** 
> - **PUT**: Replaces entire resource with new data
> - **PATCH**: Partially updates resource (only changed fields)
> 
> PUT is idempotent (same result if called multiple times), PATCH may not be.

> [!question]- Q2: How do you handle API errors in mobile apps?
> **Answer:** 
> 1. Wrap API calls in try-catch
> 2. Map HTTP status codes to error types
> 3. Show user-friendly error messages
> 4. Implement retry logic for transient errors
> 5. Log errors for debugging
> 
> Use sealed classes/enums to represent different error states.

> [!question]- Q3: What is JWT and how does it work?
> **Answer:** 
> JWT (JSON Web Token) is a compact, self-contained token for authentication.
> 
> Structure: Header.Payload.Signature
> - Header: Algorithm and token type
> - Payload: Claims (user data, expiry)
> - Signature: Verifies token integrity
> 
> Server validates signature without database lookup.

> [!question]- Q4: How do you implement token refresh?
> **Answer:** 
> 1. Store access token (short-lived) and refresh token (long-lived)
> 2. On 401 response, use refresh token to get new access token
> 3. Retry original request with new token
> 4. If refresh fails, redirect to login
> 
> Use interceptors to handle this transparently.

> [!question]- Q5: What is an OkHttp Interceptor?
> **Answer:** 
> Interceptors observe, modify, and potentially short-circuit requests/responses.
> 
> Types:
> - **Application Interceptors**: Called once, see final request
> - **Network Interceptors**: Called for each network request
> 
> Use cases: Logging, authentication, caching, retry logic.

> [!question]- Q6: How do you implement offline-first in mobile apps?
> **Answer:** 
> Strategy:
> 1. Always read from local database first
> 2. Fetch from network in background
> 3. Update local database with fresh data
> 4. UI observes database changes
> 
> Benefits: Fast initial load, works offline, single source of truth.

> [!question]- Q7: What is the difference between Gson and Moshi?
> **Answer:** 
> | Gson | Moshi |
> |------|-------|
> | Reflection-based | Code generation option |
> | Larger library | Smaller, faster |
> | Lenient by default | Strict by default |
> | No Kotlin support | Kotlin-friendly |
> 
> Moshi is preferred for Kotlin projects.

> [!question]- Q8: How do you secure API communication?
> **Answer:** 
> - Use HTTPS (TLS/SSL)
> - Certificate pinning
> - Token-based authentication
> - Don't store secrets in code
> - Validate server certificates
> - Use secure storage for tokens

> [!question]- Q9: What is certificate pinning?
> **Answer:** 
> Certificate pinning validates that the server's certificate matches a known certificate, preventing man-in-the-middle attacks.
> 
> ```kotlin
> val certificatePinner = CertificatePinner.Builder()
>     .add("api.example.com", "sha256/AAAA...")
>     .build()
> ```
> 
> Downside: Need to update app when certificate changes.

> [!question]- Q10: How do you handle pagination in APIs?
> **Answer:** 
> Common approaches:
> - **Offset-based**: `?page=2&limit=20`
> - **Cursor-based**: `?cursor=abc123&limit=20`
> 
> Cursor-based is better for real-time data (no duplicates/skips).
> 
> Use Paging library (Android) or implement infinite scroll.
