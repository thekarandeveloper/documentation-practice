# Release Management

This document describes the complete release process and versioning strategy for the Swipe iOS application.

## Versioning Strategy

We use an incremental versioning format: `v1.2.44`

### Version Format
- **Format**: `v<MAJOR>.<MINOR>.<BUILD>`
- **Example**: `v1.2.44`

### Version Increments
- **Build number** (third digit): Incremented for each new release
  - Example: `v1.2.44` → `v1.2.45`
- When build reaches 99, increment minor version:
  - Example: `v1.2.99` → `v1.3.0`
- Major version updates are reserved for significant releases:
  - Example: `v1.9.99` → `v2.0.0`

### Version Examples
- `v1.2.44` - Regular release
- `v1.2.45` - Next incremental release
- `v1.2.99` - Last build before minor bump
- `v1.3.0` - New minor version
- `v2.0.0` - Major version update

## Complete Release Process

This is the step-by-step workflow from feature preparation to production release.

### Phase 1: Pre-Release Preparation

#### 1. Ensure All Action Items Are Ready

Before starting a release, verify that all features, fixes, and enhancements are:
- ✅ Code complete
- ✅ Tested locally
- ✅ Ready for review
- ✅ Have standalone PRs from their respective branches

### Phase 2: Create Release Train

#### 2. Create TestFlight Branch from Staging

**Who:** Release manager or designated team lead

```bash
# Checkout and pull latest staging
git checkout staging
git pull origin staging

# Create testflight branch with version number
git checkout -b testflight/v1.2.45
git push origin testflight/v1.2.45
```

**Announce to Team:**
- Post in team Slack/communication channel
- Share the release train branch name
- Communicate the release timeline

### Phase 3: Version Bump

#### 3. Update Version Numbers

Update version numbers in the following files:

**Location**: `Pods/Target Support Files/Pods-Swipe/`

**Files to Update:**
1. **Staging Config**: `Pods-Swipe.debug.xcconfig`
2. **Production Config**: `Pods-Swipe.release.xcconfig`

Update the version number in both files.

#### 4. Update Build Number

**File**: `Info.plist`

Update the `CFBundleVersion` (Build Number) field.

**Commit the version bump:**
```bash
git add .
git commit -m "chore(release): version bump to v1.2.45"
git push origin testflight/v1.2.45
```

### Phase 4: Merge Features into Release Train

#### 5. Pull All Action Items into TestFlight Branch

**Who:** All developers

