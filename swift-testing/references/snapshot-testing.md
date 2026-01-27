# Snapshot Testing

Snapshot testing catches visual regressions by comparing rendered UI against recorded baselines.

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
import SwiftUI
@testable import DesignSystem

@Suite("PRCelebrationToast Snapshots")
struct PRCelebrationToastSnapshotTests {

    @Test("renders correctly for new PR")
    func newPRLayout() {
        let record = PersonalRecord.fixture(liftType: .snatch, weight: 120.0)
        let toast = PRCelebrationToast(
            newPR: record,
            quote: "New personal best!"
        )

        assertSnapshot(
            of: toast,
            as: .image(layout: .device(config: .iPhone15Pro))
        )
    }
}
```

## Parameterized Snapshots

Test multiple configurations:

```swift
@Test("renders correctly for different lift types", arguments: LiftType.allCases)
func differentLiftTypes(liftType: LiftType) {
    let record = PersonalRecord.fixture(liftType: liftType, weight: 100.0)
    let toast = PRCelebrationToast(newPR: record, quote: "Great lift!")

    assertSnapshot(
        of: toast,
        as: .image(layout: .sizeThatFits),
        named: "\(liftType)"
    )
}
```

## Layout Options

```swift
// Device-specific
.image(layout: .device(config: .iPhone15Pro))
.image(layout: .device(config: .iPadPro12_9))

// Size that fits content
.image(layout: .sizeThatFits)

// Fixed size
.image(layout: .fixed(width: 300, height: 200))
```

## Recording Mode

First run records baselines. To re-record:

```swift
// Re-record all snapshots in this test
assertSnapshot(of: view, as: .image, record: true)
```

Or use environment variable:

```bash
SNAPSHOT_TESTING_RECORD=1 swift test
```

## Multiple Device Sizes

```swift
@Test("adapts to different screen sizes")
func multipleDevices() {
    let view = SettingsScreen()

    let devices: [(String, ViewImageConfig)] = [
        ("iPhoneSE", .iPhoneSe),
        ("iPhone15Pro", .iPhone15Pro),
        ("iPadPro", .iPadPro12_9),
    ]

    for (name, config) in devices {
        assertSnapshot(
            of: view,
            as: .image(layout: .device(config: config)),
            named: name
        )
    }
}
```

## Dark Mode Testing

```swift
@Test("renders correctly in dark mode")
func darkModeAppearance() {
    let view = SettingsRow(title: "Notifications", isEnabled: true)
        .preferredColorScheme(.dark)

    assertSnapshot(
        of: view,
        as: .image(layout: .sizeThatFits),
        named: "dark"
    )
}
```

## Accessibility Testing

```swift
@Test("supports Dynamic Type")
func dynamicTypeSupport() {
    let sizes: [ContentSizeCategory] = [.small, .large, .accessibilityExtraExtraLarge]

    for size in sizes {
        let view = SettingsRow(title: "Notifications", isEnabled: true)
            .environment(\.sizeCategory, size)

        assertSnapshot(
            of: view,
            as: .image(layout: .sizeThatFits),
            named: "\(size)"
        )
    }
}
```

## Best Practices

### Consistency

- **Same simulator**: Record all snapshots on the same device/simulator
- **Match CI**: Use same configuration as CI pipeline
- **Commit baselines**: Store reference images in version control

### Organization

```
Tests/
└── DesignSystemTests/
    ├── Snapshots/
    │   ├── PRCelebrationToastSnapshotTests.swift
    │   └── __Snapshots__/           # Generated baseline images
    │       └── PRCelebrationToastSnapshotTests/
    │           ├── newPRLayout.png
    │           └── differentLiftTypes-snatch.png
```

### Review Process

1. Run tests locally before PR
2. Review snapshot diffs carefully
3. Re-record intentional changes
4. Commit new baselines with code changes

## Troubleshooting

### Flaky Tests

```swift
// Add precision tolerance for anti-aliasing differences
assertSnapshot(
    of: view,
    as: .image(precision: 0.99)
)
```

### CI Failures

- Ensure CI uses same simulator version
- Consider using `perceptualPrecision` for minor rendering differences
- Document expected simulator in README

### Large Files

```swift
// Use smaller scale for large views
assertSnapshot(
    of: view,
    as: .image(layout: .fixed(width: 375, height: 812), scale: 1)
)
```
