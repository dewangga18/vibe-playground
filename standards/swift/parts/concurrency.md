# Swift — Concurrency

> Actors, Sendable, structured concurrency, and async/await patterns.

## Do

- Use `async`/`await` over completion handlers or callbacks.
- Use actors to protect shared mutable state.
- Mark public value types as `Sendable`.
- Use structured concurrency (`TaskGroup`, `async let`) for parallel work.
- Use `@MainActor` only for UI types (Views, ViewModels).
- Prefer `nonisolated async` over `Task.detached`.
- Use `weak` captures to avoid retain cycles.

## Don't

- Don't use `DispatchQueue` or `OperationQueue` for new code.
- Don't use `Task.detached` unless fire-and-forget is required.
- Don't mark non-UI types as `@MainActor`.
- Don't use `@unchecked Sendable` without proof of thread safety.
- Don't access mutable state across actors without proper isolation.
- Don't use `nonisolated(unsafe)` without manual synchronization.

## Template

```swift
actor ImageCache {
    private var store: [URL: Image] = [:]

    func image(for url: URL) -> Image? {
        store[url]
    }

    func store(_ image: Image, for url: URL) {
        store[url] = image
    }
}

@MainActor final class ViewModel {
    private let cache: ImageCache

    init(cache: ImageCache) {
        self.cache = cache
    }

    func load(url: URL) async -> Image? {
        if let cached = await cache.image(for: url) {
            return cached
        }
        let image = try? await downloadImage(from: url)
        if let image {
            await cache.store(image, for: url)
        }
        return image
    }
}
```