# Git Workflow

This document describes the Git workflow and branching strategy for the Swipe iOS project.

## Branching Strategy Overview

### Long-Running Branches
- **master** - Safe production branch, only updated from staging after releases
- **staging** - Integration branch where all changes are tested before production

### Branch Flow
```
<developer>/feature|fix|enhancement|hotfix/* → testflight/v<VERSION> → staging → master
```

### Quick Reference
1. **Create feature branch** from `staging` with name `<your-name>/feature/feature-name`
2. **Open PR** targeting current release train (e.g., `testflight/v1.0.0`)
3. **Get approval** (at least one team member)
4. **Merge** into release train
5. **Test** on TestFlight
6. **After production release**, release train merges to `staging`, then `staging` merges to `master`

### Workflow Diagram
```
┌─────────────────────────────────────────────────────────────┐
│  Developer Workflow                                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  staging ───┬──> nitin/feature/auth ─────┐                  │
│             │                            │                  │
│             ├──> karan/fix/crash ────────┤                  │
│             │                            ├──> testflight/   │
│             ├──> sarah/enhancement/ui ───┤     v1.0.0       │
│             │                            │                  │
│             └──> mike/hotfix/payment ────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  Release Workflow (After Production Release)                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  testflight/v1.0.0  ──>  staging  ──>  master               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Branch Naming Conventions

All development branches should be prefixed with the developer's first name or a specific keyword.

### Feature Branches
```
<developer-name>/feature/description-of-feature
```
Examples:
- `nitin/feature/user-authentication`
- `karan/feature/profile-settings-ui`
- `john/feature/push-notifications`

### Bug Fix Branches
```
<developer-name>/fix/description-of-bug
```
Examples:
- `nitin/fix/login-crash`
- `karan/fix/image-loading-issue`
- `sarah/fix/memory-leak-profile-view`

### Enhancement Branches
```
<developer-name>/enhancement/description-of-enhancement
```
Examples:
- `nitin/enhancement/improve-image-caching`
- `karan/enhancement/optimize-network-calls`
- `mike/enhancement/ui-polish`

### Hotfix Branches
```
<developer-name>/hotfix/description-of-fix
```
Examples:
- `nitin/hotfix/critical-crash-on-launch`
- `karan/hotfix/payment-processing-error`

**Note:** Hotfixes follow the same workflow as features (staging → testflight → staging → master)

### Release Train Branches
```
testflight/v<VERSION>
```
Examples:
- `testflight/v1.0.0`
- `testflight/v1.2.0`
- `testflight/v2.0.0`

**Note:** Only one release train exists at a time

## Commit Message Guidelines

We follow [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format
```
type(scope): description

[optional body]

[optional footer]
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semicolons, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks, dependency updates

### Examples

#### Feature Commit
```
feat(auth): add biometric authentication

Implemented Face ID and Touch ID support for user login.
Users can now enable biometric auth in settings.

Closes #123
```

#### Bug Fix Commit
```
fix(profile): resolve image loading crash

Fixed crash that occurred when loading user profile images
on devices with limited memory.

Fixes #456
```

#### Documentation Commit
```
docs(readme): update installation instructions

Added steps for M1 Mac setup and clarified CocoaPods installation.
```

### Commit Message Best Practices
- Use imperative mood ("add" not "added")
- Keep the subject line under 50 characters
- Capitalize the subject line
- Don't end the subject line with a period
- Separate subject from body with a blank line
- Use the body to explain what and why (not how)

## Release Train Workflow

The Release Train is a branch that collects multiple features, fixes, and enhancements for a specific release version. Only one release train exists at a time to maintain a clear and organized workflow.

### Creating a Release Train

**Who:** Release manager or designated team lead

1. **Create Release Train from Staging**
   ```bash
   git checkout staging
   git pull origin staging
   git checkout -b testflight/v1.0.0
   git push origin testflight/v1.0.0
   ```

2. **Announce to Team**
   - Notify team in Slack/communication channel
   - Share the release train branch name
   - Communicate the release timeline

### Adding Features to Release Train

**Who:** All developers

1. **Merge Features into Release Train**
   - All feature, fix, enhancement, and hotfix PRs target the active release train
   - Base branch for PRs: `testflight/v1.0.0`
   - Ensure at least one approval before merging

2. **Testing on TestFlight**
   - Build is released to TestFlight for testing
   - QA and stakeholders test the release train
   - Additional fixes can be merged into the release train during testing

### Releasing to Production

**Who:** Release manager or designated team lead

1. **Release Build to Production**
   - App is submitted to App Store
   - Production release is approved and deployed

