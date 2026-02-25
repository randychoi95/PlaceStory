# AGENTS.md

## Project Overview

**PlaceStory** is a native iOS app (iOS 17+) for recording personal place/location stories. It uses a modular architecture with 8 local Swift packages, the RIBs pattern (ModernRIBs), UIKit + SnapKit, Combine, Realm (local DB), Firebase (Auth/Firestore/Analytics), and Apple MapKit.

## Cursor Cloud specific instructions

### Platform Constraint

This is a **pure iOS native app** — it requires **macOS with Xcode 15+** to build, run, and test. The Cursor Cloud Linux VM cannot compile or run the app. CI runs on `macos-13` (see `.github/workflows/PlaceStory_CI.yml`).

### What CAN be done on the Linux VM

- **SwiftLint** (installed at `/usr/local/bin/swiftlint`): Lint all project Swift source files.
  ```
  cd PlaceStory && swiftlint lint PlaceStory/ AppRoot/Sources/ Domain/Sources/ Platform/Sources/ ProxyPackage/Sources/ LoggedOut/Sources/ LoggedIn/Sources/ MyLocation/Sources/ PlaceDiary/Sources/
  ```
- **Swift Package Resolution** (`swift package resolve`): Resolves SPM dependencies. Swift 5.10.1 is installed at `/opt/swift/usr/bin/swift`. Make sure `PATH` includes `/opt/swift/usr/bin`.
  ```
  export PATH="/opt/swift/usr/bin:$PATH"
  cd PlaceStory/Domain && swift package resolve
  ```
- **Fastlane** (via Bundler): List and inspect Fastlane lanes (actual build/deploy requires macOS).
  ```
  cd PlaceStory && bundle exec fastlane lanes
  ```

### What CANNOT be done on the Linux VM

- `xcodebuild` (build, test, archive) — requires macOS + Xcode
- Running the iOS Simulator — requires macOS
- Running unit/integration tests — the CI command is:
  ```
  xcodebuild clean test -project PlaceStory/PlaceStory.xcodeproj \
    -scheme PlaceStory \
    -destination "platform=iOS Simulator,name=iPhone 15,OS=latest"
  ```

### Project Structure

All code lives under `PlaceStory/`:

| Package | Purpose |
|---------|---------|
| `PlaceStory/PlaceStory/` | Main app target (AppDelegate, SceneDelegate) |
| `AppRoot/` | Root RIB wiring all feature modules |
| `Domain/` | Entities, UseCases, Repository protocols |
| `Platform/` | Repository implementations, Realm, MapKit, Keychain |
| `ProxyPackage/` | Shared dependency proxy (ModernRIBs, SnapKit, Realm, Firebase) |
| `LoggedOut/` | Apple Sign-In / logged-out screen |
| `LoggedIn/` | Authenticated tab-based navigation |
| `MyLocation/` | Current location, map, place search |
| `PlaceDiary/` | Place record list & editor |

### No linting config file

The project does not include a `.swiftlint.yml`. SwiftLint uses default rules. When linting, always exclude `.build/` directories by specifying source paths explicitly (see command above).
