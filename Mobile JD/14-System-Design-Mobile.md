---
title: System Design for Mobile
date: 2026-01-31
tags:
  - system-design
  - architecture
  - interview
  - f2f
---

# System Design for Mobile

> [!important] Critical for F2F Interview
> Screen-sharing interviews often include "Design an app like X" questions.

## Approach Framework

```
┌─────────────────────────────────────────────────────────┐
│              MOBILE SYSTEM DESIGN STEPS                 │
├─────────────────────────────────────────────────────────┤
│  1. Clarify Requirements (2-3 min)                      │
│  2. Define Core Features (2 min)                        │
│  3. High-Level Architecture (5 min)                     │
│  4. Data Model (3 min)                                  │
│  5. API Design (3 min)                                  │
│  6. Deep Dive Components (10 min)                       │
│  7. Handle Edge Cases (5 min)                           │
└─────────────────────────────────────────────────────────┘
```

## Step 1: Clarify Requirements

**Questions to ask:**
- What platforms? (Android, iOS, both?)
- Online-only or offline support?
- Expected user base? (affects scalability)
- Real-time features needed?
- Any specific performance requirements?

## Step 2: Core Features

**Example: Design a Food Delivery App**

```
Must Have (MVP):
├── User authentication
├── Browse restaurants
├── View menu & add to cart
├── Checkout & payment
├── Order tracking
└── Order history

Nice to Have:
├── Search & filters
├── Reviews & ratings
├── Favorites
├── Push notifications
└── Live delivery tracking
```

## Step 3: High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      MOBILE APP                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │     UI      │  │  ViewModel  │  │ Repository  │         │
│  │   Layer     │◀─│   Layer     │◀─│   Layer     │         │
│  └─────────────┘  └─────────────┘  └──────┬──────┘         │
│                                           │                 │
│                    ┌──────────────────────┼──────────┐      │
│                    │                      │          │      │
│              ┌─────▼─────┐         ┌──────▼─────┐   │      │
│              │  Remote   │         │   Local    │   │      │
│              │   Data    │         │   Data     │   │      │
│              │  Source   │         │  Source    │   │      │
│              └─────┬─────┘         └────────────┘   │      │
└────────────────────┼────────────────────────────────┘      │
                     │                                        │
                     ▼                                        │
┌─────────────────────────────────────────────────────────────┐
│                      BACKEND                                │
├─────────────────────────────────────────────────────────────┤
│  API Gateway → Load Balancer → App Servers → Database       │
│                                    │                        │
│                              Cache (Redis)                  │
└─────────────────────────────────────────────────────────────┘
```

## Step 4: Data Model

```kotlin
// User
data class User(
    val id: String,
    val name: String,
    val email: String,
    val phone: String,
    val addresses: List<Address>
)

// Restaurant
data class Restaurant(
    val id: String,
    val name: String,
    val cuisine: String,
    val rating: Float,
    val deliveryTime: Int,  // minutes
    val location: LatLng,
    val isOpen: Boolean
)

// Order
data class Order(
    val id: String,
    val userId: String,
    val restaurantId: String,
    val items: List<OrderItem>,
    val status: OrderStatus,
    val totalAmount: Double,
    val createdAt: Long
)

enum class OrderStatus {
    PLACED, CONFIRMED, PREPARING, OUT_FOR_DELIVERY, DELIVERED, CANCELLED
}
```

## Step 5: API Design

```
Authentication:
POST /auth/login          → { email, password }
POST /auth/register       → { name, email, password, phone }
POST /auth/refresh        → { refreshToken }

Restaurants:
GET  /restaurants         → ?lat=&lng=&radius=&cuisine=
GET  /restaurants/{id}    → Restaurant details
GET  /restaurants/{id}/menu → Menu items

Orders:
POST /orders              → { restaurantId, items[], addressId }
GET  /orders              → User's order history
GET  /orders/{id}         → Order details
GET  /orders/{id}/track   → Real-time tracking (WebSocket)

Cart:
GET  /cart                → Current cart
POST /cart/items          → Add item
PUT  /cart/items/{id}     → Update quantity
DELETE /cart/items/{id}   → Remove item
```

## Step 6: Deep Dive Components

### Offline Support

```
┌─────────────────────────────────────────┐
│           Offline Strategy              │
├─────────────────────────────────────────┤
│  1. Cache restaurant list locally       │
│  2. Queue orders when offline           │
│  3. Sync when connection restored       │
│  4. Show cached data with "offline" UI  │
└─────────────────────────────────────────┘

// Repository pattern
class RestaurantRepository(
    private val api: RestaurantApi,
    private val dao: RestaurantDao
) {
    fun getRestaurants(): Flow<List<Restaurant>> = flow {
        // Emit cached first
        emit(dao.getAll())
        
        // Fetch fresh
        try {
            val fresh = api.getRestaurants()
            dao.insertAll(fresh)
            emit(fresh)
        } catch (e: IOException) {
            // Already emitted cached
        }
    }
}
```

### Real-time Order Tracking

```
Options:
1. Polling (simple but inefficient)
2. WebSocket (real-time, complex)
3. Server-Sent Events (one-way real-time)
4. Firebase Realtime Database (easy setup)

