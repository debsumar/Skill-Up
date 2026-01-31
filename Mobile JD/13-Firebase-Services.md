---
title: Firebase Services
date: 2026-01-31
tags:
  - firebase
  - fcm
  - crashlytics
  - analytics
  - interview
---

# Firebase Services

> [!important] JD Optional Skills
> "Exposure to Firebase services like FCM, Analytics, Crashlytics, Remote Config"
> "Knowledge of push notifications"

## Firebase Cloud Messaging (FCM)

### Android Setup

```kotlin
// build.gradle
implementation("com.google.firebase:firebase-messaging-ktx:23.4.0")

// Service
class MyFirebaseService : FirebaseMessagingService() {
    
    override fun onNewToken(token: String) {
        // Send token to your server
        sendTokenToServer(token)
    }
    
    override fun onMessageReceived(message: RemoteMessage) {
        // Handle notification
        message.notification?.let {
            showNotification(it.title, it.body)
        }
        
        // Handle data payload
        message.data.isNotEmpty().let {
            handleDataMessage(message.data)
        }
    }
    
    private fun showNotification(title: String?, body: String?) {
        val channelId = "default"
        val intent = Intent(this, MainActivity::class.java)
        val pendingIntent = PendingIntent.getActivity(
            this, 0, intent, PendingIntent.FLAG_IMMUTABLE
        )
        
        val notification = NotificationCompat.Builder(this, channelId)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(body)
            .setAutoCancel(true)
            .setContentIntent(pendingIntent)
            .build()
        
        NotificationManagerCompat.from(this).notify(0, notification)
    }
}
```

### iOS Setup

```swift
// AppDelegate
import FirebaseMessaging

class AppDelegate: UIResponder, UIApplicationDelegate, MessagingDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions...) {
        FirebaseApp.configure()
        Messaging.messaging().delegate = self
        
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, _ in
            if granted {
                DispatchQueue.main.async {
                    application.registerForRemoteNotifications()
                }
            }
        }
    }
    
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        // Send token to server
    }
    
    func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable: Any]) {
        // Handle notification
    }
}
```

### Message Types

```
┌─────────────────────────────────────────────────────────┐
│                    FCM Message Types                    │
├─────────────────────────────────────────────────────────┤
│  Notification Message  │  Data Message                  │
├────────────────────────┼────────────────────────────────┤
│  System handles display│  App handles everything        │
│  Auto shows in tray    │  Custom processing             │
│  Limited customization │  Full control                  │
│  Background: auto show │  Background: limited time      │
└────────────────────────┴────────────────────────────────┘
```

## Firebase Crashlytics

### Setup

```kotlin
// build.gradle
implementation("com.google.firebase:firebase-crashlytics-ktx:18.6.0")

// Application class
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        FirebaseCrashlytics.getInstance().setCrashlyticsCollectionEnabled(!BuildConfig.DEBUG)
    }
}
```

### Custom Logging

```kotlin
// Log custom keys
FirebaseCrashlytics.getInstance().apply {
    setCustomKey("user_id", userId)
    setCustomKey("screen", "checkout")
    setUserId(userId)
}

// Log non-fatal exceptions
try {
    riskyOperation()
} catch (e: Exception) {
    FirebaseCrashlytics.getInstance().recordException(e)
}

// Log breadcrumbs
FirebaseCrashlytics.getInstance().log("User clicked checkout button")
```

### iOS

```swift
import FirebaseCrashlytics

// Custom keys
Crashlytics.crashlytics().setCustomValue(userId, forKey: "user_id")
Crashlytics.crashlytics().setUserID(userId)

// Non-fatal
Crashlytics.crashlytics().record(error: error)

// Breadcrumb
Crashlytics.crashlytics().log("User clicked checkout")
```

## Firebase Analytics

### Events

```kotlin
// Log event
Firebase.analytics.logEvent("purchase") {
    param("item_id", "SKU123")
    param("item_name", "Premium Plan")
    param("value", 9.99)
    param("currency", "USD")
}

// Set user property
Firebase.analytics.setUserProperty("subscription_type", "premium")

// Set user ID
Firebase.analytics.setUserId(userId)

// Log screen view
Firebase.analytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW) {
    param(FirebaseAnalytics.Param.SCREEN_NAME, "Home")
    param(FirebaseAnalytics.Param.SCREEN_CLASS, "HomeFragment")
}
```

### Predefined Events

| Event | Use Case |
|-------|----------|
| `login` | User logged in |
| `sign_up` | User registered |
| `purchase` | Completed purchase |
| `add_to_cart` | Added item to cart |
| `screen_view` | Viewed a screen |
| `share` | Shared content |

## Firebase Remote Config

### Setup & Fetch

