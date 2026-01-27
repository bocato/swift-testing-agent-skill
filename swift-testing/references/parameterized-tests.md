# Parameterized Tests

Test multiple inputs with a single test function.

## Basic Parameterization

```swift
@Test(arguments: [1, 2, 3, 4, 5])
func isPositive(number: Int) {
    #expect(number > 0)
}
```

## Multiple Arguments

### Using zip (Paired)

```swift
@Test(arguments: zip(
    ["hello", "world", "test"],
    [5, 5, 4]
))
func stringLength(string: String, expectedLength: Int) {
    #expect(string.count == expectedLength)
}
```

### Cartesian Product (All Combinations)

```swift
@Test(arguments: [1, 2], ["a", "b"])
func combinations(number: Int, letter: String) {
    // Runs 4 times: (1,a), (1,b), (2,a), (2,b)
    #expect(!"\(number)\(letter)".isEmpty)
}
```

## Custom Test Cases

```swift
struct ValidationTestCase {
    let input: String
    let isValid: Bool
    let description: String
}

extension ValidationTestCase: CustomTestStringConvertible {
    var testDescription: String { description }
}

let validationCases = [
    ValidationTestCase(input: "valid@email.com", isValid: true, description: "valid email"),
    ValidationTestCase(input: "invalid", isValid: false, description: "missing @"),
    ValidationTestCase(input: "", isValid: false, description: "empty string"),
]

@Test(arguments: validationCases)
func validateEmail(testCase: ValidationTestCase) {
    let result = EmailValidator.validate(testCase.input)
    #expect(result == testCase.isValid)
}
```

## Enum Cases

```swift
enum Environment: CaseIterable {
    case development, staging, production
}

@Test(arguments: Environment.allCases)
func configurationLoads(environment: Environment) {
    let config = Configuration(environment: environment)
    #expect(config.isValid)
}
```

## Ranges

```swift
@Test(arguments: 1...100)
func withinRange(value: Int) {
    #expect(value >= 1 && value <= 100)
}
```

## Collection of Tuples

```swift
@Test(arguments: [
    ("2024-01-15", true),
    ("invalid", false),
    ("2024-13-45", false),
])
func dateValidation(dateString: String, shouldBeValid: Bool) {
    let isValid = DateValidator.validate(dateString)
    #expect(isValid == shouldBeValid)
}
```

## Avoiding Cartesian Explosion

Be careful with multiple argument lists:

```swift
// WARNING: This runs 1000 times (10 x 10 x 10)
@Test(arguments: 1...10, 1...10, 1...10)
func tooManyTests(a: Int, b: Int, c: Int) { }

// BETTER: Use zip for paired testing
@Test(arguments: zip(zip(inputs1, inputs2), expectedResults))
func pairedTest(inputs: ((Int, Int), Int)) { }
```

## Filtering Arguments

```swift
let testCases = (1...100).filter { $0 % 10 == 0 }

@Test(arguments: testCases)
func multiplesOfTen(value: Int) {
    #expect(value % 10 == 0)
}
```

## Complex Test Data

```swift
struct APITestCase: Sendable {
    let endpoint: String
    let method: HTTPMethod
    let expectedStatus: Int
    let body: Data?

    static let cases: [APITestCase] = [
        APITestCase(endpoint: "/users", method: .get, expectedStatus: 200, body: nil),
        APITestCase(endpoint: "/users", method: .post, expectedStatus: 201, body: validUserData),
        APITestCase(endpoint: "/users/999", method: .get, expectedStatus: 404, body: nil),
    ]
}

@Test(arguments: APITestCase.cases)
func apiEndpoint(testCase: APITestCase) async throws {
    let response = try await client.request(
        endpoint: testCase.endpoint,
        method: testCase.method,
        body: testCase.body
    )
    #expect(response.statusCode == testCase.expectedStatus)
}
```

## Best Practices

1. **Keep test cases focused**: Each should test one thing
2. **Use descriptive names**: Implement `CustomTestStringConvertible`
3. **Avoid Cartesian products**: Use zip for paired data
4. **Group related cases**: Create structs for complex scenarios
5. **Make test data Sendable**: Required for parallel execution

```swift
// GOOD: Clear, paired test cases
@Test(arguments: zip(["a", "ab", "abc"], [1, 2, 3]))
func stringLength(string: String, expected: Int) {
    #expect(string.count == expected)
}

// BAD: Cartesian product, unclear intent
@Test(arguments: ["a", "ab", "abc"], [1, 2, 3])
func unclearTest(string: String, number: Int) { }
```
