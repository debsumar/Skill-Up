---
title: CI/CD & DevOps
date: 2026-01-31
tags:
  - cicd
  - devops
  - git
  - interview
---

# CI/CD & DevOps

## Git Fundamentals

### Common Commands

```bash
# Branching
git branch feature/login        # Create branch
git checkout feature/login      # Switch branch
git checkout -b feature/login   # Create and switch

# Staging & Committing
git add .                       # Stage all changes
git commit -m "Add login"       # Commit
git commit --amend              # Modify last commit

# Remote
git push origin feature/login   # Push branch
git pull origin main            # Pull changes
git fetch                       # Fetch without merge

# Merging
git merge feature/login         # Merge branch
git rebase main                 # Rebase onto main

# Stashing
git stash                       # Save changes temporarily
git stash pop                   # Apply and remove stash
```

### Branching Strategies

#### Git Flow

```
main ─────●─────────────────●─────────────●───▶
          │                 │             │
          │    release/1.0  │             │
          │    ┌────●───────┤             │
          │    │            │             │
develop ──●────●────●───────●─────●───────●───▶
               │                  │
               │ feature/login    │ feature/profile
               └──────●───────────┘
```

#### Trunk-Based Development

```
main ────●────●────●────●────●────●───▶
         │    │    │    │    │    │
         └────┴────┴────┴────┴────┘
         Short-lived feature branches
```

### Commit Message Convention

```
type(scope): subject

body (optional)

footer (optional)

Types: feat, fix, docs, style, refactor, test, chore
Example: feat(auth): add biometric login support
```

## CI/CD Pipeline

### Pipeline Stages

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│  Build  │──▶│  Test   │──▶│  Lint   │──▶│ Deploy  │──▶│ Release │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
```

### GitHub Actions (Android)

```yaml
name: Android CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Grant execute permission
        run: chmod +x gradlew
      
      - name: Build with Gradle
        run: ./gradlew build
      
      - name: Run tests
        run: ./gradlew test
      
      - name: Build APK
        run: ./gradlew assembleRelease
      
      - name: Upload APK
        uses: actions/upload-artifact@v3
        with:
          name: app-release
          path: app/build/outputs/apk/release/
```

### GitHub Actions (iOS)

```yaml
name: iOS CI

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: macos-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_15.0.app
      
      - name: Install dependencies
        run: pod install
      
      - name: Build
        run: |
          xcodebuild -workspace App.xcworkspace \
            -scheme App \
            -sdk iphonesimulator \
            -destination 'platform=iOS Simulator,name=iPhone 15' \
            build
      
      - name: Run tests
        run: |
          xcodebuild test \
            -workspace App.xcworkspace \
            -scheme App \
            -sdk iphonesimulator \
            -destination 'platform=iOS Simulator,name=iPhone 15'
```

### Fastlane

```ruby
# Android - Fastfile
default_platform(:android)

platform :android do
  lane :beta do
    gradle(task: "clean assembleRelease")
    upload_to_play_store(track: 'beta')
  end
  
  lane :release do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(track: 'production')
  end
end

# iOS - Fastfile
default_platform(:ios)

platform :ios do
  lane :beta do
    build_app(scheme: "App")
    upload_to_testflight
  end
  
  lane :release do
    build_app(scheme: "App")
    upload_to_app_store
  end
end
```

## Build Tools

### Android - Gradle

```kotlin
// build.gradle.kts (app)
android {
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.example.app"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0.0"
    }
    
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro")
        }
    }
    
    flavorDimensions += "environment"
    productFlavors {
        create("dev") {
            applicationIdSuffix = ".dev"
            buildConfigField("String", "API_URL", "\"https://dev.api.com\"")
        }
        create("prod") {
            buildConfigField("String", "API_URL", "\"https://api.com\"")
        }
    }
}
```

### iOS - Xcode Build Settings

```
Build Configurations:
├── Debug
│   ├── Development signing
│   └── Debug symbols
└── Release
    ├── Distribution signing
    └── Optimizations enabled

