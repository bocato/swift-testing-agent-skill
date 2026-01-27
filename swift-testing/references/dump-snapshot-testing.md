# Dump Snapshot Testing

Dump snapshot testing captures text-based representations of data structures, perfect for testing models, state objects, and non-visual components.

## Setup

Use [SnapshotTesting](https://github.com/pointfreeco/swift-snapshot-testing):

```swift
// Package.swift
.package(url: "https://github.com/pointfreeco/swift-snapshot-testing", from: "1.15.0")
```

## Basic Usage

```swift
import SnapshotTesting
import Testing
@testable import Domain

@Suite("PersonalRecord Snapshots")
struct PersonalRecordSnapshotTests {

    @Test("captures record structure correctly")
    func recordStructure() {
        let record = PersonalRecord.fixture(
            liftType: .snatch,
            weight: 120.0,
            date: Date(timeIntervalSince1970: 1704067200) // Fixed date
        )

        assertSnapshot(of: record, as: .dump)
    }
}
```

## When to Use Dump Snapshots

| Use Case | Why Dump Snapshots |
|----------|-------------------|
| **Data models** | Verify all properties without writing assertions for each |
| **API responses** | Catch unexpected changes in decoded structures |
| **State objects** | Track complex state transitions |
| **Transformations** | Verify mapping/conversion logic output |
| **Configuration** | Ensure settings objects are correctly constructed |

## Parameterized Dump Snapshots

Test multiple configurations:

```swift
@Test("captures different lift types", arguments: LiftType.allCases)
func liftTypeSnapshots(liftType: LiftType) {
    let record = PersonalRecord.fixture(
        liftType: liftType,
        weight: 100.0
    )

    assertSnapshot(of: record, as: .dump, named: "\(liftType)")
}
```

## Complex Object Snapshots

```swift
@Test("captures workout session state")
func workoutSessionState() {
    let session = WorkoutSession(
        id: UUID(uuidString: "550e8400-e29b-41d4-a716-446655440000")!,
        exercises: [
            Exercise.fixture(name: "Snatch", sets: 5, reps: 3),
            Exercise.fixture(name: "Clean & Jerk", sets: 4, reps: 2)
        ],
        startedAt: Date(timeIntervalSince1970: 1704067200),
        status: .inProgress
    )

    assertSnapshot(of: session, as: .dump)
}
```

## Collections and Arrays

```swift
@Test("captures record history")
func recordHistory() {
    let records = [
        PersonalRecord.fixture(liftType: .snatch, weight: 100.0),
        PersonalRecord.fixture(liftType: .snatch, weight: 105.0),
        PersonalRecord.fixture(liftType: .snatch, weight: 110.0)
    ]

    assertSnapshot(of: records, as: .dump)
}
```

## Nested Structures

```swift
@Test("captures user profile with nested data")
func userProfileSnapshot() {
    let profile = UserProfile(
        user: User.fixture(name: "Alice"),
        settings: Settings.fixture(
            notifications: true,
            theme: .dark
        ),
        recentRecords: [
            PersonalRecord.fixture(liftType: .snatch)
        ]
    )

    assertSnapshot(of: profile, as: .dump)
}
```

## Comparing Dump vs Custom Dump

SnapshotTesting provides two text strategies:

```swift
// Standard Swift dump - uses Mirror API
assertSnapshot(of: object, as: .dump)

// Custom dump - more readable output (recommended)
assertSnapshot(of: object, as: .customDump)
```

**Prefer `.customDump`** for better readability with:
- Sorted dictionary keys
- Condensed output for simple values
- Better enum representation

## Recording Mode

First run records baselines. To re-record:

```swift
// Re-record this snapshot
assertSnapshot(of: record, as: .dump, record: true)
```

Or use environment variable:

```bash
SNAPSHOT_TESTING_RECORD=1 swift test
```

## Deterministic Snapshots

Ensure consistent output by controlling variable data:

```swift
@Test("captures record with deterministic values")
func deterministicSnapshot() {
    // Use fixed UUID
    let id = UUID(uuidString: "550e8400-e29b-41d4-a716-446655440000")!

    // Use fixed date
    let date = Date(timeIntervalSince1970: 1704067200) // 2024-01-01

    let record = PersonalRecord(
        id: id,
        liftType: .snatch,
        weight: 120.0,
        date: date
    )

    assertSnapshot(of: record, as: .dump)
}
```

## Organization

```
Tests/
└── DomainTests/
    ├── Snapshots/
    │   ├── PersonalRecordSnapshotTests.swift
    │   └── __Snapshots__/
    │       └── PersonalRecordSnapshotTests/
    │           ├── recordStructure.txt
    │           └── liftTypeSnapshots-snatch.txt
```

## Best Practices

### Use Fixtures with Fixed Values

```swift
// Good - deterministic
let record = PersonalRecord.fixture(
    id: UUID(uuidString: "00000000-0000-0000-0000-000000000001")!,
    date: Date(timeIntervalSince1970: 0)
)

// Bad - non-deterministic
let record = PersonalRecord.fixture() // Random UUID, current date
```

### Name Parameterized Snapshots

```swift
// Good - clear file names
assertSnapshot(of: record, as: .dump, named: "snatch-120kg")

// Avoid - generic names
assertSnapshot(of: record, as: .dump)
```

### Review Diffs Carefully

Dump snapshots capture all properties. When reviewing:
1. Verify intentional changes
2. Catch unintended side effects
3. Update baselines only after careful review

### Combine with Unit Tests

Dump snapshots complement, not replace, unit tests:

```swift
@Test("validates and snapshots transformation")
func transformRecord() {
    let input = APIResponse.fixture()
    let output = RecordMapper.map(input)

    // Unit assertion for critical behavior
    #expect(output.weight == input.weightKg)

    // Snapshot for complete structure
    assertSnapshot(of: output, as: .dump)
}
```

## Troubleshooting

### Non-Deterministic Failures

If snapshots fail intermittently:
- Check for `UUID()` or `Date()` without fixed values
- Ensure dictionary ordering is consistent
- Use `.customDump` for sorted keys

### Large Snapshots

For objects with many properties:

```swift
// Snapshot specific parts
assertSnapshot(of: session.exercises, as: .dump, named: "exercises")
assertSnapshot(of: session.metadata, as: .dump, named: "metadata")
```

### Unreadable Output

Switch to custom dump for cleaner output:

```swift
// Before: standard dump
assertSnapshot(of: complex, as: .dump)

// After: cleaner formatting
assertSnapshot(of: complex, as: .customDump)
```
