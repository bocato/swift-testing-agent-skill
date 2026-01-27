# Migration from XCTest to Swift Testing

How to migrate existing XCTest tests to Swift Testing.

## Quick Reference

| XCTest | Swift Testing |
|--------|---------------|
| `class FooTests: XCTestCase` | `@Suite struct FooTests` |
| `func testFoo()` | `@Test func foo()` |
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertTrue(x)` | `#expect(x)` |
| `XCTAssertFalse(x)` | `#expect(!x)` |
| `XCTAssertNil(x)` | `#expect(x == nil)` |
| `XCTAssertNotNil(x)` | `#expect(x != nil)` or `try #require(x)` |
| `XCTAssertThrowsError` | `#expect(throws:)` |
| `XCTFail("message")` | `Issue.record("message")` |
| `XCTSkip("reason")` | Test trait `.disabled("reason")` |
| `setUp()` | `init()` |
| `tearDown()` | `deinit` |

## Basic Test Migration

### Before (XCTest)

```swift
import XCTest

class UserTests: XCTestCase {
    func testUserCreation() {
        let user = User(name: "Alice")
        XCTAssertEqual(user.name, "Alice")
        XCTAssertNotNil(user.id)
    }
}
```

### After (Swift Testing)

```swift
import Testing

@Suite struct UserTests {
    @Test func userCreation() throws {
        let user = User(name: "Alice")
        #expect(user.name == "Alice")
        let id = try #require(user.id)
        #expect(!id.isEmpty)
    }
}
```

## Assertion Migration

### Equality

```swift
// XCTest
XCTAssertEqual(result, expected)
XCTAssertEqual(result, expected, "Custom message")

// Swift Testing
#expect(result == expected)
#expect(result == expected, "Custom message")
```

### Boolean

```swift
// XCTest
XCTAssertTrue(condition)
XCTAssertFalse(condition)

// Swift Testing
#expect(condition)
#expect(!condition)
```

### Nil Checks

```swift
// XCTest
XCTAssertNil(optional)
XCTAssertNotNil(optional)

// Swift Testing
#expect(optional == nil)
#expect(optional != nil)

// Or use #require for unwrapping
let value = try #require(optional)
```

### Error Testing

```swift
// XCTest
XCTAssertThrowsError(try riskyOperation()) { error in
    XCTAssertEqual(error as? MyError, .specific)
}

XCTAssertNoThrow(try safeOperation())

// Swift Testing
#expect(throws: MyError.specific) {
    try riskyOperation()
}

#expect(throws: Never.self) {
    try safeOperation()
}
```

## Setup and Teardown

### Before (XCTest)

```swift
class DatabaseTests: XCTestCase {
    var database: Database!

    override func setUp() {
        super.setUp()
        database = Database.inMemory()
    }

    override func tearDown() {
        database.close()
        database = nil
        super.tearDown()
    }

    func testInsert() {
        database.insert(record)
    }
}
```

### After (Swift Testing)

```swift
@Suite struct DatabaseTests {
    let database: Database

    init() throws {
        database = try Database.inMemory()
    }

    @Test func insert() {
        database.insert(record)
    }
}
```

## Async Tests

### Before (XCTest)

```swift
func testAsyncFetch() async throws {
    let result = try await service.fetch()
    XCTAssertFalse(result.isEmpty)
}

// Or with expectations
func testAsyncWithExpectation() {
    let expectation = XCTestExpectation(description: "Fetch")

    service.fetch { result in
        XCTAssertNotNil(result)
        expectation.fulfill()
    }

    wait(for: [expectation], timeout: 5)
}
```

### After (Swift Testing)

```swift
@Test func asyncFetch() async throws {
    let result = try await service.fetch()
    #expect(!result.isEmpty)
}

// For callbacks, use confirmation
@Test func asyncWithConfirmation() async {
    await confirmation { confirm in
        service.fetch { result in
            #expect(result != nil)
            confirm()
        }
    }
}
```

## Parameterized Tests

### Before (XCTest)

```swift
func testValidEmails() {
    let validEmails = ["a@b.com", "test@example.org"]
    for email in validEmails {
        XCTAssertTrue(EmailValidator.isValid(email), "\(email) should be valid")
    }
}
```

### After (Swift Testing)

```swift
@Test(arguments: ["a@b.com", "test@example.org"])
func validEmail(email: String) {
    #expect(EmailValidator.isValid(email))
}
```

## Skipping Tests

### Before (XCTest)

```swift
func testPlatformSpecific() throws {
    #if !os(iOS)
    throw XCTSkip("iOS only")
    #endif
    // Test code
}
```

### After (Swift Testing)

```swift
@Test(.enabled(if: Platform.isIOS, "iOS only"))
func platformSpecific() {
    // Test code
}

// Or
@Test(.disabled("Not implemented yet"))
func futureFeature() { }
```

## Test Organization

### Before (XCTest)

```swift
class CartTests: XCTestCase {
    // Tests grouped by comments
    // MARK: - Adding Items
    func testAddSingleItem() { }
    func testAddMultipleItems() { }

    // MARK: - Removing Items
    func testRemoveItem() { }
}
```

### After (Swift Testing)

```swift
@Suite("Cart")
struct CartTests {
    @Suite("Adding Items")
    struct AddingTests {
        @Test func singleItem() { }
        @Test func multipleItems() { }
    }

    @Suite("Removing Items")
    struct RemovingTests {
        @Test func removeItem() { }
    }
}
```

## Migration Strategy

1. **Start with leaf tests**: Tests that don't depend on XCTest infrastructure
2. **Migrate one file at a time**: Keep changes reviewable
3. **Run both simultaneously**: XCTest and Swift Testing can coexist
4. **Update CI configuration**: Ensure both are run during migration
5. **Remove XCTest after full migration**: Clean up imports and dependencies

## Coexistence

You can have both frameworks in the same project:

```swift
// XCTest (existing)
import XCTest
class OldTests: XCTestCase { }

// Swift Testing (new)
import Testing
@Suite struct NewTests { }
```

Both will be discovered and run by `swift test`.
