# UNIT_TEST.md — Backend & Frontend Test Patterns

<!--
  How this project writes unit tests. AI agents clone these patterns.
  One section per language/framework.
-->

---

## 🔧 BACKEND

### <!-- Java + Spring Boot + JUnit 5 -->

#### Test Class Template
```java
package com.example.userservice;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.*;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    // --- Happy Path ---

    @Test
    void findById_WhenUserExists_ReturnsUserDto() {
        // Arrange
        var userId = UUID.randomUUID();
        var user = User.builder()
            .id(userId)
            .email("test@example.com")
            .name("Test User")
            .build();
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));

        // Act
        var result = userService.findById(userId);

        // Assert
        assertThat(result)
            .isNotNull()
            .satisfies(dto -> {
                assertThat(dto.getId()).isEqualTo(userId);
                assertThat(dto.getEmail()).isEqualTo("test@example.com");
            });
    }

    // --- Error Paths ---

    @Test
    void findById_WhenUserNotFound_ThrowsNotFoundException() {
        var userId = UUID.randomUUID();
        when(userRepository.findById(userId)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> userService.findById(userId))
            .isInstanceOf(NotFoundException.class)
            .hasMessageContaining("User not found");
    }
}
```

#### What to Mock
- ✅ Repositories (in service tests)
- ✅ External API clients (RestTemplate, WebClient)
- ✅ Clock (`java.time.Clock` — inject, don't use `Instant.now()` directly)
- ❌ The class under test
- ❌ Value objects / DTOs (just create real instances)

#### Common Patterns
```java
// Testing exceptions
assertThatThrownBy(() -> service.delete(userId))
    .isInstanceOf(BusinessException.class)
    .hasMessageContaining("Cannot delete active user");

// Testing with Clock (time-dependent logic)
@Test
void generateToken_ExpiresIn15Minutes() {
    var fixedClock = Clock.fixed(Instant.parse("2026-06-07T00:00:00Z"), ZoneOffset.UTC);
    var service = new TokenService(fixedClock);

    var token = service.generate();
    assertThat(token.getExpiresAt()).isEqualTo(Instant.now(fixedClock).plusSeconds(900));
}

// Parameterized test
@ParameterizedTest
@ValueSource(strings = {"", "  ", "invalid-email"})
void validateEmail_WithInvalidInput_ThrowsValidationException(String email) {
    assertThatThrownBy(() -> validator.validateEmail(email))
        .isInstanceOf(ValidationException.class);
}
```

---

### <!-- Python + FastAPI + pytest -->

```python
import pytest
from unittest.mock import AsyncMock

class TestShortenerService:
    async def test_generateSlug_returnsSixCharString(self):
        slug = ShortenerService.generate_slug()
        assert len(slug) == 6
        assert slug.isalnum()

    async def test_generateSlug_retriesOnCollision(self, mocker):
        mock_db = AsyncMock()
        mock_db.execute.side_effect = [
            MagicMock(scalar_one_or_none=MagicMock(return_value=True)),   # 1st: collision
            MagicMock(scalar_one_or_none=MagicMock(return_value=None)),   # 2nd: OK
        ]
        slug = await ShortenerService(mock_db).create_slug()
        assert mock_db.execute.call_count == 2
```

---

## 🎨 FRONTEND

### <!-- React + Vitest + Testing Library -->

```typescript
// ProductCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { ProductCard } from './ProductCard';

describe('ProductCard', () => {
  const mockProduct = {
    id: 'abc-123',
    name: 'Wireless Headphones',
    price: 99.99,
    imageUrl: '/images/headphones.jpg',
    inStock: true,
  };

  // --- Rendering ---

  it('renders product name and price', () => {
    render(<ProductCard product={mockProduct} />);
    expect(screen.getByText('Wireless Headphones')).toBeInTheDocument();
    expect(screen.getByText('$99.99')).toBeInTheDocument();
  });

  // --- Interactions ---

  it('calls onAddToCart when button clicked', () => {
    const onAddToCart = vi.fn();
    render(<ProductCard product={mockProduct} onAddToCart={onAddToCart} />);

    fireEvent.click(screen.getByRole('button', { name: /add to cart/i }));
    expect(onAddToCart).toHaveBeenCalledWith('abc-123');
  });

  // --- States ---

  it('shows "Notify Me" when out of stock', () => {
    render(<ProductCard product={{ ...mockProduct, inStock: false }} />);
    expect(screen.getByText(/notify me/i)).toBeInTheDocument();
    expect(screen.queryByText(/add to cart/i)).not.toBeInTheDocument();
  });

  it('shows sale badge when compareAtPrice is set', () => {
    render(<ProductCard product={{ ...mockProduct, compareAtPrice: 149.99 }} />);
    expect(screen.getByText(/sale/i)).toBeInTheDocument();
    expect(screen.getByText('$149.99')).toHaveClass('line-through');
  });

  // --- Accessibility ---

  it('image has alt text', () => {
    render(<ProductCard product={mockProduct} />);
    expect(screen.getByRole('img')).toHaveAttribute('alt', 'Wireless Headphones');
  });
});
```

### <!-- Vue 3 + Vitest + vue/test-utils -->

```typescript
// ProductCard.test.ts
import { mount } from '@vue/test-utils';
import { describe, it, expect, vi } from 'vitest';
import ProductCard from './ProductCard.vue';

describe('ProductCard', () => {
  const mockProduct = {
    id: 'abc-123',
    name: 'Wireless Headphones',
    price: 99.99,
    inStock: true,
  };

  it('emits add-to-cart when button clicked', async () => {
    const wrapper = mount(ProductCard, {
      props: { product: mockProduct },
    });

    await wrapper.find('[data-test="add-to-cart"]').trigger('click');
    expect(wrapper.emitted('add-to-cart')).toBeTruthy();
    expect(wrapper.emitted('add-to-cart')![0]).toEqual(['abc-123']);
  });

  it('renders "Out of Stock" when inStock is false', () => {
    const wrapper = mount(ProductCard, {
      props: { product: { ...mockProduct, inStock: false } },
    });
    expect(wrapper.text()).toContain('Out of Stock');
  });
});
```

---

## Frontend Testing Checklist (Per Component)

- [ ] Renders with required props
- [ ] Renders all prop variations (loading, empty, error, edge cases)
- [ ] User interactions trigger correct callbacks
- [ ] Conditional rendering works (show/hide based on props)
- [ ] Accessible: keyboard navigable, screen-reader friendly
- [ ] No console warnings or errors in test output
