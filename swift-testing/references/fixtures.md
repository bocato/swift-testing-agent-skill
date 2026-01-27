# Fixtures

Fixtures are factory methods that simplify creating test objects with sensible defaults.

## Fixture Placement

Place fixtures **close to the model**, not in test targets:

```swift
// In Sources/Models/PersonalRecord.swift

public struct PersonalRecord: Equatable, Sendable {
    public let id: UUID
    public let liftType: LiftType
    public let weight: Double
    public let reps: Int
    public let date: Date
    public let isPersonalBest: Bool

    public init(
        id: UUID,
        liftType: LiftType,
        weight: Double,
        reps: Int,
        date: Date,
        isPersonalBest: Bool = false
    ) {
        self.id = id
        self.liftType = liftType
        self.weight = weight
        self.reps = reps
        self.date = date
        self.isPersonalBest = isPersonalBest
    }
}

// Fixture lives alongside the model
#if DEBUG
extension PersonalRecord {
    public static func fixture(
        id: UUID = UUID(),
        liftType: LiftType = .snatch,
        weight: Double = 100.0,
        reps: Int = 1,
        date: Date = Date(),
        isPersonalBest: Bool = false
    ) -> PersonalRecord {
        PersonalRecord(
            id: id,
            liftType: liftType,
            weight: weight,
            reps: reps,
            date: date,
            isPersonalBest: isPersonalBest
        )
    }
}
#endif
```

## Benefits

1. **Tests show relevant data**: Only specify properties that matter
2. **Reduces boilerplate**: Defaults for unimportant properties
3. **Consistent test data**: Same defaults across suite
4. **Auto-available**: No imports beyond model's module
5. **Zero production overhead**: `#if DEBUG` strips from release

## Usage Patterns

### Minimal Specification

```swift
@Test("returns nickname when present")
func returnsNicknameWhenPresent() {
    // Only specify what matters for THIS test
    let user = User.fixture(nickname: "Johnny")
    let sut = ProfileViewModel(user: user)

    let displayName = sut.getUserName()

    #expect(displayName == "Johnny")
}
```

### Multiple Fixtures

```swift
@Test("sorts records by date")
func sortsRecordsByDate() {
    let oldRecord = PersonalRecord.fixture(
        date: Date().addingTimeInterval(-86400)
    )
    let newRecord = PersonalRecord.fixture(
        date: Date()
    )

    let sorted = sut.sort([oldRecord, newRecord])

    #expect(sorted.first?.id == newRecord.id)
}
```

### Fixture Collections

```swift
#if DEBUG
extension PersonalRecord {
    public static func fixtures(count: Int) -> [PersonalRecord] {
        (0..<count).map { _ in .fixture() }
    }

    public static var sampleCollection: [PersonalRecord] {
        [
            .fixture(liftType: .snatch, weight: 80),
            .fixture(liftType: .cleanAndJerk, weight: 100),
            .fixture(liftType: .squat, weight: 150),
        ]
    }
}
#endif
```

### Nested Fixtures

```swift
#if DEBUG
extension User {
    public static func fixture(
        id: UUID = UUID(),
        profile: Profile = .fixture(),
        settings: Settings = .fixture()
    ) -> User {
        User(id: id, profile: profile, settings: settings)
    }
}

extension Profile {
    public static func fixture(
        name: String = "Test User",
        email: String = "test@example.com"
    ) -> Profile {
        Profile(name: name, email: email)
    }
}
#endif
```

## Fixture Guidelines

### Do

- Provide sensible defaults for all properties
- Make defaults representative of typical data
- Use `#if DEBUG` to exclude from production
- Make fixture method `public static`
- Mirror initializer parameter order

### Don't

- Use random values (breaks repeatability)
- Include fixtures in production builds
- Create fixtures in test targets (harder to share)
- Use dates like `Date()` without allowing override

## Date Handling

```swift
#if DEBUG
extension PersonalRecord {
    public static func fixture(
        // Use a fixed reference date, not Date()
        date: Date = Date(timeIntervalSince1970: 1704067200)  // 2024-01-01
    ) -> PersonalRecord {
        // ...
    }
}
#endif
```

Or use a test clock dependency:

```swift
@Dependency(\.date) var date

// In test
let fixedDate = Date(timeIntervalSince1970: 1704067200)
withDependencies {
    $0.date = .constant(fixedDate)
} operation: {
    // Tests use fixed date
}
```
