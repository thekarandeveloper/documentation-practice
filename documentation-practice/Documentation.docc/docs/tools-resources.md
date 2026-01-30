# Tools & Resources

This document provides an overview of development tools, resources, and useful links for the Swipe iOS project.

**Last Updated**: 2025-11-22
**Current Version**: 1.2.34 (Build 387)

---

## Table of Contents

1. [Development Tools](#development-tools)
2. [Dependency Management](#dependency-management)
3. [Third-Party Libraries & SDKs](#third-party-libraries--sdks)
4. [Third-Party Services](#third-party-services)
5. [Backend & API Configuration](#backend--api-configuration)
6. [Build Tools & Configuration](#build-tools--configuration)
7. [Testing & Debugging](#testing--debugging)
8. [Design Resources](#design-resources)
9. [Learning Resources](#learning-resources)
10. [Useful Links](#useful-links)

---

## Development Tools

### Primary Development

#### Xcode
Primary IDE for iOS development
- **Minimum Version**: Xcode 14.0+ (supports Swift 5.0)
- **Recommended**: Latest stable Xcode release
- **Download**: [Mac App Store](https://apps.apple.com/app/xcode/id497799835)
- **iOS Deployment Targets**:
  - Main app: iOS 18.1
  - Notification Service: iOS 17.0
  - SwipeWidget: iOS 17.5
  - Minimum supported: iOS 16.0

#### Xcode Command Line Tools
```bash
xcode-select --install
```

### Terminal & Shell

#### Homebrew
Package manager for macOS (required for installing development tools)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### Oh My Zsh
Enhanced terminal experience (optional but recommended)
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Code Quality Tools

#### SwiftLint
Code linting and style enforcement (not currently configured but recommended)
```bash
# Install
brew install swiftlint

# Run
swiftlint
swiftlint --fix  # Auto-fix issues
```

#### SwiftFormat
Code formatting tool (not currently configured but recommended)
```bash
# Install
brew install swiftformat

# Run
swiftformat .
```

---

## Dependency Management

### CocoaPods (Primary)

**Version**: 1.16.2
**Configuration**: `/Swipe/Podfile`
**Lock File**: `/Swipe/Podfile.lock`

```bash
# Install CocoaPods
sudo gem install cocoapods

# Install dependencies
cd Swipe
pod install

# Update dependencies
pod update

# Update specific pod
pod update [POD_NAME]
```

**Important**: Always open `Swipe.xcworkspace`, not `Swipe.xcodeproj`, after running `pod install`.

### Swift Package Manager (SPM)

Not currently used, but available as a secondary option. All dependencies are managed via CocoaPods.

---

## Third-Party Libraries & SDKs

### CocoaPods Dependencies

#### UI Components

**AEOTPTextField** (v1.2.6)
- Purpose: OTP (One-Time Password) input field component
- Usage: Authentication flows, OTP verification screens
- Source: [CocoaPods Trunk](https://cocoapods.org/pods/AEOTPTextField)

**RichEditorView** (v4.0)
- Purpose: Rich text editing component (WYSIWYG editor)
- Usage: Invoice notes, product descriptions
- Source: [GitHub](https://github.com/T-Pro/RichEditorView)
- Branch: master
- Commit: 3bc80911b285d663691d25441f1d92cc9ad2b8bf

#### Other UI Libraries (In Codebase)

- **BottomSheet**: Bottom sheet dialog presentations
- **Charts**: Data visualization and charting
- **DGCharts**: Alternative charting library
- **Lottie**: Animation framework for JSON-based animations
- **Mantis**: Image cropping utility
- **MarkdownUI**: Markdown rendering
- **SSDateTimePicker**: Date and time picker component
- **IQKeyboardManagerSwift**: Automatic keyboard management

#### Image & Media

- **SDWebImage**: Asynchronous image downloading and caching
- **SDWebImageSwiftUI**: SwiftUI integration for SDWebImage

#### Kotlin Multiplatform

- **shared**: KMM shared module (business logic)
- **KMMViewModelCore**: ViewModel support for KMM
- **KMMViewModelSwiftUI**: SwiftUI bindings for KMM ViewModels
- **KMPNativeCoroutinesCombine**: Kotlin coroutines to Combine integration

---

## Third-Party Services

### Firebase Suite

**Project**: vectorx-business
**Configuration**: `GoogleService-Info.plist`

#### Firebase Analytics (GA4)
- **Purpose**: Event tracking, user analytics, and behavioral insights
- **Features**:
  - Custom event logging
  - User properties tracking
  - Performance monitoring metrics
  - Screen view tracking
- **Implementation**: `FirebaseAnalyticsLogger.swift`

#### Firebase Crashlytics
- **Purpose**: Crash reporting and error tracking
- **Features**:
  - Automatic crash collection
  - Custom error logging
  - User identification
- **Enabled**: Both Debug and Release builds

#### Firebase Cloud Messaging (FCM)
- **Purpose**: Push notifications
- **Features**:
  - Remote notifications
  - Topic subscriptions: `ios_test`, `all`, `ios_all`, `ios_global_all`, `paid`, `ios_paid`, `free`, `ios_free`
  - APNS token integration
- **Implementation**: `AppDelegate.swift`, `NotificationService`

#### Firebase Remote Config
- **Purpose**: Dynamic configuration management
- **Features**:
  - Feature flags
  - A/B testing configurations
  - Dynamic content updates
- **Functions**: `fetchConfig()`, `getConfigString()`, `getConfigBool()`, `getConfigJSON()`

#### Firebase Performance Monitoring
- **Purpose**: Application performance tracking
- **Features**:
  - Custom trace monitoring
  - Network request metrics
  - Screen rendering performance
- **Traces**:
  - OTP to login flow timing
  - Encryption key fetch timing
  - Onboarding flow timing

**Firebase Configuration**:
- Database URL: `https://vectorx-business-default-rtdb.firebaseio.com`
- Storage Bucket: `vectorx-business.appspot.com`
- GCM Sender ID: `448547958118`

---

### Google Services

#### Google Sign-In (OAuth 2.0)
- **Purpose**: Social authentication
- **Client ID**: `448547958118-tme8d7hstgfj9a7baearb570cksshhcb.apps.googleusercontent.com`
- **Configuration**: `client_448547958118-tme8d7hstgfj9a7baearb570cksshhcb.apps.googleusercontent.com.plist`
- **URL Scheme**: `com.googleusercontent.apps.448547958118-tme8d7hstgfj9a7baearb570cksshhcb`

---

### Customer Support & Engagement

#### Intercom
- **Purpose**: In-app messaging and customer support
- **App ID**: `be01q1gj`
- **API Key**: `ios_sdk-a1a3b275c5b2e6b6ee4f0dc334aa89bb3052539a`
- **Features**:
  - Live chat
  - Help center
  - User messaging
  - Push notification integration
- **Configuration**: Auto-integration disabled (`IntercomAutoIntegratePushNotifications: NO`)
- **Documentation**: [Intercom iOS SDK](https://developers.intercom.com/installing-intercom/docs/ios-installation)

#### Smartech (Netcore)
- **Version**: 3.5.5
- **Purpose**: Marketing automation and push notifications
- **App ID**: `607e7d1b5336ee1a92b831e09925d4af`
- **Features**:
  - Marketing push notifications
  - Event tracking
  - User segmentation
  - Rich push notifications
- **Configuration**:
  - Bundle ID: `group.getswipe.in`
  - Location tracking: Disabled (`SmartechAutoFetchLocation: false`)
  - Advertising ID: Enabled (`SmartechUseAdvId: true`)
- **Implementation**: `NetcoreEvents.swift`, `SmartechNSEContent`
- **Documentation**: [Smartech iOS SDK](https://docs.netcoresmartech.com/)

---

### Remote Support & Co-browsing

#### CobrowseIO
- **Purpose**: Remote support and screen sharing
- **License**: `KAJCbkX9VBIGzg`
- **Features**:
  - Live screen sharing
  - Remote assistance
  - Session recording
  - Secure co-browsing
- **Configuration**: Auto-start enabled on app launch
- **Documentation**: [CobrowseIO iOS SDK](https://docs.cobrowse.io/sdk-installation/ios)

---

### User Behavior Analytics

#### Microsoft Clarity
- **Purpose**: Session recording and behavioral analytics
- **Project ID**: `t8iwo0cyh8`
- **Features**:
  - Session replays
  - Heatmaps
  - Rage clicks detection
  - User journey analysis
  - Event mirroring from Firebase
- **Configuration**: Debug logging enabled in DEBUG builds
- **Implementation**: Mirrors all Firebase events to Clarity
- **Documentation**: [Clarity iOS SDK](https://learn.microsoft.com/en-us/clarity/)

---

### In-App Purchases & Subscriptions

#### StoreKit 2 (Apple)
- **Purpose**: In-app purchases and subscription management
- **Configuration**: `SwipeInAppPurchaseConfig.storekit`
- **Test Configuration**: `SwipeInAppPurchaseConfigTest.storekit`
- **Features**:
  - Transaction listening
  - Subscription management
  - Receipt validation
- **Developer Team ID**: `BCNTHMXMSB`
- **App Store ID**: `6451307318`

---

### Payment Processing

#### Razorpay
- **Purpose**: Payment gateway integration (referenced)
- **Support**: [Razorpay Support](https://razorpay.com/support/)

---

## Backend & API Configuration

### Primary API

**Base URL**: `https://app.getswipe.in/api/`
**Network Manager**: `NetworkManager.swift`

#### Request Headers
```
Authorization: Bearer {token}
Device_Fingerprint: {device_uuid}
X-Device-ID: {device_identifier}
DeviceHash: {device_hash}
source: ios_{app_version}
Request-Timestamp: {iso8601_timestamp}
X-Request-ID: {unique_request_id}
```

#### API Endpoints

**AI/Streaming API**:
- Endpoint: `{BASE_URL}/ai/streaming_file_assistant`
- Purpose: SwipeAI search functionality

#### Security Configuration

**App Transport Security (ATS)**:
- Domain: `app.getswipe.in`
- TLS Version: TLSv1.2 minimum
- Forward Secrecy: Required
- Allows Insecure HTTP Loads: NO

### Deeplink Support

**Base Domain**: `https://app.getswipe.in`
**URL Scheme**: `swipebilling://`

**Supported Routes**:
- `/list/sales/{hashId}` - Sales documents
- `/onlineorders/{orderId}` - Online orders
- `/list/sales/pending` - Pending documents
- `/reports` - Reports
- `/paymentsTimeline` - Payment timeline
- `/user` - User settings
- Custom URLs via `iosAction` parameter

### Multi-Country Support

**Supported Countries** (19+):
IN (India), AE (UAE), US (United States), UK (United Kingdom), CA (Canada), AU (Australia), SG (Singapore), MY (Malaysia), NZ (New Zealand), ZA (South Africa), KE (Kenya), NG (Nigeria), PH (Philippines), ID (Indonesia), TH (Thailand), VN (Vietnam), BD (Bangladesh), LK (Sri Lanka), PK (Pakistan)

---

## Build Tools & Configuration

### Version Management

**Configuration Files**:
- `/Swipe/Configurations/Version.xcconfig`
- `/Swipe/Configurations/Version.Debug.xcconfig`
- `/Swipe/Configurations/Version.Release.xcconfig`

**Current Version**: 1.2.34
**Build Number**: 387
**Debug Suffix**: `_dev`

### Build Scripts

**CocoaPods Scripts**:
- `/Swipe/Pods/Target Support Files/Pods-Swipe/Pods-Swipe-frameworks.sh`
- `/Swipe/Pods/Target Support Files/Smartech-iOS-SDK/Smartech-iOS-SDK-xcframeworks.sh`

### Project Structure

**Main Project**: `/Swipe/Swipe.xcodeproj/project.pbxproj`
**Workspace**: `/Swipe/Swipe.xcodeproj/project.xcworkspace` (use this)
**Pods Project**: `/Swipe/Pods/Pods.xcodeproj`

### Configuration Files

**Info.plist Files**:
- Main: `/Swipe/Swipe/Info.plist`
- Notification Service: `/Swipe/NotificationService/Info.plist`
- SwipeWidget: `/Swipe/SwipeWidget/Info.plist`
- SmartechNSE Content: `/Swipe/SmartechNSEContent/Info.plist`

**Key Settings**:
- `CFBundleShortVersionString`: 1.2.34
- `CFBundleVersion`: 387
- `FirebaseAppDelegateProxyEnabled`: true
- `ITSAppUsesNonExemptEncryption`: false
- `LSSupportsOpeningDocumentsInPlace`: true
- `UISupportsDocumentBrowser`: true

**Required Permissions**:
- Camera
- Contacts
- Face ID (LocalAuthentication)
- Location (When In Use)
- Microphone
- Photo Library
- Speech Recognition
- User Notifications

---

## Testing & Debugging

### Testing Frameworks

**Status**: No formal testing frameworks currently configured

**Recommended Setup**:
```bash
# Add to Podfile
pod 'Quick', '~> 7.0'
pod 'Nimble', '~> 13.0'
pod 'Mocker', '~> 3.0'  # For network mocking
```

**Test Files**:
- Feature test: `SendEmailTest.swift`
- Beta testing UI: `EnterBetaTestingView.swift`
- StoreKit test config: `SwipeInAppPurchaseConfigTest.storekit`

### Debug Tools

#### Pandora's Box (Developer Menu)
- **Location**: `/Swipe/Swipe/Presentation/DebugTools/PandorasBox.swift`
- **Purpose**: Internal debug menu for multi-country testing
- **Features**:
  - Switch between 19+ countries
  - Manual token manipulation (DEBUG only)
  - Test different market configurations
- **Access**: DEBUG builds only

#### Logging
- **User Default Flag**: `NativeConstants.LOG_ENABLED`
- **Features**:
  - File-based logging
  - Automatic cleanup on logout
  - Debug print statements throughout codebase

#### Performance Monitoring
- Custom performance trace tracking
- Network latency monitoring
- Custom metric recording via Firebase Performance

### Network Debugging

#### Proxyman (Recommended)
HTTP debugging proxy
- **Download**: [proxyman.io](https://proxyman.io/)
- **Features**:
  - Request/response inspection
  - SSL proxying
  - Scripting support
  - Beautiful UI

#### Charles Proxy
Alternative HTTP proxy tool
- **Download**: [charlesproxy.com](https://www.charlesproxy.com/)
- **Use**: Network request debugging

### UI Debugging

#### Reveal
UI inspection and debugging
- **Download**: [revealapp.com](https://revealapp.com/)
- **Features**:
  - View hierarchy in 3D
  - Inspect view properties
  - Runtime modifications

#### Flipper (Facebook)
Platform for debugging mobile apps
- **Download**: [fbflipper.com](https://fbflipper.com/)
- **Features**:
  - Network inspection
  - Layout debugging
  - Plugin ecosystem

### Performance Profiling

#### Instruments (Built into Xcode)

Common templates:
- **Time Profiler**: CPU usage analysis
- **Allocations**: Memory usage tracking
- **Leaks**: Memory leak detection
- **Network**: Network activity monitoring

```bash
# Launch Instruments from terminal
open -a Instruments
```

---

## Design Resources

### Design Tools

#### Figma
Primary design tool for UI/UX
- **Access**: Request from design team
- **Features**:
  - Component library
  - Design handoff
  - Inspect mode
  - Asset export

### Assets & Resources

#### SF Symbols (Apple)
Icon library for iOS
- **Download**: [Apple SF Symbols](https://developer.apple.com/sf-symbols/)
- **Version**: Use latest version compatible with iOS 16+
- **Integration**: Built into Xcode

#### App Assets
- **App Icon**: `Assets.xcassets/AppIcon`
- **Launch Screen**: `LaunchScreen.storyboard`
- **Image Assets**: Organized by feature in `Assets.xcassets`

#### Image Optimization

**ImageOptim** (Recommended)
- **Download**: [imageoptim.com](https://imageoptim.com/)
- **Use**: Compress images before adding to assets

### Color & Design Utilities

**ColorSlurp**
- **Purpose**: Color picker and palette generator
- **Download**: Mac App Store

**Keka**
- **Purpose**: File compression utility
- **Download**: [keka.io](https://www.keka.io/)

---

## Learning Resources

### iOS Development

#### Official Apple Resources
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [Swift.org](https://swift.org/)
- [WWDC Videos](https://developer.apple.com/videos/)
- [Swift Programming Language Book](https://docs.swift.org/swift-book/)
- [Swift Evolution](https://github.com/apple/swift-evolution)

#### Community Resources
- [Ray Wenderlich](https://www.raywenderlich.com/)
- [Hacking with Swift](https://www.hackingwithswift.com/)
- [Swift by Sundell](https://www.swiftbysundell.com/)
- [iOS Dev Weekly Newsletter](https://iosdevweekly.com/)

### Architecture & Patterns

#### MVVM
- SwiftUI MVVM best practices
- Combine framework for reactive programming
- [MVVM in SwiftUI](https://www.hackingwithswift.com/books/ios-swiftui)

#### Kotlin Multiplatform
- [KMM Documentation](https://kotlinlang.org/docs/multiplatform.html)
- [KMM ViewModel](https://github.com/rickclephas/KMM-ViewModel)
- [KMP-NativeCoroutines](https://github.com/rickclephas/KMP-NativeCoroutines)

### Testing
- [Unit Testing Best Practices](https://www.swiftbysundell.com/basics/unit-testing/)
- [UI Testing in Xcode](https://developer.apple.com/documentation/xctest/user_interface_tests)
- [Test-Driven Development (TDD)](https://www.raywenderlich.com/5522-test-driven-development-tutorial-for-ios-getting-started)

### Communities
- **Swift Forums**: [forums.swift.org](https://forums.swift.org/)
- **Stack Overflow**: [Tag: iOS](https://stackoverflow.com/questions/tagged/ios)
- **Reddit**: [r/iOSProgramming](https://www.reddit.com/r/iOSProgramming/)
- **iOS Developers Slack**: Various community Slack groups

---

## Useful Links

### Company Resources

**GitHub Organization**: [swipe-YC21](https://github.com/swipe-YC21)
**Repository**: `swipe-YC21/swipe-ios`
**Main Branch**: `master`
**Current Branch**: `cisco/feature/introductory-offers`

### Apple Developer

- **Developer Portal**: [developer.apple.com](https://developer.apple.com/)
- **App Store Connect**: [appstoreconnect.apple.com](https://appstoreconnect.apple.com/)
- **Certificates & Profiles**: [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list)
- **TestFlight**: Manage beta testing via App Store Connect

### Service Dashboards

- **Firebase Console**: [Firebase Console](https://console.firebase.google.com/project/vectorx-business)
- **Google Cloud Platform**: [GCP Console](https://console.cloud.google.com/)
- **Intercom Dashboard**: [Intercom](https://app.intercom.com/)
- **Smartech Dashboard**: [Netcore Smartech](https://smartech.netcoresmartech.com/)
- **Clarity Dashboard**: [Microsoft Clarity](https://clarity.microsoft.com/)
- **CobrowseIO Dashboard**: [CobrowseIO](https://cobrowse.io/)

### Support & Documentation

- **App Support**: help@getswipe.in
- **App Link**: [https://app.getswipe.in/help](https://app.getswipe.in/help)
- **Main Domain**: [https://app.getswipe.in](https://app.getswipe.in)

---

## Version Control Tools

### Git Clients

#### SourceTree
Visual Git client
- **Download**: [sourcetreeapp.com](https://www.sourcetreeapp.com/)
- **Features**: Visual branch management, staging, commit history

#### GitHub Desktop
Simple Git interface
- **Download**: [desktop.github.com](https://desktop.github.com/)

#### GitKraken
Cross-platform Git client
- **Download**: [gitkraken.com](https://www.gitkraken.com/)

### Git Workflow

**Main Branch**: `master`
**Feature Branches**: `{username}/feature/{feature-name}`
**Bugfix Branches**: `{username}/fix/{bug-name}`
**Release Branches**: `testflight/v{version}`

**Example**:
```bash
# Create feature branch
git checkout -b cisco/feature/new-feature

# Commit changes
git add .
git commit -m "Add new feature"

# Push to remote
git push -u origin cisco/feature/new-feature
```

---

## Recommended macOS Apps

### Productivity
- **Alfred**: Spotlight replacement with workflows
- **Rectangle**: Window management (free)
- **Magnet**: Window management (paid alternative)
- **iTerm2**: Enhanced terminal emulator

### Development
- **Visual Studio Code**: Text editor for non-Xcode files
- **Postman**: API testing and development
- **Paw**: Native macOS API client
- **DB Browser for SQLite**: View Realm/SQLite databases
- **SimSim**: Quick access to iOS Simulator folders

### Utilities
- **The Unarchiver**: Archive extraction (free)
- **Keka**: File compression
- **CleanMyMac**: System maintenance (optional)

---

## CI/CD Tools

### Current Status
**CI/CD**: Not currently configured

### Recommended Setup

#### GitHub Actions
Create `.github/workflows/ios.yml`:

```yaml
name: iOS CI

on:
  push:
    branches: [ master, testflight/* ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: |
          cd Swipe
          pod install
      - name: Build
        run: xcodebuild -workspace Swipe/Swipe.xcworkspace -scheme Swipe -sdk iphonesimulator
```

#### Fastlane (Recommended)
Automation tool for building and releasing

```bash
# Install
sudo gem install fastlane

# Initialize
cd Swipe
fastlane init
```

**Common Fastlane lanes**:
- `fastlane beta` - Build and upload to TestFlight
- `fastlane release` - Build and submit to App Store
- `fastlane test` - Run unit tests

---

## App Store Configuration

**App Store ID**: `6451307318`
**Bundle Identifier**: `com.swipe.mac`
**Team ID**: `BCNTHMXMSB`
**App Category**: Business

### TestFlight

Access via [App Store Connect](https://appstoreconnect.apple.com/)
- Internal testing: Up to 100 testers
- External testing: Up to 10,000 testers
- Current TestFlight branch: `testflight/v1.2.34`

---

## Required Accounts

### Development Accounts
- [x] GitHub account (swipe-YC21 organization)
- [x] Apple Developer account (Team BCNTHMXMSB)
- [x] Firebase/Google Cloud account
- [x] Intercom account
- [x] Smartech (Netcore) account
- [x] CobrowseIO account
- [x] Microsoft Clarity account

### Access Requests
Contact your team lead to request access to:
- GitHub organization
- Firebase project (vectorx-business)
- App Store Connect
- Design files (Figma)
- Analytics dashboards

---

## Support & Help

### Internal Support
1. Check [ARCHITECTURE.md](ARCHITECTURE.md) for codebase structure
2. Check this document for tooling questions
3. Search existing GitHub issues
4. Ask in team Slack channel: `#swipe-ios`

### External Resources
- [Stack Overflow - iOS Tag](https://stackoverflow.com/questions/tagged/ios)
- [Swift Forums](https://forums.swift.org/)
- Apple Developer Forums

### Reporting Issues

**Internal Bugs**: GitHub Issues
**User-Reported Bugs**: via Intercom
**Crashes**: Monitored via Firebase Crashlytics

---

## Quick Reference

### Essential Commands

```bash
# Install dependencies
cd Swipe
pod install

# Open workspace
open Swipe.xcworkspace

# Clean build
# In Xcode: Cmd + Shift + K

# Clean derived data
rm -rf ~/Library/Developer/Xcode/DerivedData

# Reset Simulator
xcrun simctl erase all

# View logs
log stream --predicate 'subsystem contains "com.swipe.mac"' --level debug
```

### Environment Variables

Set in Xcode Scheme > Run > Arguments > Environment Variables:
- `LOG_ENABLED`: Enable debug logging
- `OS_ACTIVITY_MODE`: Set to `disable` to reduce console noise

---

## License Keys & Credentials

All sensitive credentials are stored in:
- `GoogleService-Info.plist` (Firebase)
- `Info.plist` (API keys)
- Keychain (user tokens)
- Environment configuration files (not committed)

**Never commit**:
- API keys directly in code
- Service account JSON files
- `.env` files with credentials
- Provisioning profiles
- Certificates

Use `.gitignore` to exclude sensitive files.

---

**Last Updated**: 2025-11-22
**Maintainer**: Swipe iOS Team
**Version**: 1.2.34

For questions or updates to this document, please create a pull request or contact the iOS team.

---

[← Previous: Architecture](ARCHITECTURE.md) | [Back to README](README.md)
