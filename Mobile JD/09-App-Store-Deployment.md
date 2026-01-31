---
title: App Store Deployment
date: 2026-01-31
tags:
  - deployment
  - play-store
  - app-store
  - interview
---

# App Store Deployment

## Google Play Store (Android)

### Release Types

| Track | Purpose |
|-------|---------|
| Internal | Quick testing (up to 100 testers) |
| Closed | Limited beta testing |
| Open | Public beta testing |
| Production | Full release |

### App Bundle (AAB)

```kotlin
// build.gradle.kts
android {
    bundle {
        language { enableSplit = true }
        density { enableSplit = true }
        abi { enableSplit = true }
    }
}
```

Benefits over APK:
- Smaller download size
- Dynamic delivery
- Play Feature Delivery

### Release Checklist

- [ ] Update `versionCode` and `versionName`
- [ ] Sign with release keystore
- [ ] Enable ProGuard/R8
- [ ] Test on multiple devices
- [ ] Prepare store listing (screenshots, description)
- [ ] Set content rating
- [ ] Configure pricing & distribution

### Versioning

```kotlin
android {
    defaultConfig {
        versionCode = 10          // Integer, must increment
        versionName = "1.2.0"     // User-visible version
    }
}
```

### Staged Rollout

```
Day 1: 1% of users
Day 3: 5% of users
Day 5: 20% of users
Day 7: 50% of users
Day 10: 100% of users
```

Monitor crash rates and reviews before expanding.

## Apple App Store (iOS)

### Distribution Methods

| Method | Purpose |
|--------|---------|
| TestFlight | Beta testing (internal/external) |
| App Store | Production release |
| Ad Hoc | Direct device installation |
| Enterprise | Internal company distribution |

### App Store Connect

```
App Store Connect
├── My Apps
│   ├── App Information
│   ├── Pricing and Availability
│   ├── App Privacy
│   └── Versions
│       ├── iOS App
│       │   ├── Version Information
│       │   ├── Build
│       │   └── App Review Information
│       └── App Clip (optional)
└── TestFlight
    ├── Internal Testing
    └── External Testing
```

### Release Checklist

- [ ] Increment build number and version
- [ ] Archive with distribution certificate
- [ ] Upload to App Store Connect
- [ ] Fill app metadata
- [ ] Add screenshots for all device sizes
- [ ] Submit for review
- [ ] Respond to review feedback

### Versioning

```swift
// Info.plist
CFBundleShortVersionString = "1.2.0"  // Marketing version
CFBundleVersion = "10"                 // Build number
```

### App Review Guidelines

Common rejection reasons:
- Crashes or bugs
- Incomplete information
- Placeholder content
- Privacy policy missing
- Guideline 4.2 (minimum functionality)

## Release Management

### Semantic Versioning

```
MAJOR.MINOR.PATCH

1.0.0 → 1.0.1  (Bug fix)
1.0.1 → 1.1.0  (New feature, backward compatible)
1.1.0 → 2.0.0  (Breaking change)
```

### Release Notes

```markdown
What's New in Version 1.2.0:

• New dark mode support
• Improved performance on older devices
• Fixed crash when uploading large files
• Bug fixes and stability improvements
```

### Monitoring Post-Release

```
┌─────────────────────────────────────────┐
│           Monitoring Dashboard          │
├─────────────────────────────────────────┤
│  Crash Rate    │  ANR Rate  │  Reviews  │
│     0.5%       │    0.1%    │   4.5★    │
├─────────────────────────────────────────┤
│  Active Users  │  Retention │  Revenue  │
│     50K        │    45%     │   $10K    │
└─────────────────────────────────────────┘
```

Tools:
- Firebase Crashlytics
- Play Console / App Store Connect analytics
- Third-party: Amplitude, Mixpanel

## Push Notifications

### Firebase Cloud Messaging (Android)

```kotlin
class MyFirebaseService : FirebaseMessagingService() {
    
    override fun onNewToken(token: String) {
        // Send token to server
    }
    
    override fun onMessageReceived(message: RemoteMessage) {
        message.notification?.let {
            showNotification(it.title, it.body)
        }
    }
}
```

### APNs (iOS)

```swift
// AppDelegate
func application(_ application: UIApplication, 
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    // Send token to server
}

func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any]) {
    // Handle notification
}
```

## Deep Linking

### Android

```xml
<!-- AndroidManifest.xml -->
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https"
          android:host="example.com"
          android:pathPrefix="/product" />
</intent-filter>
```

### iOS

```swift
// Universal Links - apple-app-site-association
{
    "applinks": {
        "apps": [],
        "details": [{
            "appID": "TEAMID.com.example.app",
            "paths": ["/product/*"]
        }]
    }
}
```

## Questions & Answers

> [!question]- Q1: What is the difference between APK and AAB?
> **Answer:** 
> - **APK**: Single file containing all resources for all devices
> - **AAB**: Upload format that Play Store uses to generate optimized APKs
> 
> AAB benefits: Smaller downloads, dynamic delivery, Play Feature Delivery.

> [!question]- Q2: What is TestFlight?
> **Answer:** 
> Apple's beta testing platform:
> - **Internal testing**: Up to 100 team members, no review
> - **External testing**: Up to 10,000 testers, requires review
> 
> Builds expire after 90 days.

> [!question]- Q3: How do you handle app versioning?
> **Answer:** 
> - **Android**: `versionCode` (integer, must increment), `versionName` (display)
> - **iOS**: `CFBundleVersion` (build), `CFBundleShortVersionString` (marketing)
> 
> Use semantic versioning (MAJOR.MINOR.PATCH) for user-facing version.

> [!question]- Q4: What is staged rollout?
> **Answer:** 
> Gradually releasing to percentage of users (1% → 5% → 20% → 100%).
> 
> Benefits:
> - Catch issues early
> - Limit impact of bugs
> - Monitor metrics before full release

> [!question]- Q5: What are common App Store rejection reasons?
> **Answer:** 
> - Crashes or bugs
> - Incomplete metadata
> - Privacy policy missing
> - Guideline 4.2 (minimum functionality)
> - Misleading app description
> - Inappropriate content

> [!question]- Q6: How do you implement deep linking?
> **Answer:** 
> - **Android**: Intent filters in manifest, App Links for verified domains
> - **iOS**: URL schemes, Universal Links with apple-app-site-association
> 
> Handle incoming URLs in app to navigate to correct screen.

> [!question]- Q7: What is Firebase Crashlytics?
> **Answer:** 
> Real-time crash reporting tool:
> - Automatic crash detection
> - Stack traces with line numbers
> - User impact metrics
> - Integration with Firebase Console
> 
> Essential for monitoring production apps.

> [!question]- Q8: How do you manage release signing?
> **Answer:** 
> - **Android**: Keystore file with private key, Play App Signing recommended
> - **iOS**: Certificates + Provisioning Profiles from Apple Developer Portal
> 
> Store signing credentials securely, never in version control.

> [!question]- Q9: What is Play Feature Delivery?
> **Answer:** 
> Allows delivering app features on-demand:
> - **Install-time**: Included at install
> - **On-demand**: Downloaded when needed
> - **Conditional**: Based on device features
> 
> Reduces initial app size.

> [!question]- Q10: How do you handle app updates?
> **Answer:** 
> - **Forced update**: Block app until updated (for critical fixes)
> - **Flexible update**: Prompt user, allow later
> - **In-app update API** (Android): Native update flow
> 
> Check minimum version from server, compare with installed version.
