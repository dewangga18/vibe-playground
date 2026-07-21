# Swift — Dependency Injection

> Protocol-based DI patterns; avoid singletons and global state.

## Do

- Define protocols for all external collaborators.
- Inject dependencies via initializer (preferred) or property injection.
- Use factory protocols when creation is non-trivial.
- Keep DI containers lightweight; prefer manual wiring.
- Accept protocols in public API (`func process(service: DataServiceProtocol)`).
- Use `@MainActor` injection for UI-bound dependencies.

## Don't

- Don't use singletons (`static let shared`) for testable code.
- Don't access global state from services or view models.
- Don't use service locators (`ServiceLocator.get()`) — inject explicitly.
- Don't mock the System Under Test (SUT).
- Don't inject concrete types where a protocol suffices.

## Template

```swift
protocol AuthService {
    func login(user: String, pass: String) async throws -> User
}

final class LoginViewModel {
    private let authService: AuthService

    init(authService: AuthService) {
        self.authService = authService
    }

    func login(user: String, pass: String) async throws -> User {
        try await authService.login(user: user, pass: pass)
    }
}
```