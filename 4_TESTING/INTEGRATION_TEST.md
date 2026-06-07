# INTEGRATION_TEST.md — Service & API Boundary Tests

<!--
  Tests that cross boundaries: API → DB, Service → External API, etc.
  Slower than unit tests. More valuable for catching real bugs.
-->

---

## 🔧 BACKEND

### Spring Boot + TestContainers

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@AutoConfigureMockMvc
@Testcontainers
class AuthControllerIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    // --- Happy Path ---

    @Test
    void login_WithValidCredentials_ReturnsTokens() throws Exception {
        // Arrange: create user directly in DB
        var user = User.builder()
            .email("test@example.com")
            .passwordHash(passwordEncoder.encode("password123"))
            .build();
        userRepository.save(user);

        // Act + Assert
        mockMvc.perform(post("/api/v1/auth/login")
                .contentType(APPLICATION_JSON)
                .content("""
                    {"email": "test@example.com", "password": "password123"}
                """))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.accessToken").isNotEmpty())
            .andExpect(jsonPath("$.data.user.email").value("test@example.com"));
    }

    // --- Error Path ---

    @Test
    void login_WithWrongPassword_Returns401() throws Exception {
        var user = User.builder()
            .email("test@example.com")
            .passwordHash(passwordEncoder.encode("correct"))
            .build();
        userRepository.save(user);

        mockMvc.perform(post("/api/v1/auth/login")
                .contentType(APPLICATION_JSON)
                .content("""
                    {"email": "test@example.com", "password": "wrong"}
                """))
            .andExpect(status().isUnauthorized())
            .andExpect(jsonPath("$.error.code").value("INVALID_CREDENTIALS"));
    }

    // --- Edge Case ---

    @Test
    void login_RateLimited_Returns429() throws Exception {
        // Hit rate limit: 5 requests
        for (int i = 0; i < 5; i++) {
            mockMvc.perform(post("/api/v1/auth/login")
                .contentType(APPLICATION_JSON)
                .content("""{"email": "test@example.com", "password": "wrong"}"""));
        }

        // 6th request should be rate limited
        mockMvc.perform(post("/api/v1/auth/login")
                .contentType(APPLICATION_JSON)
                .content("""{"email": "test@example.com", "password": "wrong"}"""))
            .andExpect(status().isTooManyRequests())
            .andExpect(jsonPath("$.error.code").value("RATE_LIMITED"));
    }
}
```

### Integration Test Rules

- ✅ Use real database (TestContainers or dedicated test DB). Never mock the DB.
- ✅ Test the full slice: controller → service → repository → database.
- ✅ Clean up before each test, not after (if test crashes, next one isn't polluted).
- ✅ Test authentication: valid token, expired token, missing token, wrong role.
- ❌ Don't mock the database. That's an unit test pretending to be integration.

---

## 🎨 FRONTEND (Component Integration)

### Testing a form submission flow

```typescript
// LoginForm.integration.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.post('/api/v1/auth/login', async ({ request }) => {
    const body = await request.json() as { email: string; password: string };
    if (body.email === 'test@example.com' && body.password === 'correct') {
      return HttpResponse.json({
        success: true,
        data: { accessToken: 'mock-token', user: { id: '123', email: body.email } }
      });
    }
    return HttpResponse.json(
      { success: false, error: { code: 'INVALID_CREDENTIALS' } },
      { status: 401 }
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

it('shows error message on invalid login', async () => {
  render(<LoginForm />);

  fireEvent.change(screen.getByLabelText(/email/i), {
    target: { value: 'test@example.com' }
  });
  fireEvent.change(screen.getByLabelText(/password/i), {
    target: { value: 'wrong' }
  });
  fireEvent.click(screen.getByRole('button', { name: /sign in/i }));

  await waitFor(() => {
    expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
  });
});
```

### MSW (Mock Service Worker) Rules

- Define handlers for every endpoint the component calls.
- Match happy path, error path, and edge cases.
- Use `server.use()` to override handlers for specific tests.
