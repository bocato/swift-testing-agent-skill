# Integration Testing

Integration tests verify module interactions - the 15% of your testing pyramid.

## When to Write Integration Tests

- Testing component boundaries (use case -> repository -> storage)
- Verifying workflows across multiple components
- Testing real implementations with in-memory storage
- Validating data flows end-to-end within a module

## Basic Structure

```swift
import Testing
@testable import PersonalRecordsCore

@Suite("PersonalRecords Integration Tests")
struct PersonalRecordsIntegrationTests {

    @Test("save and retrieve workflow completes successfully")
    func saveAndRetrieveWorkflow() async throws {
        // Use real implementations with in-memory storage
        let storage = InMemoryStorageService()
        let repository = PersonalRecordsRepository(storage: storage)
        let saveUseCase = SavePRUseCase(repository: repository)
        let loadUseCase = LoadPRUseCase(repository: repository)

        let record = PersonalRecord.fixture(weight: 120.0)

        // Save
        try await saveUseCase.dispatch(record)

        // Retrieve and verify
        let loaded = try await loadUseCase.dispatch()

        #expect(loaded.count == 1)
        #expect(loaded.first?.weight == 120.0)
    }
}
```

## In-Memory Implementations

Create fakes for external dependencies:

```swift
final class InMemoryStorageService: StorageServiceProtocol {
    private var storage: [String: Data] = [:]
    private let lock = NSLock()

    func save(_ data: Data, forKey key: String) async throws {
        lock.lock()
        defer { lock.unlock() }
        storage[key] = data
    }

    func load(forKey key: String) async throws -> Data? {
        lock.lock()
        defer { lock.unlock() }
        return storage[key]
    }

    func delete(forKey key: String) async throws {
        lock.lock()
        defer { lock.unlock() }
        storage.removeValue(forKey: key)
    }
}
```

## Testing Workflows

### Multi-Step Operations

```swift
@Test("complete user registration workflow")
func registrationWorkflow() async throws {
    // Setup real components with test dependencies
    let userStorage = InMemoryUserStorage()
    let tokenStorage = InMemoryTokenStorage()
    let userService = UserService(storage: userStorage)
    let authService = AuthService(tokenStorage: tokenStorage)

    let sut = RegistrationWorker(
        userService: userService,
        authService: authService
    )

    // Execute workflow
    let result = try await sut.register(
        username: "testuser",
        password: "SecurePass123"
    )

    // Verify end state
    #expect(result.isSuccess)
    #expect(userStorage.users.contains { $0.username == "testuser" })
    #expect(tokenStorage.hasToken)
}
```

### Error Propagation

```swift
@Test("propagates storage errors through use case")
func errorPropagation() async throws {
    let failingStorage = FailingStorageService()
    let repository = PersonalRecordsRepository(storage: failingStorage)
    let sut = LoadPRUseCase(repository: repository)

    #expect(throws: PRError.storageUnavailable) {
        try await sut.dispatch()
    }
}
```

## Tagging Integration Tests

Use tags to filter tests:

```swift
extension Tag {
    @Tag static var integration: Self
}

@Suite("Integration Tests", .tags(.integration))
struct PersonalRecordsIntegrationTests {
    // ...
}
```

Run only integration tests:

```bash
swift test --filter integration
```

## Integration Test Guidelines

### Do

- Test component boundaries
- Use in-memory implementations for storage
- Test complete workflows
- Verify data flows correctly
- Tag tests for filtering

### Don't

- Test UI (use snapshot/UI tests)
- Use real network calls
- Use real databases
- Test third-party libraries
- Write too many (15% of pyramid)

## Test Organization

```
Tests/
└── PersonalRecordsCoreTests/
    ├── Unit/
    │   ├── UseCases/
    │   └── Repositories/
    ├── Integration/
    │   ├── WorkflowTests.swift
    │   └── DataFlowTests.swift
    └── Helpers/
        └── InMemoryStorage.swift
```

## Performance Considerations

Integration tests are slower than unit tests:

```swift
@Test("bulk import performance", .timeLimit(.minutes(1)))
func bulkImportPerformance() async throws {
    let storage = InMemoryStorageService()
    let repository = PersonalRecordsRepository(storage: storage)
    let sut = BulkImportUseCase(repository: repository)

    let records = PersonalRecord.fixtures(count: 1000)

    try await sut.dispatch(records)

    #expect(storage.count == 1000)
}
```
