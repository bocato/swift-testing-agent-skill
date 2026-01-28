# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-01-28

### Added
- Gemini CLI support with installation instructions in README.md
- Gemini CLI documentation link in CONTRIBUTING.md

## [1.1.0] - 2025-01-28

### Added
- Dump snapshot testing reference guide (`references/dump-snapshot-testing.md`)
  - Text-based snapshot testing using SnapshotTesting's `.dump` and `.customDump` strategies
  - Guidance on deterministic values and date handling
  - Parameterized snapshot testing patterns
  - Best practices for data structure verification

## [1.0.0] - 2025-01-27

### Added
- Initial release of Swift Testing Agent Skill
- Core skill definition (`swift-testing/SKILL.md`) with:
  - Agent behavior contract
  - Quick decision tree for testing guidance
  - Triage-first playbook for common errors
  - Core Swift Testing syntax examples
  - Arrange-Act-Assert pattern guidance
  - F.I.R.S.T. principles reference
  - Test double quick reference (Martin Fowler's taxonomy)
  - Test pyramid guidance
- Reference documentation:
  - `test-organization.md` - Suites, tags, traits, parallel execution
  - `parameterized-tests.md` - Testing multiple inputs efficiently
  - `async-testing.md` - Async patterns, confirmation, timeouts
  - `migration-xctest.md` - XCTest to Swift Testing migration guide
  - `test-doubles.md` - Complete taxonomy (Dummy, Fake, Stub, Spy, SpyingStub, Mock)
  - `fixtures.md` - Fixture patterns and placement
  - `integration-testing.md` - Module interaction testing
  - `snapshot-testing.md` - UI regression testing with SnapshotTesting library
  - `_index.md` - Quick navigation for all topics
- Installation support for:
  - skills.sh (recommended)
  - GitHub Copilot (project and personal)
  - Claude Code Plugin (personal and project configuration)
  - Manual installation
- Claude Code marketplace configuration (`.claude-plugin/marketplace.json`)
- Contributing guidelines with Agent Skills format requirements
- MIT License

[1.2.0]: https://github.com/bocato/swift-testing-agent-skill/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/bocato/swift-testing-agent-skill/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/bocato/swift-testing-agent-skill/releases/tag/v1.0.0
