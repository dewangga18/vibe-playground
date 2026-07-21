# Swift — Documentation

> DocC comments — one-liner by default; full breakdown only on request.

## Rules

- Every function (any access level) gets a one-line `///` comment, no exceptions.
- Computed properties: document (they're behavior). Stored, self-descriptive properties (`let id: UUID`): skip, no comment at all.
- Default one-liner only. Add `- Parameter:`/`- Returns:`/`- Throws:` ONLY when explicitly requested — never by default, even for `public`.
- Keep summaries under 150 characters, one sentence. Use `///`, not `//`.
- No `// MARK:` sections unless asked — unrequested, they clutter more than help.

## Do / Don't

- Do: explain purpose, not implementation. Use ``Type/symbol`` backtick links. Add `- Complexity:` for non-trivial algorithms, only inside a requested full breakdown.
- Don't: restate the code (`/// Returns true if true`), use `TODO`/`FIXME`/emoji, or add breakdown/MARK unprompted.

## Template

```swift
/// Fetches a user by ID from the remote service.
public func fetchUser(id: UUID) async throws -> User

/// The user's initials, derived from their display name.
public var initials: String { /* ... */ }

public let id: UUID   // stored, self-descriptive — no comment
```

Full breakdown, only if explicitly requested:
```swift
/// Fetches a user by ID from the remote service.
/// - Parameter id: The unique identifier for the user.
/// - Returns: The user object.
/// - Throws: `NetworkError.notFound` if the user doesn't exist.
public func fetchUser(id: UUID) async throws -> User
```
