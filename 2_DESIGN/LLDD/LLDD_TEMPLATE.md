# LLDD_TEMPLATE.md — Low-Level Design Document

<!--
  Per-component detail. Class design, method signatures, database queries.
  Copy to 2_DESIGN/LLDD/LLDD_01_COMPONENT_NAME.md
-->

## Document Control
- **Document ID:** LLDD-XXX
- **Component:** <!-- e.g., TotpService -->
- **Version:** 1.0
- **Status:** Draft | Review | Approved
- **Traces to:** <!-- HLDD-001 (Auth Service), DESIGN_SPEC.md -->

---

## 1. Component Overview

### 1.1 Responsibility
<!-- One sentence. What does this single component do? -->
**TotpService** generates TOTP secrets, creates QR code URIs, and verifies TOTP codes per RFC 6238.

### 1.2 Place in Module
```
TwoFactorController
    │
    ▼
TotpService  ◄── this component
    │
    ├── UserTotpRepository (data access)
    ├── EncryptionService   (secret encryption)
    └── java-otp library    (RFC 6238 implementation)
```

---

## 2. Class Design

### 2.1 Public Interface

```java
public interface TotpService {

    /**
     * Generate a new TOTP secret + QR code URI for the given user.
     * @throws TotpAlreadyEnabledException if user already has active 2FA
     */
    TotpSetupResponse generateSecret(UUID userId);

    /**
     * Verify a TOTP code against the user's stored secret.
     * @return true if code is valid for the current time window
     */
    boolean verifyCode(UUID userId, String code);

    /**
     * Get the QR code URI for manual setup (re-display).
     * @throws TotpNotConfiguredException if user hasn't started 2FA setup
     */
    String getQrCodeUri(UUID userId);

    /**
     * Disable 2FA for a user. Requires current password for security.
     */
    void disable(UUID userId, String password);
}
```

### 2.2 Implementation Structure

```java
@Service
@RequiredArgsConstructor
public class TotpServiceImpl implements TotpService {

    private final UserTotpRepository totpRepository;
    private final EncryptionService encryptionService;
    private final Clock clock;  // Inject for testability

    // --- Public Methods ---

    @Override
    @Transactional
    public TotpSetupResponse generateSecret(UUID userId) {
        // 1. Check not already enabled
        // 2. Generate 20-byte secret (SecureRandom)
        // 3. Encrypt secret (AES-256-GCM)
        // 4. Save to DB (status: PENDING_VERIFICATION)
        // 5. Generate QR URI
        // 6. Return response
    }

    @Override
    @Transactional
    public boolean verifyCode(UUID userId, String code) {
        // 1. Fetch stored secret
        // 2. Decrypt
        // 3. Verify TOTP (±1 time step for clock skew)
        // 4. Replay protection: store lastUsedTimestamp, reject same window
        // 5. On first success: set status ACTIVE
        // 6. Return true/false
    }

    // --- Private Helpers ---

    private String generateSecret() {
        byte[] bytes = new byte[20];
        new SecureRandom().nextBytes(bytes);
        return Base32.encode(bytes);
    }

    private String buildQrCodeUri(String secret, String email, String issuer) {
        return String.format(
            "otpauth://totp/%s:%s?secret=%s&issuer=%s&algorithm=SHA1&digits=6&period=30",
            issuer, email, secret, issuer
        );
    }
}
```

---

## 3. Data Access

### 3.1 Repository Interface

```java
public interface UserTotpRepository extends JpaRepository<UserTotp, UUID> {

    Optional<UserTotp> findByUserId(UUID userId);

    @Modifying
    @Query("UPDATE UserTotp t SET t.isActive = false, t.deactivatedAt = :now WHERE t.userId = :userId")
    void deactivate(UUID userId, Instant now);
}
```

### 3.2 Entity Mapping

```java
@Entity
@Table(name = "user_totp")
public class UserTotp {

    @Id
    @GeneratedValue
    private UUID id;

    @Column(name = "user_id", nullable = false, unique = true)
    private UUID userId;

    @Column(name = "encrypted_secret", nullable = false, columnDefinition = "BYTEA")
    private byte[] encryptedSecret;

    @Column(name = "encryption_key_id", nullable = false)
    private String encryptionKeyId;  // Which KMS key version

    @Column(name = "is_active", nullable = false)
    private boolean isActive;

    @Column(name = "last_used_at")
    private Instant lastUsedAt;  // For replay protection

    @Column(name = "verified_at")
    private Instant verifiedAt;

    @Column(name = "created_at", nullable = false)
    private Instant createdAt;

    @Column(name = "updated_at", nullable = false)
    private Instant updatedAt;

    @PrePersist
    void onCreate() {
        createdAt = Instant.now();
        updatedAt = Instant.now();
    }

    @PreUpdate
    void onUpdate() {
        updatedAt = Instant.now();
    }
}
```

---

## 4. Algorithm Detail: TOTP Verification

```
verifyCode(userId, code):
    1. FETCH totpRecord WHERE user_id = userId AND is_active IN (true, pending)
       → NULL? throw TotpNotConfiguredException

    2. DECRYPT secret = encryptionService.decrypt(totpRecord.encryptedSecret)

    3. CALCULATE currentWindow = floor(now() / 30)

    4. FOR window IN [currentWindow-1, currentWindow, currentWindow+1]:
         expectedCode = TOTP(secret, window)
         IF constantTimeEquals(code, expectedCode):
             GOTO step 5
       → No match: RETURN false

    5. REPLAY CHECK:
         IF totpRecord.lastUsedWindow == currentWindow:
             RETURN false  // Code already used this window
         totpRecord.lastUsedWindow = currentWindow

    6. FIRST VERIFICATION?
         IF totpRecord.status == PENDING:
             totpRecord.status = ACTIVE
             totpRecord.verifiedAt = now()

    7. SAVE totpRecord

    8. RETURN true
```

---

## 5. Error Handling

| Scenario | Exception | HTTP Status | Error Code |
|----------|-----------|-------------|------------|
| User has no TOTP setup | `TotpNotConfiguredException` | 404 | `TOTP_NOT_CONFIGURED` |
| User already has active 2FA | `TotpAlreadyEnabledException` | 409 | `TOTP_ALREADY_ENABLED` |
| Invalid TOTP code | (return false) | 401 | `INVALID_TOTP_CODE` |
| Rate limited | `RateLimitExceededException` | 429 | `RATE_LIMITED` |
| Encryption failure | `EncryptionException` | 500 | `INTERNAL_ERROR` |

---

## 6. Test Cases (Unit)

```java
class TotpServiceTest {

    // Happy path
    void generateSecret_ReturnsQrCodeUri() { }
    void verifyCode_WithValidCode_ReturnsTrue() { }
    void verifyCode_FirstVerification_ActivatesTotp() { }

    // Edge cases
    void verifyCode_WithClockSkewMinus30s_ReturnsTrue() { }
    void verifyCode_WithClockSkewPlus30s_ReturnsTrue() { }
    void verifyCode_WithClockSkew60s_ReturnsFalse() { }
    void verifyCode_ReplaySameWindow_ReturnsFalse() { }

    // Error paths
    void verifyCode_NoTotpConfigured_ThrowsException() { }
    void generateSecret_AlreadyEnabled_ThrowsException() { }
    void verifyCode_AfterFiveFailures_RateLimited() { }
}
```

---

_Last updated: YYYY-MM-DD_