2. **Merge Release Train Back to Main Branches**
   ```bash
   # First, merge release train into staging
   git checkout staging
   git pull origin staging
   git merge testflight/v1.0.0
   git push origin staging

   # Then, merge staging into master
   git checkout master
   git pull origin master
   git merge staging
   git push origin master
   ```

3. **Cleanup**
   ```bash
   # Delete the release train branch after successful merge
   git branch -d testflight/v1.0.0
   git push origin --delete testflight/v1.0.0
   ```

4. **Notify Team**
   - Confirm production release is complete
   - Team can now start working on next release train

**Important Notes:**
- Only one release train exists at a time
- Merging back to staging/master happens AFTER production release, not before
- All developers should pull latest staging after release train is merged

## Pull Request Process

### Creating a Pull Request

1. **Ensure Your Branch is Up to Date**
   ```bash
   git checkout staging
   git pull origin staging
   git checkout your-feature-branch
   git rebase staging
   ```

2. **Push Your Branch**
   ```bash
   git push origin your-feature-branch
   ```

3. **Open a Pull Request**
   - Navigate to the repository on GitHub
   - Click "New Pull Request"
   - **Base branch:** Current release train (e.g., `testflight/v1.0.0`)
   - **Compare branch:** Your feature branch (e.g., `nitin/feature/user-auth`)
   - Fill out the PR template

### Pull Request Guidelines

#### PR Title
Use the same format as commit messages:
```
feat(auth): add biometric authentication
```

#### PR Description
Include:
- Summary of changes
- Motivation and context
- Screenshots (for UI changes)
- Link to related issues
- Testing instructions

#### Example PR Description
```markdown
## Summary
Adds biometric authentication support (Face ID and Touch ID) for user login.

## Changes
- Added biometric authentication service
- Updated login view to support biometric option
- Added settings toggle for biometric auth
- Implemented keychain storage for auth tokens

## Screenshots
[Attach screenshots]

## Related Issues
Closes #123

## Testing
1. Enable biometric auth in Settings
2. Log out
3. Attempt to log in using Face ID/Touch ID
4. Verify successful authentication
```

### PR Size Guidelines
- Keep PRs focused and reasonably sized (< 400 lines changed ideal)
- Break large features into smaller, reviewable chunks
- Each PR should have a single, clear purpose

## Code Review Checklist

### For Authors
Before requesting review, ensure:
- [ ] Code follows style guidelines
- [ ] All tests pass
- [ ] New tests are added for new functionality
- [ ] Documentation is updated
- [ ] No sensitive data is committed
- [ ] SwiftLint passes without warnings
- [ ] PR description is complete and clear

### For Reviewers
When reviewing, check:
- [ ] Code correctness and logic
- [ ] Adherence to coding standards
- [ ] Test coverage
- [ ] Performance implications
- [ ] Security considerations
- [ ] Edge cases handled
- [ ] Error handling is appropriate
- [ ] Documentation is clear

### Review Response Time
- Aim to review PRs within 24 hours
- For urgent PRs, communicate in Slack
- Use GitHub's review features (approve, request changes, comment)

## Merging Strategy

### Merge Requirements
- ✅ At least one approval from a team member
- ✅ All CI checks passing
- ✅ No merge conflicts
- ✅ Branch is up to date with base branch

### Merge Methods
We use **Squash and Merge** for most PRs to maintain a clean commit history.

### After Merging
1. Delete the feature branch (GitHub can do this automatically)
2. Update your local repository:
   ```bash
   git checkout staging
   git pull origin staging
   git branch -d your-feature-branch
   ```

## Working with Branches

### Creating a New Branch

**For new features, fixes, or enhancements:**
```bash
git checkout staging
git pull origin staging
git checkout -b <your-name>/feature/your-feature-name
```

Examples:
```bash
# Feature branch
git checkout -b nitin/feature/user-authentication

# Fix branch
git checkout -b karan/fix/login-crash

# Enhancement branch
git checkout -b sarah/enhancement/improve-caching

# Hotfix branch
git checkout -b nitin/hotfix/critical-payment-bug
```

### Keeping Your Branch Updated

For long-running feature branches, keep them updated with staging:

```bash
# Rebase approach (recommended)
git checkout staging
git pull origin staging
git checkout your-feature-branch
git rebase staging

# Merge approach (if rebase is complicated)
git checkout your-feature-branch
git merge staging
```

### Resolving Conflicts
```bash
# After a rebase or merge conflict
# 1. Resolve conflicts in your editor
# 2. Mark as resolved
git add .
# 3. Continue rebase or complete merge
git rebase --continue
# OR
git commit
```

## Complete Workflow Example

Here's a complete example of the workflow from feature development to production:

