# Swift — Error Handling

> Typed throws, Result type, do/catch patterns, and error propagation.

## Do

- Use typed throws for domain-specific errors: `throws(MyError)`.
- Use `Result<T, Error>` for synchronous APIs that may fail.
- Use `do`/`catch` with specific error matching.
- Create descriptive error enums with associated values when needed.
- Propagate errors up the call stack when recovery isn't possible.
- Use `try?` only when failure is non-critical and can be ignored.

## Don't

- Don't use `try!` unless the failure is truly impossible.
- Don't catch `Error` without matching specifics (use `let error as MyError`).
- Don't throw generic `NSError` when a typed error exists.
- Don't silently swallow errors with empty `catch` blocks.
- Don't use `Result` when async/await is more appropriate.

## Template

```swift
enum NetworkError: Error {
    case invalidURL
    case notFound
    case serverError(statusCode: Int)
}

func fetchData(from url: URL) async throws(NetworkError) -> Data {
    let (data, response) = try await URLSession.shared.data(from: url)
    guard let httpResponse = response as? HTTPURLResponse else {
        throw .invalidURL
    }
    guard (200...299).contains(httpResponse.statusCode) else {
        throw .serverError(statusCode: httpResponse.statusCode)
    }
    return data
}

// Usage
do {
    let data = try await fetchData(from: url)
} catch {
    print("Failed: \(error)")
}
```