Schemes:
├── App-Dev (Development)
├── App-Staging (TestFlight)
└── App-Prod (App Store)
```

## Code Signing

### Android

```kotlin
android {
    signingConfigs {
        create("release") {
            storeFile = file("keystore.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

### iOS

```
Certificates:
├── Development Certificate (local testing)
├── Distribution Certificate (App Store/Ad Hoc)
└── Push Notification Certificate

Provisioning Profiles:
├── Development Profile
├── Ad Hoc Profile (internal testing)
└── App Store Profile (production)
```

## Environment Management

```kotlin
// Android - BuildConfig
buildConfigField("String", "API_KEY", "\"${System.getenv("API_KEY")}\"")

// Access in code
val apiKey = BuildConfig.API_KEY
```

```swift
// iOS - xcconfig files
// Dev.xcconfig
API_URL = https:/$()/dev.api.com

// Info.plist
<key>API_URL</key>
<string>$(API_URL)</string>

// Access in code
let apiUrl = Bundle.main.infoDictionary?["API_URL"] as? String
```

## Questions & Answers

> [!question]- Q1: What is the difference between `git merge` and `git rebase`?
> **Answer:** 
> - **Merge**: Creates a merge commit, preserves history
> - **Rebase**: Rewrites history, creates linear timeline
> 
> Use merge for shared branches, rebase for local cleanup.

> [!question]- Q2: Explain Git Flow branching strategy.
> **Answer:** 
> Branches:
> - `main` - Production code
> - `develop` - Integration branch
> - `feature/*` - New features
> - `release/*` - Release preparation
> - `hotfix/*` - Production fixes
> 
> Features merge to develop, releases merge to main and develop.

> [!question]- Q3: What is CI/CD and why is it important?
> **Answer:** 
> - **CI (Continuous Integration)**: Automatically build and test on every commit
> - **CD (Continuous Delivery/Deployment)**: Automatically deploy to staging/production
> 
> Benefits: Early bug detection, faster releases, consistent builds.

> [!question]- Q4: How do you handle secrets in CI/CD?
> **Answer:** 
> - Store in CI/CD platform's secret management
> - Use environment variables
> - Never commit secrets to repository
> - Use tools like GitHub Secrets, GitLab CI Variables
> - Rotate secrets regularly

> [!question]- Q5: What is Fastlane and why use it?
> **Answer:** 
> Fastlane automates mobile app deployment tasks:
> - Building apps
> - Running tests
> - Taking screenshots
> - Uploading to stores
> - Managing certificates
> 
> Reduces manual steps and human error.

> [!question]- Q6: How do you manage different environments (dev/staging/prod)?
> **Answer:** 
> - **Android**: Product flavors with different buildConfigFields
> - **iOS**: Build configurations with xcconfig files
> 
> Each environment has its own API URLs, keys, and settings.

> [!question]- Q7: What is code signing and why is it needed?
> **Answer:** 
> Code signing verifies app authenticity and integrity.
> 
> - **Android**: Keystore with private key signs APK/AAB
> - **iOS**: Certificates + Provisioning Profiles
> 
> Required for app store distribution and device installation.

> [!question]- Q8: How do you handle merge conflicts?
> **Answer:** 
> 1. Identify conflicting files
> 2. Open files and find conflict markers (`<<<<`, `====`, `>>>>`)
> 3. Manually resolve by choosing/combining changes
> 4. Remove conflict markers
> 5. Stage and commit resolved files
> 
> Prevention: Frequent pulls, small PRs, clear ownership.

> [!question]- Q9: What are GitHub Actions?
> **Answer:** 
> GitHub's built-in CI/CD platform. Uses YAML workflow files to define:
> - Triggers (push, PR, schedule)
> - Jobs (build, test, deploy)
> - Steps (individual commands)
> 
> Runs on GitHub-hosted or self-hosted runners.

> [!question]- Q10: What is the purpose of ProGuard/R8 in Android?
> **Answer:** 
> ProGuard/R8 optimizes Android apps:
> - **Shrinking**: Removes unused code
> - **Obfuscation**: Renames classes/methods
> - **Optimization**: Optimizes bytecode
> 
> Results in smaller APK and harder reverse engineering.