```kotlin
class RemoteConfigManager {
    private val remoteConfig = Firebase.remoteConfig
    
    init {
        val settings = remoteConfigSettings {
            minimumFetchIntervalInSeconds = if (BuildConfig.DEBUG) 0 else 3600
        }
        remoteConfig.setConfigSettingsAsync(settings)
        remoteConfig.setDefaultsAsync(R.xml.remote_config_defaults)
    }
    
    fun fetchConfig(onComplete: () -> Unit) {
        remoteConfig.fetchAndActivate().addOnCompleteListener { task ->
            if (task.isSuccessful) {
                onComplete()
            }
        }
    }
    
    fun getString(key: String): String = remoteConfig.getString(key)
    fun getBoolean(key: String): Boolean = remoteConfig.getBoolean(key)
    fun getLong(key: String): Long = remoteConfig.getLong(key)
}

// Usage
val featureEnabled = remoteConfigManager.getBoolean("new_feature_enabled")
val apiUrl = remoteConfigManager.getString("api_base_url")
```

### Use Cases

```
┌─────────────────────────────────────────┐
│         Remote Config Use Cases         │
├─────────────────────────────────────────┤
│  Feature Flags    │ Enable/disable      │
│  A/B Testing      │ Different variants  │
│  Emergency Config │ Kill switch         │
│  Personalization  │ User-specific       │
│  Gradual Rollout  │ % of users          │
└─────────────────────────────────────────┘
```

## Firebase Authentication

```kotlin
// Email/Password
Firebase.auth.createUserWithEmailAndPassword(email, password)
    .addOnSuccessListener { result -> /* User created */ }
    .addOnFailureListener { e -> /* Handle error */ }

Firebase.auth.signInWithEmailAndPassword(email, password)
    .addOnSuccessListener { result -> /* Signed in */ }

// Google Sign-In
val credential = GoogleAuthProvider.getCredential(idToken, null)
Firebase.auth.signInWithCredential(credential)

// Current user
val user = Firebase.auth.currentUser
user?.let {
    val uid = it.uid
    val email = it.email
}

// Sign out
Firebase.auth.signOut()
```

## Questions & Answers

> [!question]- Q1: What is the difference between notification and data messages in FCM?
> **Answer:**
> | Notification | Data |
> |--------------|------|
> | System displays automatically | App handles display |
> | Limited to title/body | Custom key-value pairs |
> | Background: auto shows | Background: limited processing |
> | Foreground: onMessageReceived | Always: onMessageReceived |
> 
> Use data messages for full control.

> [!question]- Q2: How do you handle push notifications when app is killed?
> **Answer:**
> - **Notification messages**: System shows automatically
> - **Data messages**: May not be delivered reliably
> - **High priority**: Better delivery but use sparingly
> 
> For critical messages, use notification+data combo.

> [!question]- Q3: What information does Crashlytics capture automatically?
> **Answer:**
> - Stack trace with line numbers
> - Device info (model, OS version)
> - App version
> - Memory state
> - Disk space
> - Battery level
> - Network state
> - Breadcrumb logs

> [!question]- Q4: How do you use Remote Config for feature flags?
> **Answer:**
> 1. Create boolean parameter in Firebase Console
> 2. Set default value in app
> 3. Fetch config on app start
> 4. Check value before showing feature
> ```kotlin
> if (remoteConfig.getBoolean("new_checkout_enabled")) {
>     showNewCheckout()
> }
> ```

> [!question]- Q5: What is the difference between setUserId and setUserProperty in Analytics?
> **Answer:**
> - **setUserId**: Unique identifier for the user (for cross-device tracking)
> - **setUserProperty**: Attributes about the user (subscription type, preferences)
> 
> User properties can be used for audience segmentation.

> [!question]- Q6: How do you test push notifications locally?
> **Answer:**
> Options:
> 1. Firebase Console → Cloud Messaging → Send test message
> 2. Use FCM token in Postman to call FCM API
> 3. Firebase CLI: `firebase functions:shell`
> 
> Always test on real device, not emulator.

> [!question]- Q7: What is Remote Config fetch interval?
> **Answer:**
> Minimum time between fetches to avoid throttling:
> - Default: 12 hours
> - Debug: Can set to 0
> - Production: Recommended 1+ hour
> 
> Fetching too frequently gets throttled (error).

> [!question]- Q8: How do you handle notification channels in Android 8+?
> **Answer:**
> Must create channel before showing notification:
> ```kotlin
> val channel = NotificationChannel(
>     "channel_id",
>     "Channel Name",
>     NotificationManager.IMPORTANCE_HIGH
> )
> notificationManager.createNotificationChannel(channel)
> ```
> Users can disable channels individually.

> [!question]- Q9: What is Firebase App Distribution?
> **Answer:**
> Distribute pre-release builds to testers:
> - Upload APK/IPA via CLI or CI/CD
> - Invite testers by email
> - Testers get notified of new builds
> - Collect feedback
> 
> Alternative to Play Store internal testing / TestFlight.

> [!question]- Q10: How do you implement A/B testing with Remote Config?
> **Answer:**
> 1. Create experiment in Firebase Console
> 2. Define variants (control vs treatment)
> 3. Set target audience (% of users)
> 4. Define goal metric (from Analytics)
> 5. Fetch config in app - Firebase assigns variant
> 6. Analyze results in Console
