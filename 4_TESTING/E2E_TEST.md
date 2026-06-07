# E2E_TEST.md — End-to-End User Flow Tests

<!--
  Real browser tests. Slow but definitive. AI agents write these for
  critical user journeys only — not for every button click.
-->

## E2E Strategy

- **Tool:** Playwright (preferred) or Cypress
- **Scope:** Critical user flows only. Happy path + key error paths.
- **Run:** On PR merge to main + nightly. Not on every push.
- **Timeout:** 5 minutes max per test file.

---

## Playwright Example (TypeScript)

### Setup

```typescript
// tests/e2e/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[data-test="email"]', 'e2e-test@example.com');
  await page.fill('[data-test="password"]', process.env.E2E_TEST_PASSWORD!);
  await page.click('[data-test="submit"]');
  await expect(page).toHaveURL('/dashboard');
  await page.context().storageState({ path: 'e2e/.auth/user.json' });
});
```

### Critical Flow: User Enables 2FA

```typescript
// tests/e2e/2fa-setup.spec.ts
import { test, expect } from '@playwright/test';

test.use({ storageState: 'e2e/.auth/user.json' });

test.describe('Two-Factor Authentication Setup', () => {

  test('UAT-01: enables 2FA with valid TOTP code', async ({ page }) => {
    // Navigate to security settings
    await page.goto('/settings/security');
    await expect(page.locator('[data-test="2fa-status"]')).toHaveText('Disabled');

    // Click enable
    await page.click('[data-test="enable-2fa"]');
    await expect(page.locator('[data-test="qr-code"]')).toBeVisible();
    await expect(page.locator('[data-test="manual-key"]')).toBeVisible();

    // Enter valid TOTP code (generated from test secret)
    const totpCode = generateTotpCode('JBSWY3DPEHPK3PXP'); // test secret
    await page.fill('[data-test="totp-input"]', totpCode);
    await page.click('[data-test="verify-2fa"]');

    // Assert success
    await expect(page.locator('[data-test="2fa-status"]')).toHaveText('Active');
    await expect(page.locator('[data-test="recovery-codes"]')).toBeVisible();

    // Count recovery codes
    const codes = await page.locator('[data-test="recovery-code"]').all();
    expect(codes).toHaveLength(10);
  });

  test('UAT-02: shows error on invalid TOTP code', async ({ page }) => {
    await page.goto('/settings/security');
    await page.click('[data-test="enable-2fa"]');

    await page.fill('[data-test="totp-input"]', '000000');
    await page.click('[data-test="verify-2fa"]');

    await expect(page.locator('[data-test="totp-error"]'))
      .toHaveText('Invalid code. Please try again.');
    await expect(page.locator('[data-test="qr-code"]')).toBeVisible(); // stays visible
  });

  test('UAT-06: login with recovery code', async ({ page }) => {
    // Use a fresh context (no stored auth) to force login
    const freshPage = await page.context().newPage();
    await freshPage.goto('/login');
    await freshPage.fill('[data-test="email"]', 'e2e-2fa@example.com');
    await freshPage.fill('[data-test="password"]', process.env.E2E_TEST_PASSWORD!);
    await freshPage.click('[data-test="submit"]');

    // Should see 2FA prompt
    await expect(freshPage.locator('[data-test="2fa-prompt"]')).toBeVisible();

    // Click "Use recovery code instead"
    await freshPage.click('[data-test="use-recovery-code"]');
    await freshPage.fill('[data-test="recovery-input"]', 'ABCD-EFGH-IJKL-MNOP');
    await freshPage.click('[data-test="verify-recovery"]');

    await expect(freshPage).toHaveURL('/dashboard');
  });
});
```

---

## Cypress Example (Alternative)

```typescript
// cypress/e2e/checkout.cy.ts
describe('Checkout Flow', () => {
  beforeEach(() => {
    cy.login('test-user@example.com', 'password123');
  });

  it('completes checkout with valid payment', () => {
    // Add item to cart
    cy.visit('/products');
    cy.get('[data-test="product-card"]').first().within(() => {
      cy.get('[data-test="add-to-cart"]').click();
    });
    cy.get('[data-test="cart-count"]').should('have.text', '1');

    // Go to cart
    cy.visit('/cart');
    cy.get('[data-test="checkout-button"]').click();

    // Fill payment
    cy.get('[data-test="card-number"]').type('4242424242424242');
    cy.get('[data-test="card-expiry"]').type('12/28');
    cy.get('[data-test="card-cvc"]').type('123');
    cy.get('[data-test="submit-payment"]').click();

    // Assert success
    cy.url().should('include', '/order-confirmation');
    cy.get('[data-test="order-status"]').should('have.text', 'Confirmed');
  });
});
```

---

## E2E Test Checklist

- [ ] Critical user flows covered (signup, login, core feature, logout)
- [ ] Test data isolated (dedicated test user, not shared with dev)
- [ ] Tests runnable in CI (headless browser)
- [ ] Failed tests produce screenshot + trace
- [ ] No flaky tests (pass 10/10 runs before merging)
- [ ] `data-test` attributes used (not CSS classes or text content)

## Rules for AI Agents

1. **E2E test only critical paths.** Not "every button exists." That's unit test territory.
2. **Use `data-test` attributes.** Never select by CSS class or XPath.
3. **Test data must be deterministic.** Seed test DB. Don't rely on dev data.
4. **Screenshot on failure.** `screenshot: 'only-on-failure'` in Playwright config.
5. **One `test.describe` per user flow.**