// WebSocket approach
class OrderTrackingSocket {
    private var socket: WebSocket? = null
    
    fun connect(orderId: String, onUpdate: (OrderStatus) -> Unit) {
        val request = Request.Builder()
            .url("wss://api.example.com/orders/$orderId/track")
            .build()
        
        socket = client.newWebSocket(request, object : WebSocketListener() {
            override fun onMessage(webSocket: WebSocket, text: String) {
                val status = parseStatus(text)
                onUpdate(status)
            }
        })
    }
}
```

### Image Loading & Caching

```kotlin
// Use Coil (Android) or SDWebImage (iOS)
AsyncImage(
    model = ImageRequest.Builder(context)
        .data(restaurant.imageUrl)
        .crossfade(true)
        .memoryCachePolicy(CachePolicy.ENABLED)
        .diskCachePolicy(CachePolicy.ENABLED)
        .build(),
    contentDescription = restaurant.name
)
```

### Pagination

```kotlin
// Paging 3 (Android)
class RestaurantPagingSource(
    private val api: RestaurantApi
) : PagingSource<Int, Restaurant>() {
    
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Restaurant> {
        val page = params.key ?: 1
        return try {
            val response = api.getRestaurants(page, params.loadSize)
            LoadResult.Page(
                data = response.data,
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (response.data.isEmpty()) null else page + 1
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }
}
```

## Step 7: Edge Cases

| Scenario | Solution |
|----------|----------|
| No internet | Show cached data, queue actions |
| Slow network | Show loading states, timeouts |
| Payment failure | Retry mechanism, save cart |
| App killed during order | Persist order state locally |
| Location unavailable | Manual address entry |
| Session expired | Auto-refresh token |

## Common Design Questions

### Design Instagram/Photo App

```
Key Components:
├── Feed (infinite scroll, pagination)
├── Image upload (compression, background upload)
├── Image caching (memory + disk)
├── Stories (auto-advance, caching)
└── Real-time likes/comments
```

### Design Chat App

```
Key Components:
├── WebSocket for real-time messages
├── Local database for message history
├── Message queue for offline
├── Push notifications
├── Read receipts
└── Media upload/download
```

### Design E-commerce App

```
Key Components:
├── Product catalog (search, filters)
├── Cart management (local + server sync)
├── Checkout flow (payment integration)
├── Order tracking
└── Wishlist
```

## Questions & Answers

> [!question]- Q1: How do you handle offline-first in mobile apps?
> **Answer:**
> 1. Always read from local database first
> 2. Show cached data immediately
> 3. Fetch from network in background
> 4. Update local database
> 5. UI observes database changes
> 
> Use Room/CoreData as single source of truth.

> [!question]- Q2: How do you implement infinite scroll/pagination?
> **Answer:**
> - Use Paging library (Android) or similar
> - Load next page when user near end
> - Show loading indicator at bottom
> - Handle errors with retry
> - Cache pages locally for offline

> [!question]- Q3: How do you handle image loading efficiently?
> **Answer:**
> - Use libraries: Coil/Glide (Android), SDWebImage (iOS)
> - Memory cache for recently viewed
> - Disk cache for persistence
> - Resize images to view size
> - Use placeholders during load

> [!question]- Q4: How do you implement real-time features?
> **Answer:**
> Options by use case:
> - **Chat**: WebSocket
> - **Live updates**: Server-Sent Events
> - **Presence**: Firebase Realtime DB
> - **Simple updates**: Polling (last resort)
> 
> Consider battery and data usage.

> [!question]- Q5: How do you secure sensitive data in mobile apps?
> **Answer:**
> - Use HTTPS for all API calls
> - Store tokens in secure storage (Keychain/EncryptedPrefs)
> - Certificate pinning
> - Don't log sensitive data
> - Obfuscate code (ProGuard)
> - Biometric authentication for sensitive actions

> [!question]- Q6: How do you handle app state when process is killed?
> **Answer:**
> - Save critical state to persistent storage
> - Use `onSaveInstanceState` for UI state
> - ViewModel survives config changes
> - Restore state in `onCreate`
> - For forms: auto-save drafts

> [!question]- Q7: How do you design for different screen sizes?
> **Answer:**
> - Use constraint-based layouts
> - Responsive breakpoints
> - Adaptive layouts (tablet vs phone)
> - Test on multiple devices
> - Use density-independent units (dp/pt)

> [!question]- Q8: How do you handle background tasks?
> **Answer:**
> - **Android**: WorkManager for guaranteed execution
> - **iOS**: Background Tasks framework
> - Foreground service for long-running tasks
> - Consider battery optimization restrictions

> [!question]- Q9: How do you implement search with filters?
> **Answer:**
> - Debounce search input (300-500ms)
> - Local filtering for small datasets
> - Server-side for large datasets
> - Cache recent searches
> - Show search suggestions

> [!question]- Q10: How do you handle deep linking?
> **Answer:**
> 1. Register URL schemes/Universal Links
> 2. Parse incoming URL in app
> 3. Navigate to appropriate screen
> 4. Handle case when app not installed (deferred deep link)
> 5. Track deep link analytics
