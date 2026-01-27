# Reference Index

Quick navigation for Swift Testing topics.

## Framework Basics

| File | Description |
|------|-------------|
| `test-organization.md` | Suites, tags, traits, parallel execution |
| `parameterized-tests.md` | Testing multiple inputs efficiently |
| `async-testing.md` | Async patterns, confirmation, timeouts |
| `migration-xctest.md` | XCTest to Swift Testing migration |

## Test Infrastructure

| File | Description |
|------|-------------|
| `test-doubles.md` | Complete taxonomy: Dummy, Fake, Stub, Spy, SpyingStub, Mock |
| `fixtures.md` | Fixture patterns, placement, and best practices |
| `integration-testing.md` | Module interaction testing patterns |
| `snapshot-testing.md` | UI regression testing with SnapshotTesting |
| `dump-snapshot-testing.md` | Text-based snapshot testing for data structures |

## Quick Links by Problem

### "I need to..."

- **Start using Swift Testing** -> `test-organization.md`
- **Test multiple inputs** -> `parameterized-tests.md`
- **Test async code** -> `async-testing.md`
- **Migrate from XCTest** -> `migration-xctest.md`
- **Create test doubles** -> `test-doubles.md`
- **Create test data** -> `fixtures.md`
- **Test module interactions** -> `integration-testing.md`
- **Test UI for regressions** -> `snapshot-testing.md`
- **Snapshot data structures** -> `dump-snapshot-testing.md`

### "I'm having issues with..."

- **Flaky tests** -> Check `fixtures.md` (date handling), `async-testing.md` (timing)
- **Slow tests** -> Check `test-doubles.md` (proper mocking), `integration-testing.md` (pyramid)
- **Test organization** -> `test-organization.md` (suites, tags)
- **XCTest syntax errors** -> `migration-xctest.md`
- **Choosing test doubles** -> `test-doubles.md` (decision table)

### "I want to learn about..."

- **F.I.R.S.T. principles** -> Main SKILL.md
- **Test pyramid** -> Main SKILL.md, `integration-testing.md`
- **Arrange-Act-Assert** -> Main SKILL.md
- **Martin Fowler's test double taxonomy** -> `test-doubles.md`

## File Statistics

| File | Description | Key Topics |
|------|-------------|------------|
| `test-organization.md` | ~180 lines | Suites, tags, traits, setup/teardown |
| `parameterized-tests.md` | ~160 lines | Arguments, zip, Cartesian products |
| `async-testing.md` | ~200 lines | Async/await, confirmation, timeouts |
| `migration-xctest.md` | ~220 lines | XCTest -> Swift Testing mapping |
| `test-doubles.md` | ~220 lines | Dummy, Fake, Stub, Spy, SpyingStub, Mock |
| `fixtures.md` | ~140 lines | Placement, patterns, date handling |
| `integration-testing.md` | ~160 lines | In-memory implementations, workflows |
| `snapshot-testing.md` | ~180 lines | SnapshotTesting setup, devices, modes |
| `dump-snapshot-testing.md` | ~200 lines | Text snapshots, deterministic values, customDump |
