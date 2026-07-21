# Swift — Package Manager

> Package.swift structure, dependency commands, and versioning policy.

## Package.swift Structure

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyPackage",
    platforms: [ .iOS(.v17) ],
    products: [ .library(name: "MyPackage", targets: ["MyPackage"]) ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.0.0"),
        .package(url: "https://github.com/Quick/Quick.git", exact: "7.0.0")
    ],
    targets: [
        .target(name: "MyPackage", dependencies: ["Alamofire"])
    ]
)
```

## Dependency Commands

```bash
swift package add <url>                    # Add dependency
swift package update                       # Update all
swift package resolve                      # Resolve versions
swift package show-dependencies            # List tree
swift package reset                        # Reset 
```

## Versioning Policy

- **Libraries**: Range-based (`from: "1.0.0"`) — SemVer guaranteed.
- **Apps**: Exact version (`.exact("1.2.3")`) — reproducible builds.
- **Major**: Breaking API changes.
- **Minor**: New features, backwards compatible.
- **Patch**: Bug fixes only.

## Do

- Maintain `CHANGELOG.md` with Keep a Changelog format.
- Use `internal import` (Swift 5.9+) for internal dependencies.
- Commit `Package.resolved` for apps, ignore for libraries.

## Don't

- Don't commit `Package.resolved` for libraries.
- Don't use outdated or unmaintained packages.