# Standards — Swift
<!-- locale: en -->

> Swift for iOS/macOS/tvOS/watchOS/server-side apps — strong typing, protocols, structured concurrency, modern package management.

## Core Rules

- **Swift 6.0 minimum** — strict concurrency checking enabled per target.
- **iOS deployment target** — user-specified: 15.6+ (ObservedObject) or 17.0+ (Observable).
- **Documentation** — one-liner DocC on every function + computed property; skip trivial stored properties; no breakdown or `// MARK:` unless asked. Detail: `documentation.md`.
- **Formatter + Linter** — SwiftFormat + SwiftLint in CI, no manual formatting debates.
- **API Design** — follow Official Swift API Design Guidelines.
- **New file registration** — a file on disk isn't registered in Xcode (reference/target/scheme) until verified. Detail: `structure.md` → Xcode Project Registration. Applies to every new file — check it yourself, don't wait for a keyword match.

## Sections

| Section | Trigger pattern | Description | File |
|---|---|---|---|
| `structure` | "where should this file go", "file naming", "folder structure", "access control", "new file", "pbxproj" | File/folder org, naming, access control, Xcode registration | `~/.ai/standards/swift/parts/structure.md` |
| `testing` | "add a test", "write tests", "unit test", "test target" | Swift Testing conventions, test target setup | `~/.ai/standards/swift/parts/testing.md` |
| `dependency-injection` | "DI", "dependency injection", "mock", "avoid singleton" | Protocol-based DI, factories, avoid global state | `~/.ai/standards/swift/parts/dependency-injection.md` |
| `concurrency` | "async", "await", "actor", "Sendable", "TaskGroup" | Actors, Sendable, structured concurrency | `~/.ai/standards/swift/parts/concurrency.md` |
| `error-handling` | "throws", "do/catch", "Result", "typed throws" | Typed throws, Result, do/catch | `~/.ai/standards/swift/parts/error-handling.md` |
| `documentation` | "doc comment", "DocC", "full documentation" | Wording style, symbol links, templates | `~/.ai/standards/swift/parts/documentation.md` |
| `package-manager` | "Package.swift", "package", "SPM", "dependency" |  SPM commands, dependencies, libraries | `~/.ai/standards/swift/parts/package-manager.md` |

Read the matched file before responding. Preload every section only when bootstrapping a brand-new/empty project.
The two Core Rules above always apply regardless of trigger match — they're your responsibility to check, not the user's to ask for.
