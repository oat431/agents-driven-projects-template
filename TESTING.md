# TESTING.md — Strategy, Patterns & Coverage

<!--
  How this project tests. Agents use this to write tests that match existing patterns,
  run the right commands, and hit coverage targets.
-->

## Test Pyramid

```
       ┌──────┐
       │ E2E  │  ~5%   Critical user flows. Cypress / Playwright / Selenium.
       ├──────┤
       │ Int. │ ~20%   API + DB. Spring Boot Test / Supertest / pytest.
       ├──────┤
       │ Unit │ ~75%   Pure logic. JUnit 5 / Vitest / pytest.
       └──────┘
```

## Commands

```bash
# All tests
./mvnw test

# Unit tests only
./mvnw test -Dgroups="unit"

# Integration tests (needs Docker)
./mvnw verify -P integration

# Single test
./mvnw test -Dtest=UserServiceTest

# Watch mode (Node.js)
npm test -- --watch

# Coverage
./mvnw jacoco:report
# Report: target/site/jacoco/index.html
```

## Coverage Targets
- Overall: 80% line coverage
- Service layer: 90%+ (business logic)
- Controllers: 70% (thin, tested via integration)
- Models/DTOs: 50% (mostly data, low value)

## Unit Test Patterns

### Naming Convention
```
{ClassName}Test.java
MethodName_StateUnderTest_ExpectedBehavior
```

### Structure (AAA Pattern)
```java
@Test
void findById_WhenUserExists_ReturnsUser() {
    // Arrange
    var user = User.builder().id(UUID.randomUUID()).email("test@example.com").build();
    when(userRepository.findById(any())).thenReturn(Optional.of(user));

    // Act
    var result = userService.findById(user.getId());

    // Assert
    assertThat(result).isNotNull();
    assertThat(result.getEmail()).isEqualTo("test@example.com");
}
```

### What to Test
✅ Business logic, edge cases, error paths, validation
✅ One behavior per test — not "test everything"
❌ Framework code, getters/setters, trivial delegation

### What to Mock
✅ External services (APIs, message queues, file system)
✅ Repositories (in unit tests)
❌ The class under test
❌ Value objects / DTOs (just create real ones)

## Integration Test Patterns

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@AutoConfigureMockMvc
class UserControllerIT {

    @Test
    void createUser_WithValidData_Returns201() throws Exception {
        mockMvc.perform(post("/api/v1/users")
                .contentType(APPLICATION_JSON)
                .content("""
                    {"email": "new@example.com", "name": "Test"}
                """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.data.email").value("new@example.com"));
    }
}
```

### Integration Test Rules
- Real database (test container or dedicated test DB)
- Clean up after each test (`@Transactional` rollback or `@AfterEach` delete)
- Test the full slice: controller → service → repository → database
- Don't mock the database connection

## Test Data

### Fixtures
- **Location:** `src/test/resources/db/testdata/`
- **Format:** SQL insert scripts, JSON fixtures, or builder methods
- **Rule:** Each test sets up its own data. No shared mutable state.

### Factory Helpers
```java
// Test data builders make tests readable
var user = TestDataFactory.aUser()
    .withEmail("test@example.com")
    .withRole(Role.ADMIN)
    .build();
```

## Smoke Tests (Post-Deploy)
```bash
# Run against deployed environment
curl -s https://api.example.com/actuator/health | jq .status
# Expected: "UP"

# Critical flow
curl -s -X POST https://api.example.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"smoke@example.com","password":"***"}' | jq .success
# Expected: true
```
