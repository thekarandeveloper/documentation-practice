# Getting Started

Welcome to the Swipe iOS team! This guide will help you get up and running with our iOS development environment.

## Prerequisites

### Required Software & Tools
- Xcode (version  26.1.1 or later)
- Git
- [Homebrew](https://brew.sh/) (recommended)

### Recommended IDE Setup
- Xcode version: [26.1.1]
- Xcode Command Line Tools
- Recommended Xcode extensions and plugins

### macOS Version Requirements
- Recommended: macOS [26.1]

## Repository Access & Setup

### GitHub Access
1. Ensure you have access to the [swipe-YC21/swipe-ios](https://github.com/swipe-YC21/swipe-ios/) repository
2. Set up SSH keys for GitHub authentication
3. Configure your Git credentials

### Cloning the Repository
Please refer to our [Installation Guide](installation.md) for instructions on how to clone and set up the project using Android Studio.

### Branch Strategy
- `master` - Safe branch only staging should merge into master after a release
- `staging` - Integration branch for features
- `<KEYWORD>/feature/*` - Feature branches
- `<KEYWORD>/fix/*` - Bug fix branches
- `<KEYWORD>/enhancement/*` - Enhancements for existing features
- `<KEYWORD>/hotfix/*` - Emergency production fixes
- `testflight/v<VERSION>` - Release Train Branch name (e.g. `testflight/v1.0.0`)

## Next Steps

1. Complete [Development Environment Setup](development-setup.md)
2. Read through [Project Architecture](architecture.md)
3. Review [Coding Standards](coding-standards.md)
4. Pick up your first task from the project board
5. Schedule 1:1s with team members

---

[Back to Index](../README.md) | [Next: Development Setup →](development-setup.md)
