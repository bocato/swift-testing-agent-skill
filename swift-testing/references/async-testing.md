# Async Testing

Testing asynchronous code with Swift Testing.

## Basic Async Tests

```swift
@Test func asyncOperation() async {
    let result = await service.fetch()
    #expect(result.isValid)
}

@Test func asyncThrowingOperation() async throws {
    let data = try await service.fetchData()
    #expect(!data.isEmpty)
}
```

## Testing Async Sequences

```swift
@Test func asyncSequence() async {
    let sequence = Counter().values
    var collected: [Int] = []

    for await value in sequence.prefix(3) {
        collected.append(value)
    }

    #expect(collected == [1, 2, 3])
}
```

## Confirmation (for callbacks/delegates)

Use `confirmation` when testing delegate patterns or callbacks:

```swift
@Test func delegateCallback() async {
    await confirmation { confirm in
        let delegate = TestDelegate(onComplete: {
            confirm()
        })

        service.delegate = delegate
        service.performAction()
    }
}
```

### Multiple Confirmations

```swift
@Test func multipleCallbacks() async {
    await confirmation(expectedCount: 3) { confirm in
        let observer = Observer(onEvent: { _ in
            confirm()
        })

        emitter.emit(.event1)
        emitter.emit(.event2)
        emitter.emit(.event3)
    }
}
```

### Optional Confirmation

```swift
@Test func optionalCallback() async {
    await confirmation(expectedCount: 0...1) { confirm in
        // May or may not be called
        service.maybeNotify { confirm() }
    }
}
```

## Testing Timeouts

### Built-in Time Limit

```swift
@Test(.timeLimit(.seconds(5)))
func mustCompleteQuickly() async {
    await slowOperation()
}
```

### Custom Timeout

```swift
@Test func withCustomTimeout() async throws {
    try await withTimeout(seconds: 2) {
        try await service.fetch()
    }
}

func withTimeout<T>(
    seconds: Double,
    operation: @escaping () async throws -> T
) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask { try await operation() }
        group.addTask {
            try await Task.sleep(for: .seconds(seconds))
            throw TimeoutError()
        }
        let result = try await group.next()!
        group.cancelAll()
        return result
    }
}
```

## Testing Cancellation

```swift
@Test func cancellation() async {
    let task = Task {
        try await longRunningOperation()
    }

    // Give it time to start
    try? await Task.sleep(for: .milliseconds(100))

    task.cancel()

    do {
        _ = try await task.value
        Issue.record("Should have been cancelled")
    } catch is CancellationError {
        // Expected
    }
}
```

## Testing Actors

```swift
actor Counter {
    var value = 0
    func increment() { value += 1 }
}

@Test func actorState() async {
    let counter = Counter()

    await counter.increment()
    await counter.increment()

    let value = await counter.value
    #expect(value == 2)
}
```

## Testing MainActor Code

```swift
@MainActor
class ViewModel {
    var items: [Item] = []
    func load() async {
        items = await fetchItems()
    }
}

@Test @MainActor
func viewModelLoading() async {
    let viewModel = ViewModel()
    await viewModel.load()
    #expect(!viewModel.items.isEmpty)
}
```

## Mocking Async Dependencies

```swift
struct APIClient {
    var fetch: @Sendable (URL) async throws -> Data
}

@Test func withMockedClient() async throws {
    let mockData = "test".data(using: .utf8)!
    let client = APIClient(
        fetch: { _ in mockData }
    )

    let service = Service(client: client)
    let result = try await service.getData()

    #expect(result == mockData)
}
```

## Testing Debounced Operations

```swift
@Test func debounce() async throws {
    let debouncer = Debouncer(delay: .milliseconds(100))
    var callCount = 0

    // Rapid calls
    for _ in 1...5 {
        await debouncer.submit {
            callCount += 1
        }
    }

    // Wait for debounce
    try await Task.sleep(for: .milliseconds(150))

    #expect(callCount == 1)  // Only last call executed
}
```

## Testing Retry Logic

```swift
@Test func retryOnFailure() async throws {
    var attempts = 0
    let service = Service(
        fetch: {
            attempts += 1
            if attempts < 3 {
                throw NetworkError.timeout
            }
            return Data()
        }
    )

    let result = try await service.fetchWithRetry(maxAttempts: 3)

    #expect(attempts == 3)
    #expect(result != nil)
}
```

## Best Practices

1. **Use async/await directly**: No need for expectations/wait
2. **Use confirmation for callbacks**: When testing delegate patterns
3. **Set time limits**: Prevent hanging tests
4. **Test cancellation**: Ensure proper cleanup
5. **Mock async dependencies**: Use closures for testability
6. **Run on correct actor**: Use @MainActor for UI tests
