# Troubleshooting

Common issues and their solutions for the Swipe iOS project.

## Common Setup Issues

### CocoaPods Issues

#### Pod Install Fails
```bash
# Solution 1: Deintegrate and reinstall
pod deintegrate
pod install

# Solution 2: Update repo and reinstall
pod install --repo-update

# Solution 3: Clear cache
rm -rf ~/Library/Caches/CocoaPods
pod install

# Solution 4: Update CocoaPods
sudo gem install cocoapods
pod install
```

#### Pod Update Takes Forever
```bash
# Update specific pod only
pod update PodName

# Skip repo update
pod install --verbose
```

#### Ruby Version Issues
```bash
# Check Ruby version
ruby --version

# Install rbenv for Ruby version management
brew install rbenv
rbenv install 3.2.0
rbenv global 3.2.0
```

### Code Signing Issues

#### No Signing Certificate Found
1. Check Apple Developer account access
2. Download certificates from developer portal
3. Import certificate into Keychain Access
4. Verify certificate is not expired

#### Provisioning Profile Issues
```bash
# Clear provisioning profiles
rm -rf ~/Library/MobileDevice/Provisioning\ Profiles

# Re-download from Xcode
Xcode > Preferences > Accounts > Download Manual Profiles
```

#### "Automatically manage signing" Not Working
1. Turn off automatic signing
2. Clean build folder (`Cmd + Shift + K`)
3. Turn on automatic signing again
4. Select your team

#### Fastlane Match Issues
```bash
# Reset match certificates (use with caution)
fastlane match nuke development
fastlane match nuke distribution

# Re-create certificates
fastlane match development
fastlane match appstore
```

### Xcode Build Issues

#### Build Fails with "No Such Module"
```bash
# Clean build folder
Cmd + Shift + K

# Or from terminal
rm -rf ~/Library/Developer/Xcode/DerivedData
```

#### Swift Version Mismatch
1. Check project Swift version: Build Settings → Swift Language Version
2. Ensure all targets use same version
3. Update dependencies if needed

#### "Command PhaseScriptExecution failed"
- Check script build phases for errors
- Verify script paths are correct
- Check file permissions
- Review script output in build log

#### Xcode Indexing Stuck
```bash
# Reset Xcode index
rm -rf ~/Library/Developer/Xcode/DerivedData
rm -rf ~/Library/Caches/com.apple.dt.Xcode
```

#### Simulator Not Showing
```bash
# Kill and restart simulator
killall Simulator
open -a Simulator

# Reset simulator
xcrun simctl erase all
```

### Dependency Issues

#### Swift Package Manager Issues
1. File → Swift Packages → Reset Package Caches
2. File → Swift Packages → Update to Latest Package Versions
3. Clean build folder and rebuild

#### Conflicting Dependencies
- Check dependency versions in Podfile/Package.swift
- Review dependency graph for conflicts
- Update to compatible versions

### Git Issues

#### Merge Conflicts
```bash
# View conflicted files
git status

# Resolve conflicts in editor
# Then mark as resolved
git add .
git commit

# Abort merge if needed
git merge --abort
```

#### Accidentally Committed to Wrong Branch
```bash
# Move commits to new branch
git branch new-branch-name
git reset --hard HEAD~n  # n = number of commits to undo
git checkout new-branch-name
```

#### Need to Undo Last Commit
```bash
# Undo commit but keep changes
git reset --soft HEAD~1

# Undo commit and discard changes (careful!)
git reset --hard HEAD~1
```

## Runtime Issues

### App Crashes on Launch

#### Check Crash Logs
1. Window → Devices and Simulators
2. Select device
3. View Device Logs
4. Look for crash report

#### Common Causes
- Missing or incorrect Info.plist settings
- Incorrect scene configuration
- Missing required resources
- Dependency initialization issues

### Debugging Network Issues

#### API Calls Failing
- Check API endpoint URLs
- Verify authentication tokens
- Review request/response in Proxyman/Charles
- Check API server status
- Verify SSL certificate (disable pinning for debugging)

#### SSL Certificate Errors
```swift
// Temporary fix for development (NEVER in production)
#if DEBUG
// Disable SSL pinning for debug builds
#endif
```

### Memory Issues

#### App Using Too Much Memory
- Profile with Instruments (Allocations)
- Check for retain cycles
- Review image loading and caching
- Look for large data structures

#### Memory Leaks
```bash
# Run with leak detection
# Product → Profile → Leaks template
```

Common causes:
- Retain cycles in closures
- Delegates not marked as weak
- NotificationCenter observers not removed

