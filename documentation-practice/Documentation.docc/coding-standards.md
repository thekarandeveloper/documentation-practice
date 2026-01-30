# Coding Standards & Best Practices

This document outlines the coding standards and best practices for the Swipe iOS project.

## Swift Style Guide

We follow [Swift.org API Design Guidelines](https://swift.org/documentation/api-design-guidelines/) and [Ray Wenderlich Swift Style Guide](https://github.com/raywenderlich/swift-style-guide).

### Key Principles
- Use descriptive variable and function names
- Prefer `let` over `var` when possible
- Use type inference where appropriate
- Follow naming conventions for protocols, enums, and classes
- Write self-documenting code

### Code Examples

```swift
// Good
let maximumNumberOfLoginAttempts = 3
func fetchUserProfile(for userId: String) { }

// Avoid
let max = 3
func get(id: String) { }
```

## Code Review Process

1. Create a pull request from your feature branch
2. Request review from at least one team member
3. Address feedback and make necessary changes
4. Ensure all CI checks pass
5. Obtain approval before merging

### What Reviewers Look For
- Code correctness and logic
- Adherence to style guidelines
- Test coverage
- Performance considerations
- Security implications
- Documentation completeness

## Naming Conventions

### Types
- **Classes and Structs**: `PascalCase`
  ```swift
  class UserProfileViewController { }
  struct LoginRequest { }
  ```

- **Protocols**: Descriptive nouns or `-able` suffix
  ```swift
  protocol Networking { }
  protocol Clickable { }
  ```

- **Enums**: `PascalCase` for type, `camelCase` for cases
  ```swift
  enum UserStatus {
      case active
      case inactive
      case suspended
  }
  ```

### Functions and Variables
- **Functions**: `camelCase`, descriptive verbs
  ```swift
  func authenticateUser() { }
  func calculateTotalPrice() { }
  ```

- **Variables and Constants**: `camelCase`
  ```swift
  let userName = "John"
  var itemCount = 0
  ```

- **Static Constants**: `camelCase` or `UPPER_SNAKE_CASE`
  ```swift
  static let defaultTimeout = 30.0
  static let API_BASE_URL = "https://api.example.com"
  ```

## SwiftLint Configuration

We use SwiftLint to enforce code style. Run it locally before committing:

```bash
swiftlint
```

### Automatic Fixes
Some violations can be auto-corrected:
```bash
swiftlint --fix
```

### Configuration
SwiftLint rules are configured in `.swiftlint.yml` at the project root.

## Documentation Standards

### Public APIs
Document all public APIs using Swift documentation comments:

```swift
/// Authenticates a user with the provided credentials.
///
/// - Parameters:
///   - username: The user's username
///   - password: The user's password
/// - Returns: An authenticated user object
/// - Throws: `AuthenticationError` if credentials are invalid
func authenticate(username: String, password: String) throws -> User {
    // Implementation
}
```

### Complex Logic
Add comments for complex algorithms or non-obvious business logic:

```swift
// Calculate compound interest using the formula: A = P(1 + r/n)^(nt)
// where P = principal, r = rate, n = compounds per year, t = years
let finalAmount = principal * pow(1 + rate / compoundsPerYear, compoundsPerYear * years)
```

## Best Practices

### Optionals
- Use optional binding when appropriate
- Prefer `guard` for early exits
- Use optional chaining for safe access

```swift
// Good
guard let user = currentUser else { return }
processUser(user)

// Also good
if let name = user?.profile?.name {
    displayName(name)
}
```

### Error Handling
- Use Swift's error handling mechanism
- Create custom error types for domain-specific errors
- Provide meaningful error messages

```swift
enum NetworkError: Error {
    case invalidURL
    case requestFailed(statusCode: Int)
    case decodingFailed
}
```

### Memory Management
- Be aware of retain cycles
- Use `[weak self]` in closures when appropriate
- Avoid unnecessary object retention

```swift
networkService.fetchData { [weak self] result in
    guard let self = self else { return }
    self.handleResult(result)
}
```

### Code Organization
- Use `// MARK:` comments to organize code
- Group related functionality together
- Keep files focused and reasonably sized

```swift
// MARK: - Lifecycle
override func viewDidLoad() { }

// MARK: - UI Setup
private func setupViews() { }

// MARK: - Actions
@objc private func buttonTapped() { }

// MARK: - Networking
private func fetchData() { }
```

## Code Formatting

### Line Length
- Maximum line length: 120 characters
- Break long lines at logical points

### Spacing
- Use 4 spaces for indentation (no tabs)
- Add blank lines to separate logical sections
- Single space after commas and around operators

### Braces
- Opening braces on the same line
- Closing braces on a new line

```swift
func example() {
    if condition {
        doSomething()
    } else {
        doSomethingElse()
    }
}
```

## Security Considerations

- Never hardcode sensitive information
- Validate all user input
- Use secure communication (HTTPS)
- Follow iOS security best practices
- Protect against common vulnerabilities (XSS, injection, etc.)

---

[← Previous: Architecture](architecture.md) | [Back to Index](../README.md) | [Next: Git Workflow →](git-workflow.md)
