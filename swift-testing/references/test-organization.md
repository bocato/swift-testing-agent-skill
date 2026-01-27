# Test Organization

Organizing tests with suites, tags, and traits in Swift Testing.

## Test Suites

Group related tests:

```swift
@Suite("User Management")
struct UserTests {
    @Test func createUser() { }
    @Test func deleteUser() { }
}

@Suite("Authentication")
struct AuthTests {
    @Test func login() { }
    @Test func logout() { }
}
```

### Nested Suites

```swift
@Suite("Shopping Cart")
struct CartTests {
    @Suite("Adding Items")
    struct AddTests {
        @Test func addSingleItem() { }
        @Test func addMultipleItems() { }
    }

    @Suite("Removing Items")
    struct RemoveTests {
        @Test func removeSingleItem() { }
        @Test func clearCart() { }
    }
}
```

## Tags

Categorize tests for selective running:

```swift
extension Tag {
    @Tag static var integration: Self
    @Tag static var slow: Self
    @Tag static var network: Self
}

@Test(.tags(.integration))
func databaseIntegration() { }

@Test(.tags(.slow, .network))
func networkRequest() { }
```

### Running Tagged Tests

```bash
# Run only integration tests
swift test --filter .tags:integration

# Exclude slow tests
swift test --skip .tags:slow
```

## Traits

### Disabled Tests

```swift
@Test(.disabled("Waiting for API fix"))
func brokenTest() { }

@Test(.disabled(if: isCI, "Flaky on CI"))
func sometimesFlaky() { }
```

### Time Limits

```swift
@Test(.timeLimit(.minutes(1)))
func slowTest() async { }
```

### Bug References

```swift
@Test(.bug("https://github.com/org/repo/issues/123"))
func testWithKnownBug() { }
```

### Custom Traits

```swift
@Test(.enabled(if: ProcessInfo.processInfo.environment["RUN_SLOW_TESTS"] != nil))
func conditionalTest() { }
```

## Setup and Teardown

### Per-Test Setup

```swift
@Suite struct DatabaseTests {
    var database: Database

    init() throws {
        // Runs before each test
        database = try Database.inMemory()
    }

    @Test func insertRecord() {
        // database is fresh for each test
    }
}
```

### Suite-Level Setup

```swift
@Suite struct ServerTests {
    static var server: TestServer!

    init() async throws {
        // Per-test setup
    }

    @Test func request() async { }
}
```

## Test Organization Best Practices

### File Structure

```
Tests/
├── UnitTests/
│   ├── Models/
│   │   ├── UserTests.swift
│   │   └── ProductTests.swift
│   ├── Services/
│   │   ├── AuthServiceTests.swift
│   │   └── CartServiceTests.swift
│   └── Utilities/
│       └── FormatterTests.swift
├── IntegrationTests/
│   ├── DatabaseTests.swift
│   └── APITests.swift
└── TestHelpers/
    ├── Fixtures.swift
    └── Mocks.swift
```

### Naming Files

- Name test files after the type they test: `UserTests.swift` for `User`
- Use `Tests` suffix for test files

### Organizing Within Files

```swift
@Suite("User")
struct UserTests {
    // MARK: - Initialization

    @Test func initWithValidData() { }
    @Test func initWithInvalidData() { }

    // MARK: - Properties

    @Test func fullName() { }
    @Test func age() { }

    // MARK: - Methods

    @Test func update() { }
    @Test func delete() { }
}
```

## Test Discovery

Swift Testing automatically discovers:
- Functions marked with `@Test`
- Types marked with `@Suite`
- Nested suites and tests

No need to:
- Inherit from XCTestCase
- Prefix with "test"
- Register tests manually

## Parallel Execution

Tests run in parallel by default:

```swift
@Suite(.serialized)  // Run tests in this suite serially
struct SerialTests {
    @Test func first() { }
    @Test func second() { }
}
```

## FIRST Principles

Structure tests to be:

- **Fast**: Run quickly
- **Isolated**: No dependencies between tests
- **Repeatable**: Same result every time
- **Self-validating**: Clear pass/fail
- **Timely**: Written with or before code

```swift
@Test func fastAndIsolated() {
    // Uses in-memory database, not real one
    let db = Database.inMemory()

    // Self-contained data
    let user = User.fixture()

    // Clear assertion
    #expect(db.save(user))
}
```