### Performance Issues

#### Slow App Startup
- Profile with Time Profiler
- Review app delegate initialization
- Defer non-critical tasks
- Optimize dependency injection

#### UI Freezing
- Check for work on main thread
- Move heavy operations to background
- Use Instruments to identify bottlenecks

## Testing Issues

### Tests Failing Unexpectedly

#### UI Tests Timing Out
```swift
// Increase timeout
let element = app.buttons["loginButton"]
let exists = NSPredicate(format: "exists == true")
expectation(for: exists, evaluatedWith: element, handler: nil)
waitForExpectations(timeout: 10, handler: nil)
```

#### Flaky Tests
- Add proper wait conditions
- Ensure tests are isolated
- Reset app state between tests
- Avoid hardcoded delays

#### Test Data Issues
- Use mock data
- Reset database before tests
- Don't rely on external services

### CI/CD Issues

#### Build Failing in CI But Works Locally
- Check Xcode version matches CI
- Verify environment variables
- Review dependency versions
- Check file permissions

#### Code Signing Issues in CI
- Verify certificates are valid
- Check Fastlane Match setup
- Ensure secrets are configured
- Review provisioning profiles

## FAQ

### General Questions

**Q: How do I get access to the Apple Developer account?**
A: Contact [team lead name] to be added to the team.

**Q: Where can I find the latest design files?**
A: Design files are in Figma. Link available in [Tools & Resources](tools-resources.md#design-resources).

**Q: How often do we release?**
A: [Specify release cadence - e.g., bi-weekly, monthly]

**Q: Can I work on multiple features at once?**
A: It's better to focus on one feature per branch for easier code review and integration.

### Development Questions

**Q: Should I use CocoaPods or SPM?**
A: [Specify project preference - check current setup in project]

**Q: What iOS versions do we support?**
A: [Specify minimum iOS version, e.g., iOS 15.0+]

**Q: How do I add a new third-party library?**
A:
1. Discuss with team first
2. Add to Podfile or SPM
3. Update documentation
4. Create PR with justification

**Q: My feature branch is behind main. Should I merge or rebase?**
A: We prefer rebasing to keep a clean git history. See [Git Workflow](git-workflow.md#keeping-your-branch-updated).

### Testing Questions

**Q: Do I need to write tests for every change?**
A: Write tests for:
- New business logic
- Bug fixes (to prevent regression)
- Complex algorithms
- Public APIs

**Q: How much code coverage is required?**
A: [Specify target coverage, e.g., aim for 70%+, focus on critical paths]

**Q: UI tests are slow. Can I skip them?**
A: Write UI tests for critical flows only. Unit tests should cover most logic.

## Who to Ask for Help

### By Topic

| Topic | Contact | Channel |
|-------|---------|---------|
| Architecture & Design Patterns | [Name] | `@handle` |
| CI/CD & Build Issues | [Name] | `#ios-builds` |
| Testing & QA | [Name] | `#swipe-ios` |
| App Store & Releases | [Name] | `#ios-releases` |
| Design & UI | Design Team | `#design` |
| API & Backend | Backend Team | `#backend` |
| General Questions | Anyone | `#swipe-ios` |

### Escalation

1. **First**: Check this documentation
2. **Second**: Search Slack history
3. **Third**: Ask in appropriate Slack channel
4. **Fourth**: DM team member directly
5. **Urgent Issues**: Contact iOS lead directly

## Still Stuck?

If you've tried the solutions above and still have issues:

1. **Document the problem**:
   - What are you trying to do?
   - What did you expect?
   - What actually happened?
   - What have you tried?
   - Include error messages and logs

2. **Ask in Slack**:
   - Post in `#swipe-ios` with details above
   - Tag relevant people if known
   - Share screenshots or code snippets

3. **Create a ticket**:
   - For bugs: Create Jira issue
   - For infrastructure: Create DevOps ticket
   - For urgent issues: Escalate to iOS lead

## Useful Debugging Commands

```bash
# Check Xcode version
xcodebuild -version

# List available simulators
xcrun simctl list devices

# Reset simulator
xcrun simctl erase all

# View system logs
log stream --predicate 'process == "YourAppName"'

# Check code signing
codesign -dv --verbose=4 YourApp.app

# Validate IPA
xcrun altool --validate-app -f YourApp.ipa -t ios --apiKey KEY --apiIssuer ISSUER
```

---

[← Previous: Tools & Resources](tools-resources.md) | [Back to Index](../README.md) | [Next: Team Contacts →](team-contacts.md)
