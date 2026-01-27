# Swift Testing Agent Skill

Expert guidance for any AI coding tool that supports the [Agent Skills open format](https://agentskills.io/home) — Swift Testing framework, test doubles, fixtures, and testing best practices.

Based on the comprehensive testing guide from [Eduardo Bocato's AI Dotfiles](https://github.com/bocato/ai-dotfiles), distilled into actionable, concise references for agents.

## Who this is for
- Teams adopting Swift Testing framework who need modern testing patterns.
- Developers writing unit tests, integration tests, or snapshot tests.
- Anyone wanting reliable test infrastructure (mocks, stubs, spies, fixtures).
- Teams migrating from XCTest to Swift Testing.

## How to Use This Skill

### Option A: Using skills.sh (recommended)
Install this skill with a single command:
```bash
npx skills add https://github.com/bocato/swift-testing-agent-skill --skill swift-testing
```

Then use the skill in your AI agent, for example:
> Use the swift testing skill and help me write tests for the UserService class

### Option B: Claude Code Plugin

#### Personal Usage

To install this Skill for your personal use in Claude Code:

1. Add the marketplace:
   ```bash
   /plugin marketplace add bocato/swift-testing-agent-skill
   ```

2. Install the Skill:
   ```bash
   /plugin install swift-testing@swift-testing-agent-skill
   ```

#### Project Configuration

To automatically provide this Skill to everyone working in a repository, configure the repository's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "swift-testing@swift-testing-agent-skill": true
  },
  "extraKnownMarketplaces": {
    "swift-testing-agent-skill": {
      "source": {
        "source": "github",
        "repo": "bocato/swift-testing-agent-skill"
      }
    }
  }
}
```

When team members open the project, Claude Code will prompt them to install the Skill.

### Option C: Manual install
1) **Clone** this repository.
2) **Install or symlink** the `swift-testing/` folder following your tool's official skills installation docs (see links below).
3) **Use your AI tool** as usual and ask it to use the "swift-testing" skill for testing tasks.

#### Where to Save Skills

Follow your tool's official documentation, here are a few popular ones:
- **Codex:** [Where to save skills](https://developers.openai.com/codex/skills/#where-to-save-skills)
- **Claude:** [Using Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#using-skills)
- **Cursor:** [Enabling Skills](https://cursor.com/docs/context/skills#enabling-skills)

**How to verify**:

Your agent should reference the triage/playbook in `swift-testing/SKILL.md` and jump into the relevant reference file for your testing question or task.

## What This Skill Offers

This skill gives your AI coding tool comprehensive Swift Testing guidance. It can:

### Guide Your Testing Decisions
- Choose the right assertion (`#expect` vs `#require`)
- Structure tests with Arrange-Act-Assert pattern
- Apply F.I.R.S.T. principles (Fast, Isolated, Repeatable, Self-Validating, Timely)
- Follow the test pyramid (80% unit, 15% integration, 5% UI)

### Write Proper Test Doubles
- Understand Martin Fowler's test double taxonomy
- Create Dummies, Fakes, Stubs, Spies, SpyingStubs, and Mocks
- Choose between state verification and behavior verification
- Place test doubles close to interfaces with `#if DEBUG`

### Create Maintainable Test Infrastructure
- Build fixtures with sensible defaults
- Place fixtures close to models
- Handle dates deterministically
- Organize test suites with tags and traits

### Test Various Scenarios
- Write parameterized tests for multiple inputs
- Test async code with proper patterns
- Use confirmation for callback/delegate testing
- Set up snapshot testing for UI regression

### Migrate from XCTest
- Complete XCTest to Swift Testing migration guide
- Assertion mapping (XCTAssertEqual -> #expect)
- Setup/teardown patterns (setUp -> init)
- Async test migration patterns

## What Makes This Skill Different

**Practical Focus**: Based on real-world testing patterns from production codebases, focusing on what developers actually need.

**Proper Taxonomy**: Uses Martin Fowler's official test double terminology, clarifying the confusion between "mocks" and "spies" in the Swift community.

**Non-Opinionated**: Focuses on industry-standard best practices like F.I.R.S.T. principles and Arrange-Act-Assert, not architectural preferences.

**Swift Testing Ready**: Covers the modern Swift Testing framework with `@Test`, `#expect`, `#require`, and `@Suite`.

**Infrastructure Guidance**: Includes patterns for test doubles, fixtures, and their proper placement in your codebase.

## Skill Structure

```
swift-testing/
├── SKILL.md                      # Main skill file with decision trees
└── references/
    ├── _index.md                 # Quick navigation for all topics
    ├── test-organization.md      # Suites, tags, traits, parallel execution
    ├── parameterized-tests.md    # Testing multiple inputs efficiently
    ├── async-testing.md          # Async patterns, confirmation, timeouts
    ├── migration-xctest.md       # XCTest to Swift Testing migration
    ├── test-doubles.md           # Dummy, Fake, Stub, Spy, SpyingStub, Mock
    ├── fixtures.md               # Fixture patterns and placement
    ├── integration-testing.md    # Module interaction testing
    └── snapshot-testing.md       # UI regression testing
```

## Core Concepts

### Swift Testing Basics
```swift
import Testing

@Test("User can be created with valid data")
func createUser() {
    let user = User(name: "Alice", age: 30)
    #expect(user.name == "Alice")
    #expect(user.age == 30)
}
```

### Arrange-Act-Assert
```swift
@Test func calculateTotal() {
    // Arrange
    let cart = ShoppingCart()
    cart.add(Item(price: 10))

    // Act
    let total = cart.calculateTotal()

    // Assert
    #expect(total == 10)
}
```

### Test Double (SpyingStub)
```swift
#if DEBUG
final class UserRepositorySpyingStub: UserRepositoryProtocol {
    // Spy
    private(set) var savedUsers: [User] = []

    // Stub
    var usersToReturn: [User] = []

    func save(_ user: User) async throws {
        savedUsers.append(user)
    }

    func getAll() async throws -> [User] {
        usersToReturn
    }
}
#endif
```

### Fixture
```swift
#if DEBUG
extension User {
    static func fixture(
        id: UUID = UUID(),
        name: String = "Test User",
        email: String = "test@example.com"
    ) -> User {
        User(id: id, name: name, email: email)
    }
}
#endif
```

## Contributing

Contributions are welcome! This repository follows the [Agent Skills open format](https://agentskills.io/home), which has specific structural requirements.

**We strongly recommend using AI assistance for contributions:**
- Use the [skill-creator skill](https://github.com/anthropics/skills/tree/main/skills/skill-creator) with Claude to ensure proper formatting
- This helps maintain the Agent Skills format and ensures your contribution works correctly with AI agents

**Please read [CONTRIBUTING.md](CONTRIBUTING.md) for:**
- How to use the skill-creator skill for contributions
- Agent Skills format requirements
- Quality standards and best practices
- Pull request process

This skill is maintained to reflect the latest Swift Testing best practices and will be updated as the framework evolves.

## About the Author

Based on the Swift Testing guide from [Eduardo Bocato's AI Dotfiles](https://github.com/bocato/ai-dotfiles). Eduardo is a software engineer focused on iOS development and testing best practices.

## License

This skill is open-source and available under the MIT License. See [LICENSE](LICENSE) for details.
