# Development Environment Setup

This guide covers the setup of your development environment for the Swipe iOS project.

## Xcode Configuration

[Instructions for configuring Xcode, including themes, shortcuts, and preferences]

## Dependency Management

### Package Manager
The project uses **Swift Package Manager (SPM)** for dependency management.

### Installing Dependencies
Dependencies are managed natively within Xcode and should resolve automatically when you open the project.

To manually resolve or update dependencies:
1. Open the project in Xcode
2. Navigate to **File > Packages > Resolve Package Versions**
3. Wait for the resolution to complete (check the progress bar in the status area)

## Code Signing & Certificates

[Instructions for setting up development certificates and provisioning profiles]

1. Access the Apple Developer account
2. Download development certificates
3. Install provisioning profiles
4. Configure signing in Xcode

## Environment Variables & Configuration

Configuration files are located in `Swipe > Pods`. These files are currently used to manage version numbers and append a `_dev` identifier to debug builds. This identifier helps distinguish between production and debug environments when analyzing alerts (emails) or crash reports.

### Configuration Files
- Staging: `Pods-Swipe.debug.xcconfig`
- Production: `Pods-Swipe.release.xcconfig`

### API Keys & Secrets
- Never commit API keys to version control
- Use environment-specific configuration files
- Store sensitive data in keychain or secure storage

## Running the App Locally

1. Open `Swipe.xcworkspace` in Xcode
2. Select the appropriate scheme (Development/Staging/Production)
3. Choose a simulator or connected device
4. Press `Cmd + R` to build and run

### Selecting Build Schemes
- **Development**: Local development with debug logging
- **Staging**: Internal Release Candidates
- **Production**: Production configuration (use with caution)

## Troubleshooting Setup

If you encounter issues, check the [Troubleshooting Guide](troubleshooting.md) or ask in the `#swipe-ios` Slack channel.

---

[← Previous: Getting Started](getting-started.md) | [Back to Index](../README.md) | [Next: Architecture →](architecture.md)