```bash
# 1. Start a new feature from staging
git checkout staging
git pull origin staging
git checkout -b nitin/feature/user-profile

# 2. Develop your feature
# ... make changes ...
git add .
git commit -m "feat(profile): add user profile page"

# 3. Keep your branch updated (for long-running features)
git checkout staging
git pull origin staging
git checkout nitin/feature/user-profile
git rebase staging

# 4. Push and create PR targeting the release train
git push origin nitin/feature/user-profile
# Create PR: nitin/feature/user-profile → testflight/v1.0.0

# 5. After PR approval and merge, delete your branch
git checkout staging
git pull origin staging
git branch -d nitin/feature/user-profile

# 6. (Done by release manager) After production release
# Merge testflight → staging → master
```

## Best Practices

### Commit Frequently
- Make small, logical commits
- Each commit should represent a single logical change
- Easier to review and revert if needed

### Write Meaningful Commit Messages
- Future you will thank present you
- Helps teammates understand changes
- Makes git history useful

### Keep Branches Short-Lived
- Merge feature branches as soon as they're ready
- Reduces merge conflicts
- Keeps changes focused
- For longer features, regularly rebase with staging

### Don't Commit Generated Files
- Build artifacts
- IDE-specific files (unless shared team settings)
- Dependencies (add to `.gitignore`)

### Protect Sensitive Data
- Never commit API keys, passwords, or tokens
- Use environment variables or secure configuration
- Check commits before pushing

## Common Scenarios & FAQ

### What if there's no active release train?
- Wait for the release manager to create one
- Coordinate with the team in Slack
- Continue developing on your feature branch from staging

### Can I merge directly to staging?
- No, all changes must go through the release train (testflight branch)
- This ensures proper testing on TestFlight before reaching staging

### What if I need to make changes while PR is in review?
- Simply push new commits to your feature branch
- The PR will automatically update
- Request re-review after making changes

### My feature will take several weeks. What should I do?
- Create your branch from staging
- Regularly rebase with staging to stay up to date (at least weekly)
- When the release train is created, create a PR targeting it
- Consider breaking the feature into smaller, reviewable PRs if possible

### What if a critical bug is found in production?
- Create a hotfix branch from staging: `<your-name>/hotfix/description`
- Create a new release train if needed (coordinate with team)
- Follow the same workflow: hotfix → testflight → staging → master
- Never push directly to master

### How do I know which release train to target?
- Check with your team in Slack
- Look at open branches in the repository
- Release manager will announce the active release train

### Can I work on multiple features at once?
- Yes, create separate branches for each feature
- Keep branch names descriptive and prefixed with your name
- Each feature gets its own PR to the release train

### What if I accidentally committed to the wrong branch?
```bash
# If you haven't pushed yet
git reset HEAD~1  # Undo last commit, keep changes
git stash         # Save your changes
git checkout correct-branch
git stash pop     # Apply your changes

# If you already pushed, contact the team for help
```

## GitHub Labels

Use these labels to categorize and track pull requests:

### Status Labels

- **`WORK-IN-PROGRESS`** - Mark the branch as incomplete if it's not fully completed
- **`🏁 chequered flag`** - Do not merge these PRs, as they are merged into another PR. These will auto-close.
- **`completed`** - Mark completed only if the feature/task is completed
- **`☂️🏎️ parc fermé`** - PR which are ready-to-merge and reviewed
- **`🚓 SAFETY-CAR`** - Mark as on hold if the branch is supposed to be on hold
- **`changes-requested`** - After a review, if there are any changes requested
- **`verification-required`** - Task which is complete, but requires additional verification

### Type Labels

- **`Feature`** - New feature or request
- **`enhancement`** - Enhancement to existing feature
- **`fix`** - Minor bug fixes
- **`bug`** - Something isn't working
- **`💥crashfix💥`** - Fix for a crash which should go in the most recent release
- **`documentation`** - Improvements or additions to documentation

### Special Labels

- **`testflight🛫`** - Tag for testflight branches
- **`DO-NOT-MERGE🙅🏿‍♀️`** - Do not merge this PR under any circumstances
- **`Apple Design Awards`** - Fix for UI Issues
- **`Intercom`** - Issues which were raised in Intercom
- **`INTERNATIONAL 🌏`** - International features/fixes

### Management Labels

- **`duplicate`** - This issue or pull request already exists
- **`good first issue`** - Good for newcomers
- **`help wanted`** - Extra attention is needed
- **`question`** - Further information is requested

### Branch Protection Rules
The following branches should be protected:
- **master** - Requires PR, at least 1 approval, no direct pushes
- **staging** - Requires PR, at least 1 approval, no direct pushes
- **testflight/*** - Requires PR, at least 1 approval, no direct pushes

---

[← Previous: Coding Standards](coding-standards.md) | [Back to Index](../README.md) | [Next: Testing →](testing.md)
