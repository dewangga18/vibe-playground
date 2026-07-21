# Swift — Testing

> Unit test conventions using Swift Testing (`import Testing`).

## Rules

* Use Swift Testing (`import Testing`), never XCTest. AAA pattern (Arrange → Act → Assert).
* One `@Test` per behavior; isolated, deterministic, fast — no conditionals/loops/logic inside tests.
* Inject all dependencies; no Singleton/shared/global state. Mock only collaborators, never the SUT (always `sut`).
* `#expect` for assertions, `#require` only for prerequisites. Descriptive behavior-based names.
* Name files `<TypeName>Tests.swift`, mirror the production folder hierarchy under `Tests/`.
* New test file isn't done until registered (reference+target+scheme) — see `structure.md` → Xcode Project Registration.
* Test target doesn't exist yet? See "Test Target Setup" below first.

## Template

```swift
import Testing
@testable import YourAppTarget

@Suite("Authentication Tests")
struct LoginTests {
    @Test("Login fails with invalid credentials")
    func loginFailure() {
        let sut = AuthManager(service: MockAuthService())
        let result = sut.login(user: "badUser", pass: "wrongPass")
        #expect(result == .failure(.invalidCredentials))
    }
}
```

## Test Target Setup (new targets only)

1. **Exists?** `ruby -rxcodeproj -e 'puts Xcodeproj::Project.open("<Proj>.xcodeproj").targets.map(&:name)'`. Yes → register the file in its tree (`structure.md`). No → create one matching the app target's Swift version + min iOS deployment target.
2. **Wire it** — each item below is independently easy to forget:
   - **Dependency:** `PBXTargetDependency` → app target via `PBXContainerItemProxy` (gem: `test_target.add_dependency(app_target)`).
   - **`baseConfigurationReference`:** reuse sibling targets' `.xcconfig` if the project uses flavors — don't leave it nil.
   - **`PRODUCT_NAME`:** one consistent value across configs (`TEST_HOST` resolves against this). `PRODUCT_BUNDLE_IDENTIFIER` legitimately varies per environment — that's fine, don't flag it.
   - **`TEST_HOST`:** dynamic via `$(APP_NAME)`/`$(PRODUCT_NAME)`, never hardcoded.
   - **`ENABLE_TESTABILITY = YES`:** on the app target's Debug config, or `@testable import` can't see `internal` members.
3. Always `@testable import <AppModule>`, never plain `import` — "cannot find X in scope" means the import or step 2's testability setting is wrong.
4. Run via `xcodebuild test -scheme "<Scheme>" -destination "..."` — `.xcodeproj` app, not pure SPM; `swift test` won't work.
