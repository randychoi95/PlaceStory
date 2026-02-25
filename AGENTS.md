# AGENTS.md

## Project Overview

**PlaceStory** (나만의 장소기록 프로젝트) is a native iOS app for recording personal place diaries. It uses a modular Swift Package Manager architecture with the RIBs pattern (ModernRIBs).

### Architecture

- 8 local Swift packages under `PlaceStory/`: `ProxyPackage`, `Domain`, `Platform`, `AppRoot`, `LoggedIn`, `LoggedOut`, `MyLocation`, `PlaceDiary`
- RIBs architecture via [ModernRIBs](https://github.com/DevYeom/ModernRIBs)
- UIKit + SnapKit for UI, Realm for local storage, Firebase for auth/backend
- Swift 5.9, iOS 17+, Xcode project at `PlaceStory/PlaceStory.xcodeproj`

## Cursor Cloud specific instructions

### Platform Limitation

This is an **iOS-only project** — it requires **macOS with Xcode** to build, run, and test. The Cloud Agent Linux VM **cannot**:

- Build the app (`xcodebuild` is macOS-only)
- Run the iOS Simulator
- Execute unit/UI tests (they require iOS Simulator targets)

### What CAN be done on Linux

1. **Validate Swift Package manifests**: `swift package dump-package` works for each package under `PlaceStory/`
2. **Resolve SPM dependencies**: `swift package resolve --package-path PlaceStory/ProxyPackage` fetches all external deps
3. **Fastlane**: `cd PlaceStory && bundle exec fastlane lanes` lists available lanes (deployment only)
4. **Code review / static analysis**: Read and review Swift source files (96 `.swift` files across 8 modules)

### Key Commands (macOS only)

| Task | Command |
|------|---------|
| Build & Test | `xcodebuild clean test -project PlaceStory/PlaceStory.xcodeproj -scheme PlaceStory -destination "platform=iOS Simulator,name=iPhone 15,OS=latest"` |
| Fastlane TestFlight | `cd PlaceStory && bundle exec fastlane upload_testflight` |
| Fastlane App Store | `cd PlaceStory && bundle exec fastlane upload_appstore` |

### Dependency Management

- **Swift dependencies**: Managed via SPM. Packages auto-resolve when opening `.xcodeproj` in Xcode, or via `swift package resolve`.
- **Ruby/Fastlane**: `cd PlaceStory && bundle install` (uses `Gemfile`). Gems are installed to `vendor/bundle` with local path config.

### CI

- GitHub Actions workflow at `.github/workflows/PlaceStory_CI.yml` runs on `macos-13` with `xcodebuild`
- SonarCloud analysis at `.github/workflows/sonarcloud-analyze.yml` runs on `ubuntu-latest`

### Gotchas

- All Swift packages specify `platforms: [.iOS(.v17)]` — `swift build` on Linux will fail due to missing iOS SDK. Use `swift package dump-package` or `swift package resolve` for validation instead.
- `vendor/bundle` is gitignored; run `cd PlaceStory && bundle config set --local path vendor/bundle && bundle install` to restore Fastlane gems.
- Firebase config (`GoogleService-Info.plist`) is committed in the repo.