Each feature/fix/enhancement branch should:
1. Create a PR targeting the testflight branch (e.g., `testflight/v1.2.45`)
2. Get at least one approval from a team member
3. Apply appropriate GitHub labels (see [GitHub Labels](git-workflow.md#github-labels))

**Example PR Flow:**
```
nitin/feature/user-profile → testflight/v1.2.45
karan/fix/login-crash → testflight/v1.2.45
sarah/enhancement/ui-polish → testflight/v1.2.45
```

**Label your PRs appropriately** (refer to [git-workflow.md](git-workflow.md#github-labels) for complete label list):
- `Feature` - New features
- `enhancement` - Enhancements to existing features
- `fix` - Bug fixes
- `💥crashfix💥` - Critical crash fixes
- `☂️🏎️ parc fermé` - Ready to merge and reviewed
- `testflight🛫` - Tag for testflight branches

**Important**: Do NOT merge these PRs into staging directly. They merge into the testflight branch and will auto-merge to staging later.

### Phase 5: Build TestFlight

#### 6. Build TestFlight Release

**Build Configuration:**
- **Scheme**: `Release`
- **Build Target**: `Any iOS Device (arm64)`

**Steps in Xcode:**

1. Select Scheme → `Swipe (Release)`
2. Select Build Target → `Any iOS Device`
3. Product → Archive
4. Wait for archive to complete

### Phase 6: Upload to App Store

#### 7. Distribute to App Store Connect

After successful archive:

1. In the Organizer window, select the archive
2. Click **Distribute App**
3. Select **App Store Connect**
4. Choose **Upload**
5. Select signing options (automatic or manual)
6. Click **Upload**
7. Wait for processing to complete

**Verify Upload:**
- Log into [App Store Connect](https://appstoreconnect.apple.com)
- Navigate to **My Apps** → **Swipe**
- Go to **TestFlight** tab
- Verify the new build appears with correct version number

### Phase 7: TestFlight Distribution

#### 8. Configure TestFlight Release

In App Store Connect → TestFlight:

1. **Select the Build**
   - Click on the newly uploaded build

2. **Add Test Information**
   - What to Test: Describe new features and areas to focus on
   - Export Compliance: Complete if applicable

3. **Select Test Groups**
   - Internal Testing: Team members
   - External Testing: Beta testers (if applicable)

4. **Enable Distribution**
   - For Internal: Automatic distribution
   - For External: Submit for review if needed

#### 9. Test Extensively

**Internal Testing (1-2 days):**
- Core functionality verification
- New features testing
- Regression testing of critical flows
- Performance validation

**External Testing (3-7 days, if applicable):**
- Real-world usage scenarios
- Edge case testing
- Different device types and iOS versions
- Gather feedback from beta testers

**Critical Test Areas:**
- [ ] App launches without crashes
- [ ] Login/authentication flow
- [ ] Core features work as expected
- [ ] New features function correctly
- [ ] Payment flows (if applicable)
- [ ] No memory leaks or performance issues
- [ ] UI/UX is polished
- [ ] No regression in existing features

### Phase 8: App Store Release

#### 10. Create App Store Release

Once TestFlight testing is complete and approved:

**In App Store Connect:**

1. Navigate to **App Store** tab (not TestFlight)
2. Click **+ Version or Platform**
3. Select **iOS**
4. Enter new version number (e.g., `1.2.45`)

5. **Select Build**
   - Click **+ Build** next to Build section
   - Select the TestFlight build you tested

6. **Update Release Information**
   - **What's New in This Version**: Add release notes
   - Update screenshots (if UI changed)
   - Review app description (update if needed)
   - Review keywords and categories

7. **Configure Release**
   - **Manually release this version**: Keep control over release timing
   - OR
   - **Automatically release this version**: Releases after approval

8. **Submit for Review**
   - Complete the required questionnaires
   - Provide demo account if required
   - Add notes for reviewer if necessary
   - Click **Submit for Review**

#### 11. Phased Release (Recommended for Major Updates)

Phased Release gradually rolls out your app over 7 days, allowing you to monitor performance and pause if issues arise.

**How to Enable Phased Release:**

1. In App Store Connect, before submitting for review
2. Scroll to **Phased Release for Automatic Updates**
3. Toggle **ON**

**Phased Release Schedule:**
- Day 1: 1% of users
- Day 2: 2% of users
- Day 3: 5% of users
- Day 4: 10% of users
- Day 5: 20% of users
- Day 6: 50% of users
- Day 7: 100% of users

**When to Use Phased Release:**
- Major version updates (e.g., v2.0.0)
- Significant architecture changes
- Large UI/UX overhauls
- After a previous problematic release
- When introducing new critical features

**Monitoring During Phased Release:**
- Watch crash-free rate (should be > 99.5%)
- Monitor user reviews and ratings
- Track key metrics and analytics
- **Pause release** if you detect issues (button in App Store Connect)

**To Pause Phased Release:**
1. Go to App Store Connect → App Store tab
2. Click on the version
3. Click **Pause Phased Release**
4. Fix issues and submit update or resume release

### Phase 9: Monitor Review Status

#### 12. Track Review Progress

**Review Statuses:**
- **Waiting For Review**: Typical wait time 24-48 hours
- **In Review**: Usually 1-24 hours
- **Pending Developer Release**: Approved, ready to publish (if manual release)
- **Ready for Sale**: Live on App Store
- **Rejected**: Issues found, needs resolution

**Typical Timeline:**
- Submission to In Review: 1-2 days
- Review process: 1-24 hours
- Total time: 1-3 days

**Common Rejection Reasons:**
- Crashes or bugs during review
- Incomplete or misleading app information
- Privacy policy issues or missing disclosures
- Guideline violations (content, functionality)
- Missing features described in metadata
- Performance issues

**If Rejected:**
1. Read the rejection reason carefully
2. Address all issues mentioned
3. Update build if necessary
4. Respond in Resolution Center
5. Resubmit for review

### Phase 10: Post-Release Merging

#### 13. Merge Release Train Back to Main Branches

**Who:** Release manager

**After the build is LIVE on the App Store:**

```bash
# First, merge testflight branch into staging
git checkout staging
git pull origin staging
git merge testflight/v1.2.45
git push origin staging

# Then, merge staging into master
git checkout master
git pull origin master
git merge staging
git push origin master
```

**Cleanup:**
```bash
# Delete the testflight branch locally and remotely
git branch -d testflight/v1.2.45
git push origin --delete testflight/v1.2.45
```

**Notify Team:**
- Post in Slack that release is complete
- Team can pull latest staging
- Ready to start next release cycle

### Phase 11: Create GitHub Release

#### 14. Draft GitHub Release

Navigate to: https://github.com/swipe-YC21/swipe-ios/releases

**Steps:**

1. **Click "Draft a new release"**

2. **Choose a tag**
   - Click "Choose a tag"
   - Type the version number with `v` prefix: `v1.2.45`
   - Click "Create new tag: v1.2.45 on publish"

3. **Target branch**
   - Select the testflight branch: `testflight/v1.2.45`

4. **Release title**
   - Enter the version number: `v1.2.45`

5. **Generate release notes**
   - Click "Generate release notes"
   - GitHub will auto-generate PR list

6. **Clean up release notes**

Transform the auto-generated notes into a clean, human-readable format:

**Format:**
```markdown
## What's Changed
* [Fix] Description of fix by @username in https://github.com/swipe-YC21/swipe-ios/pull/1234
* [Enhancement] Description of enhancement by @username in https://github.com/swipe-YC21/swipe-ios/pull/1235
* [Feature] Description of feature by @username in https://github.com/swipe-YC21/swipe-ios/pull/1236

**Full Changelog**: https://github.com/swipe-YC21/swipe-ios/compare/v1.2.44...v1.2.45
```

**Example:**
```markdown
## What's Changed
* [Fix] Potential Fix for Add Customer Screen hang by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1401
* [Fix] User Profile → Mobile Change issue by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1402
* [Enhancement] Updating existing products to batches by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1403
* [Enhancement] Added reports option under features in more section by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1404
* [Fix] Address Issue in Credit and Debit Notes + UX enhancements in Document Details by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1407
* [Fix] Empty item list issue when duplicating by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1408
* [Enhancement] New background for InAppPurchaseDetailView.swift and notification icon in the more section by @nitinfication in https://github.com/swipe-YC21/swipe-ios/pull/1409

**Full Changelog**: https://github.com/swipe-YC21/swipe-ios/compare/v1.2.44...v1.2.45
```

**Tips for cleaning release notes:**
- Prefix each item with type: `[Fix]`, `[Enhancement]`, `[Feature]`, `[Hotfix]`
- Capitalize first letter of description
- Keep descriptions concise but clear
- Group similar changes together
- Remove unnecessary PRs (like version bumps)

7. **Publish release**
   - Review everything
   - Click **Publish release**

## Post-Release Monitoring

### Metrics to Track

Monitor these metrics closely for 48 hours after release:

**App Performance:**
- **Crash-free rate**: Should be > 99.5%
- **App launch time**: Should not regress
- **API response times**: Monitor for slowdowns
- **Memory usage**: Watch for memory leaks

**User Feedback:**
- **App Store rating**: Monitor for sudden drops
- **User reviews**: Read and respond within 24 hours
- **Support tickets**: Track increase in issues

**Adoption:**
- **Download rate**: Track new installs
- **Update adoption**: % of users on new version
- **Version distribution**: Monitor rollout progress

### Monitoring Tools

**Crash Monitoring:**
- Firebase Crashlytics
- Xcode Organizer
- App Store Connect Analytics

**Analytics:**
- Firebase Analytics
- App Store Connect Analytics
- Custom analytics dashboard

### Action Items

**Within 24 Hours:**
- [ ] Verify crash-free rate is acceptable
- [ ] Check critical user reviews
- [ ] Monitor support channels
- [ ] Validate key metrics

**Within 48 Hours:**
- [ ] Analyze adoption rate
- [ ] Review all crash reports
- [ ] Respond to critical reviews
- [ ] Document any issues for next release

**Within 1 Week:**
- [ ] Complete post-release retrospective
- [ ] Plan hotfix if critical issues found
- [ ] Update documentation based on findings
- [ ] Prepare backlog for next release

## Hotfix Process

### When to Create a Hotfix

Immediate hotfix required for:
- **Critical crashes** affecting > 1% of users
- **Security vulnerabilities**
- **Data loss or corruption issues**
- **Payment processing failures**
- **Legal/compliance issues**

### Hotfix Workflow

1. **Create Hotfix Branch from Staging**
   ```bash
   git checkout staging
   git pull origin staging
   git checkout -b <your-name>/hotfix/critical-issue-description
   ```

2. **Fix the Issue**
   - Make minimal, focused changes
   - Add tests to verify fix
   - Test thoroughly

3. **Create New TestFlight Branch**
   ```bash
   git checkout staging
   git checkout -b testflight/v1.2.46
   git push origin testflight/v1.2.46
   ```

4. **Follow Expedited Release Process**
   - Version bump (increment build number)
   - Merge hotfix PR into testflight branch
   - Build and upload to App Store
   - Submit with request for expedited review

5. **Request Expedited Review**
   - In App Store Connect, use "Request Expedited Review" option
   - Provide clear explanation of critical issue
   - Include impact on users

6. **After Approval**
   - Follow standard merge process: testflight → staging → master
   - Create GitHub release
   - Monitor closely

## Release Checklist

Use this checklist for every release:

### Pre-Release (Before Creating TestFlight Branch)
- [ ] All features are code complete
- [ ] All PRs have been reviewed
- [ ] No known critical bugs
- [ ] Team is aligned on release scope

### Creating Release Train
- [ ] TestFlight branch created from staging
- [ ] Version number updated in config files
- [ ] Build number updated in Info.plist
- [ ] Version bump committed and pushed
- [ ] Team notified of release train

### Merging Features
- [ ] All feature PRs target testflight branch
- [ ] All PRs have proper labels
- [ ] All PRs are reviewed and approved
- [ ] No merge conflicts
- [ ] All CI checks passing

### Building
- [ ] Release scheme selected
- [ ] Any iOS Device target selected
- [ ] Archive successful
- [ ] Build uploaded to App Store Connect
- [ ] Build appears in TestFlight

### Testing
- [ ] Internal testing complete (1-2 days)
- [ ] External testing complete (if applicable)
- [ ] No critical bugs found
- [ ] Performance validated
- [ ] QA sign-off received

### App Store Submission
- [ ] Release created in App Store Connect
- [ ] Build selected
- [ ] Release notes written
- [ ] Screenshots updated (if needed)
- [ ] Phased release configured (if applicable)
- [ ] Submitted for review

### Post-Release
- [ ] Build approved and live
- [ ] TestFlight branch merged to staging
- [ ] Staging merged to master
- [ ] TestFlight branch deleted
- [ ] GitHub release created
- [ ] Release notes cleaned and published
- [ ] Team notified
- [ ] Monitoring dashboards checked
- [ ] No critical issues detected

## Release Calendar

### Typical Release Cadence

**Regular Releases:**
- Frequency: Bi-weekly or monthly (based on team capacity)
- Planning: 1 week before release train creation
- Development: Ongoing
- Testing: 3-5 days on TestFlight
- Review: 1-3 days
- Total cycle: ~2 weeks from feature freeze to production

**Hotfix Releases:**
- As needed for critical issues
- Fast-tracked through all phases
- Minimal testing required
- Expedited review requested

### Release Windows

**Recommended Release Days:**
- **Tuesday or Wednesday**: Best days to release
  - Allows time to monitor before weekend
  - Apple review times are typically faster mid-week
- **Avoid Monday**: Team may not be fully available for monitoring
- **Avoid Friday**: Issues over weekend are harder to address

**Avoid Releasing:**
- Major holidays
- Company-wide events
- When key team members are unavailable
- During iOS beta seasons (June-September) - review times increase

## Version Support Policy

### Supported Versions

- **Current Version**: Full support, all features
- **Previous Version (N-1)**: Critical fixes only
- **Older Versions (N-2 and below)**: No support

### Minimum iOS Version

- Support latest iOS version and previous major version
- Example: If iOS 18 is latest, support iOS 18 and iOS 17
- Evaluate dropping older iOS versions annually

### Force Update Policy

When to require minimum app version:
- Critical security vulnerability in older versions
- Backend API changes incompatible with old versions
- Major features require newer app version
- Performance or stability improvements

## Communication Templates

### Internal Release Announcement

```
🚀 Swipe iOS v1.2.45 Release Train Created

TestFlight Branch: testflight/v1.2.45
Target Release Date: [Date]

All feature, fix, and enhancement PRs should now target: testflight/v1.2.45

Please ensure:
✅ PRs are properly labeled
✅ Code is reviewed
✅ All tests pass

Questions? Ask in #swipe-ios
```

### Release Complete Notification

```
✅ Swipe iOS v1.2.45 is now LIVE on the App Store!

What's new:
• [Summary of key features]
• [Summary of important fixes]

The release has been merged back to staging and master.
Please pull the latest staging branch.

```

### Hotfix Notification

```
🚨 Hotfix Release: Swipe iOS v1.2.46

Critical Issue: [Brief description]
Impact: [User impact]
Fix: [What was fixed]

The hotfix has been submitted for expedited review.
ETA: [Expected timeline]

Monitoring: [Team members monitoring]
```

## Tips for Successful Releases

### Before Release
1. **Communicate early and often** with the team
2. **Test on multiple devices** and iOS versions
3. **Run through critical user flows** manually
4. **Check analytics** to understand current baseline
5. **Have a rollback plan** (hotfix ready if needed)

### During Release
1. **Don't release on Friday** (hard to monitor over weekend)
2. **Have team available** for first 24 hours
3. **Monitor crash reports** in real-time
4. **Watch user reviews** closely
5. **Be ready to pause phased release** if needed

### After Release
1. **Respond to user reviews** within 24 hours
2. **Track key metrics** for at least 48 hours
3. **Document issues** for next release
4. **Conduct retrospective** to improve process
5. **Celebrate the win** with the team! 🎉

---

[← Previous: Git Workflow](git-workflow.md) | [Back to Index](../README.md) | [Next: Tools & Resources →](tools-resources.md)
