# EveryX API Documentation

> Version: 3.4.1 | Generated: 2026-02-11

## Table of Contents

- [Authentication Guide](#authentication-guide)
- [Security Verification](#security-verification)
- [Error Handling](#error-handling)
- [Authentication](#authentication) (12 endpoints)
- [User Profile](#user-profile) (17 endpoints)
- [Two-Factor Auth](#two-factor-auth) (6 endpoints)
- [Wallets](#wallets) (4 endpoints)
- [Trading](#trading) (3 endpoints)
- [Events](#events) (21 endpoints)
- [Dashboard](#dashboard) (5 endpoints)
- [Deposits](#deposits) (2 endpoints)
- [Withdrawals](#withdrawals) (2 endpoints)

---

## Authentication Guide

### JWT Bearer Token

Most endpoints require authentication via JWT Bearer Token in the `Authorization` header:

```
Authorization: Bearer <token>
```

**Obtaining a token:**

1. **Email + OTP flow**: `POST /auth/v2/otp/token` with email to receive OTP via email, then `POST /tokens` with OTP code to get JWT
2. **Wallet flow**: `GET /auth/v2/wallet/nonce` to get a nonce, sign it with your wallet, then `POST /auth/v2/wallet/verify` with the signature
3. **Google OAuth**: `POST /auth/v2/google/verify` with Google ID token

**Token lifecycle:**

- Tokens are valid until explicitly destroyed or expired
- Use `POST /tokens/validate` to check token validity
- Use `DELETE /tokens` to invalidate a token

### Developer Authentication (HMAC-SHA256)

For server-to-server integrations, developer authentication is available as an alternative to JWT. This uses HMAC-SHA256 signatures and bypasses fingerprint/captcha requirements.

**Headers required:**

- `X-Developer-Id`: Your developer account ID
- `X-Developer-Timestamp`: Unix timestamp (seconds) - must be within 5 minutes of server time
- `X-Developer-Signature`: HMAC-SHA256 signature of the request

**Signature computation:**

```
message = HTTP_METHOD + path + timestamp + canonical_body
signature = HMAC-SHA256(message, developer_secret)
```

Where `canonical_body` is the JSON body with sorted keys (empty string if no body).

Contact the EveryX team for developer credentials.

---

## Security Verification

Certain API endpoints may require additional security verification beyond authentication. The system uses a layered approach:

### HTTP 428 - Security Verification Required

When an endpoint requires security verification and the required headers are missing, you will receive:

```json
{
    "statusCode": 428,
    "error": "Precondition Required",
    "message": "Security verification required"
}
```

### Fingerprint Verification

Some endpoints require device fingerprint verification. When required, include these headers:

- `X-Fingerprint-Sealed-Result`: Sealed fingerprint result from Fingerprint.com
- `X-Fingerprint-Request-Id`: Request ID from Fingerprint.com

**Fingerprint levels:**

- **Required**: Always required (e.g., registration). Returns HTTP 428 without headers.
- **Conditional**: Based on server-side rules (user level, country, risk score). The API may return `X-Security-Required` response header to indicate frontend should collect fingerprint for future requests.
- **None**: Public endpoints with no security verification.

### CAPTCHA Verification

Some high-risk endpoints require CAPTCHA verification:

- `X-Captcha-Token`: CAPTCHA solution token

### Developer Auth Bypass

Requests authenticated via Developer Auth (HMAC-SHA256) bypass all fingerprint and captcha requirements. This is the recommended approach for server-to-server integrations.

---

## Error Handling

### Response Format

**Success responses:**

```json
{
    "success": true,
    "data": { ... },
    "meta": {
        "version": "3.5.1",
        "timestamp": "2024-01-01T00:00:00Z"
    }
}
```

**Error responses:**

```json
{
    "statusCode": 400,
    "error": "Bad Request",
    "message": "Description of what went wrong"
}
```

### Common HTTP Status Codes

| Code | Meaning                                                |
| ---- | ------------------------------------------------------ |
| 200  | Success                                                |
| 201  | Created                                                |
| 400  | Bad Request - invalid input or validation failure      |
| 401  | Unauthorized - missing or invalid authentication       |
| 403  | Forbidden - insufficient permissions or trust level    |
| 404  | Not Found                                              |
| 409  | Conflict - resource already exists                     |
| 422  | Unprocessable Entity - valid syntax but semantic error |
| 428  | Precondition Required - security verification needed   |
| 429  | Too Many Requests - rate limit exceeded                |
| 500  | Internal Server Error                                  |
| 503  | Service Unavailable - maintenance mode                 |

### Rate Limiting

API endpoints are rate-limited. When exceeded, you receive HTTP 429 with a `Retry-After` header indicating seconds to wait.

### Pagination

List endpoints support pagination via query parameters:

- `page` - Page number (default: 1)
- `limit` - Items per page (default varies, max 100)

Paginated responses include:

```json
{
    "pagination": {
        "page": 1,
        "limit": 20,
        "total": 150,
        "pages": 8
    }
}
```

---

## Authentication

### GET /tokens/validate

> Checks if the provided token is still valid

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

**Response 200:**

```json
{
    "valid": true
}
```

---

### GET /auth/v2/login/{email}

> Login v2, Send Login Mail

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name  | In   | Type   | Required | Description |
| ----- | ---- | ------ | -------- | ----------- |
| email | path | string | yes      |             |

**Response 200:**

```json
{
    "message": "string"
}
```

---

### GET /auth/v2/user/exists/{email}

> Check if User Exists

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name  | In   | Type   | Required | Description |
| ----- | ---- | ------ | -------- | ----------- |
| email | path | string | yes      |             |

**Response 200:**

```json
true
```

---

### POST /passwords

> User Forget Password

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name     | In       | Type   | Required | Description |
| -------- | -------- | ------ | -------- | ----------- |
| email    | formData | string | yes      |             |
| token    | formData | string | no       |             |
| password | formData | string | no       |             |

**Response 201:**

```json
"string"
```

---

### PUT /passwords

> Change User Password

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description                                      |
| ------------- | -------- | ------ | -------- | ------------------------------------------------ |
| authorization | header   | string | yes      |                                                  |
| old_password  | formData | string | yes      | Current password of the user to verify identity. |
| password      | formData | string | yes      |                                                  |

---

### POST /register

> Register User with process-based rate limiting, email validation, and CAPTCHA support. Use /v1/register for legacy endpoint. This endpoint is blocked during system maintenance.

| Security    |               |
| ----------- | ------------- |
| Auth        | None (public) |
| Fingerprint | **Required**  |
| CAPTCHA     | **Required**  |

**Parameters:**

| Name           | In       | Type   | Required | Description |
| -------------- | -------- | ------ | -------- | ----------- |
| email          | formData | string | yes      |             |
| display_name   | formData | string | no       |             |
| first_name     | formData | string | no       |             |
| middle_name    | formData | string | no       |             |
| last_name      | formData | string | no       |             |
| country        | formData | string | no       |             |
| state          | formData | string | no       |             |
| city           | formData | string | no       |             |
| street         | formData | string | no       |             |
| building       | formData | string | no       |             |
| zip            | formData | string | no       |             |
| phone          | formData | string | no       |             |
| referral_code  | formData | string | no       |             |
| myaffiliate_id | formData | string | no       |             |
| tracking_id    | formData | string | no       |             |
| source         | formData | string | no       |             |

**Response 200:**

```json
{
    "action": "login",
    "message": "string",
    "email": "string"
}
```

---

### POST /tokens

> Request User Token for Accessing User API. This endpoint is blocked during system maintenance.

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name     | In       | Type   | Required | Description |
| -------- | -------- | ------ | -------- | ----------- |
| email    | formData | string | yes      |             |
| password | formData | string | yes      |             |

**Response 201:**

```json
{
    "_id": "string",
    "created_at": "string",
    "token": "string"
}
```

---

### DELETE /tokens

> User logs out from the system

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

---

### POST /auth/v2/google/verify

> Verify the Google ID token received from the frontend. This endpoint is blocked during system maintenance.

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name           | In   | Type   | Required | Description |
| -------------- | ---- | ------ | -------- | ----------- |
| idToken        | body | string | yes      |             |
| email          | body | string | yes      |             |
| referral_code  | body | string | no       |             |
| myaffiliate_id | body | string | no       |             |
| tracking_id    | body | string | no       |             |
| source         | body | source | no       |             |

**Request Body:**

```json
{
    "idToken": "string",
    "email": "string",
    "referral_code": "string",
    "myaffiliate_id": "string",
    "tracking_id": "string",
    "source": "organic"
}
```

**Response 201:**

```json
{
    "success": true,
    "user": {
        "id": "string",
        "email": "string",
        "name": "string"
    },
    "user_action": "login",
    "is_new_user": true,
    "created_at": "string",
    "token": {},
    "tracking_data": {
        "ip": "string",
        "wallet_address": "string"
    },
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

### POST /auth/v2/otp/token

> Generate authentication token for users who have been verified via OTP (no password required)

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name     | In   | Type   | Required | Description                             |
| -------- | ---- | ------ | -------- | --------------------------------------- |
| email    | body | string | yes      | Email address of the user               |
| otp_code | body | string | yes      | 6-digit OTP code sent to the user email |

**Request Body:**

```json
{
    "email": "string",
    "otp_code": "string"
}
```

**Response 201:**

```json
{
    "success": true,
    "token": "string",
    "user": {
        "id": "string",
        "email": "string",
        "name": "string"
    }
}
```

---

### POST /auth/v2/wallet/nonce

> Generate a nonce for MetaMask authentication

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name          | In   | Type   | Required | Description |
| ------------- | ---- | ------ | -------- | ----------- |
| walletAddress | body | string | yes      |             |

**Request Body:**

```json
{
    "walletAddress": "string"
}
```

---

### POST /auth/v2/wallet/verify

> Verify MetaMask signature and issue JWT token. This endpoint is blocked during system maintenance.

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name           | In   | Type   | Required | Description                                                  |
| -------------- | ---- | ------ | -------- | ------------------------------------------------------------ |
| signature      | body | string | yes      | Ethereum signature in 0x-prefixed 65-byte hex format (r,s,v) |
| walletAddress  | body | string | yes      |                                                              |
| referral_code  | body | string | no       |                                                              |
| myaffiliate_id | body | string | no       |                                                              |
| tracking_id    | body | string | no       |                                                              |
| source         | body | source | no       |                                                              |

**Request Body:**

```json
{
    "signature": "string",
    "walletAddress": "string",
    "referral_code": "string",
    "myaffiliate_id": "string",
    "tracking_id": "string",
    "source": "organic"
}
```

**Response 201:**

```json
{
    "success": true,
    "user": {
        "id": "string",
        "email": "string",
        "name": "string"
    },
    "user_action": "login",
    "is_new_user": true,
    "created_at": "string",
    "token": {},
    "tracking_data": {
        "ip": "string",
        "wallet_address": "string"
    },
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

## User Profile

### GET /me

> Request Current User Data

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

**Response 200:**

```json
{
    "email": "string",
    "wallet_address": "string",
    "full_name": "string",
    "first_name": "string",
    "middle_name": "string",
    "last_name": "string",
    "display_name": "string",
    "about": "string",
    "country": "string",
    "state": "string",
    "city": "string",
    "street": "string",
    "building": "string",
    "zip": "string",
    "phone": "string",
    "currency": "string",
    "current_level": 0,
    "trader_id": "string",
    "avatar": "string",
    "email_verified": true,
    "identity_verified": true,
    "redo_kyc": true,
    "referral_code": "string",
    "referral_code_expires_at": "string",
    "referral_code_quota": 0,
    "phone_verified": true,
    "is_new": true,
    "isUpgraded": true,
    "is_upgraded": true,
    "leaderBoardCampaignShown": true,
    "leaderboard_campaign_shown": true,
    "terms_accepted": true,
    "terms_accepted_at": "string",
    "user_level": {
        "current_level": 0,
        "level_name": "string",
        "coratio": 0,
        "perks": {
            "can_browse_events": true,
            "can_deposit_crypto": true,
            "can_trade_regular": true,
            "can_refer_friends": true,
            "can_receive_bonuses": true,
            "can_use_bonuses": true,
            "can_withdraw": true,
            "can_deposit_credit_card": true,
            "can_trade_vip_events": true
        },
        "all_levels_progress": {
            "string": {
                "level": "...",
                "name": "...",
                "coratio": "...",
                "is_completed": "...",
                "is_current": "...",
                "requirements": "...",
                "progress": "..."
            }
        },
        "bypass_enabled": true
    },
    "withdrawal_limits": {
        "period": "daily",
        "limit": 0,
        "used": 0,
        "remaining": 0,
        "reset_date": "string"
    },
    "can_add_invite_code": true,
    "invite_code_deadline": "string",
    "terms_compliance": {
        "compliant": true,
        "requiredVersion": "string",
        "acceptedVersion": "string",
        "acceptedAt": "string",
        "reason": "string",
        "url": "string"
    }
}
```

---

### GET /profile/level-upgrade-requirements

> Returns T&C and other requirements needed before user can upgrade to next level. Used to check if user must accept new terms before level promotion.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description                                                            |
| ------------- | ------ | ------- | -------- | ---------------------------------------------------------------------- |
| authorization | header | string  | yes      |                                                                        |
| target_level  | query  | integer | no       | Target level to check requirements for. Defaults to current level + 1. |

**Response 200:**

```json
{
    "success": true,
    "current_level": 0,
    "target_level": 0,
    "requirements": {
        "terms": {
            "required": true,
            "compliant": true,
            "required_version": "string",
            "accepted_version": "string",
            "url": "string",
            "reason": "string"
        }
    },
    "can_upgrade": true
}
```

---

### POST /email-verification

> Send a verification OTP code to the user. Optionally update email if provided.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description                  |
| ------------- | -------- | ------ | -------- | ---------------------------- |
| authorization | header   | string | yes      |                              |
| email         | formData | string | no       | New email address (optional) |
| type          | formData | string | no       | Verification type            |
| ref           | formData | string | no       | Referral code                |

**Response 201:**

```json
{
    "success": true,
    "message": "string"
}
```

---

### POST /email-verification/resend

> Resend email verification for the authenticated user

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description   |
| ------------- | -------- | ------ | -------- | ------------- |
| authorization | header   | string | yes      |               |
| ref           | formData | string | no       | Referral code |

**Response 201:**

```json
{
    "success": true,
    "message": "string"
}
```

---

### POST /email-verification/verify

> Verify 6-digit OTP code for login flow

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name  | In   | Type   | Required | Description             |
| ----- | ---- | ------ | -------- | ----------------------- |
| email | body | string | yes      | Email address to verify |
| token | body | string | yes      | 6-digit OTP code        |

**Request Body:**

```json
{
    "email": "string",
    "token": "string"
}
```

**Response 200:**

```json
{
    "success": true,
    "message": "string",
    "tracking_data": {
        "ip": "string"
    }
}
```

---

### POST /email-verification/verify-signup

> Verify OTP code for signup flow (user may not exist yet)

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name  | In   | Type   | Required | Description             |
| ----- | ---- | ------ | -------- | ----------------------- |
| email | body | string | yes      | Email address to verify |
| token | body | string | yes      | 6-digit OTP code        |

**Request Body:**

```json
{
    "email": "string",
    "token": "string"
}
```

**Response 200:**

```json
{
    "success": true,
    "message": "string",
    "tracking_data": {
        "ip": "string"
    }
}
```

---

### POST /phone/complete

> Verifies OTP and moves phone_pending to phone. Phone number is taken from user profile.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description               |
| ------------- | ------ | ------ | -------- | ------------------------- |
| authorization | header | string | yes      |                           |
| token         | body   | string | yes      | OTP token entered by user |

**Request Body:**

```json
{
    "token": "string"
}
```

**Response 200:**

```json
{
    "ok": true,
    "verificationData": {}
}
```

---

### POST /phone/resend-otp

> Uses UserOtpService with 10-minute deduplication window (matches OTP expiry) to prevent duplicate SMS sends. Protected by rate limiting (3 requests per 5 minutes) and deduplication. SECURITY: Phone number is NOT accepted from request payload - uses user's stored phone only.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type   | Required | Description                                                                                               |
| --------------- | ------ | ------ | -------- | --------------------------------------------------------------------------------------------------------- |
| authorization   | header | string | yes      |                                                                                                           |
| x-captcha-token | header | string | no       | CAPTCHA response token (Google reCAPTCHA or Cloudflare Turnstile) - required only when CAPTCHA is enabled |

**Response 200:**

```json
{
    "ok": true,
    "channel": "string",
    "ttl": 0
}
```

---

### POST /profile/accept-terms

> Accept a specific version of terms and conditions. Records the version and timestamp of acceptance.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description                                                                                       |
| ------------- | ------ | ------- | -------- | ------------------------------------------------------------------------------------------------- |
| authorization | header | string  | yes      |                                                                                                   |
| terms_version | body   | string  | no       | Version of terms being accepted. If omitted, uses the latest version for user level.              |
| level         | body   | integer | no       | User level for T&C acceptance (-1=registration, 0-3=user levels). Defaults to current user level. |

**Request Body:**

```json
{
    "terms_version": "string",
    "level": 0
}
```

**Response 200:**

```json
{
    "success": true,
    "version": "string",
    "level": 0,
    "accepted_at": "string",
    "already_accepted": true
}
```

---

### POST /profile/check-phone

> Returns whether a phone number is available for use

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description           |
| ------------- | ------ | ------ | -------- | --------------------- |
| authorization | header | string | yes      |                       |
| phone         | body   | string | yes      | Phone number to check |

**Request Body:**

```json
{
    "phone": "string"
}
```

**Response 200:**

```json
{
    "ok": true,
    "available": true,
    "message": "string"
}
```

---

### POST /profile/invite-code

> Allows users who registered without an invite code to add one within a configurable time window.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description          |
| ------------- | ------ | ------ | -------- | -------------------- |
| authorization | header | string | yes      |                      |
| code          | body   | string | yes      | Invite code to apply |

**Request Body:**

```json
{
    "code": "string"
}
```

**Response 200:**

```json
{
    "success": true,
    "code": "string",
    "bonus_amount": 0
}
```

---

### POST /profile/verify-email

> Verify 6-digit OTP code sent to email and mark email as verified

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description             |
| ------------- | -------- | ------ | -------- | ----------------------- |
| authorization | header   | string | yes      |                         |
| email         | formData | string | yes      | Email address to verify |
| otp           | formData | string | yes      | 6-digit OTP code        |
| type          | formData | string | no       | Verification type       |

**Response 200:**

```json
{
    "success": true,
    "message": "string"
}
```

---

### POST /profile/verify-phone

> Update phone verification status for the authenticated user

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name               | In     | Type                  | Required | Description                                    |
| ------------------ | ------ | --------------------- | -------- | ---------------------------------------------- |
| authorization      | header | string                | yes      |                                                |
| x-totp-token       | header | string                | no       | Optional TOTP code for inline 2FA verification |
| phone              | body   | string                | yes      | Phone number to verify                         |
| verified           | body   | boolean               | yes      | Verification status                            |
| verificationData   | body   | PhoneVerificationData | no       |                                                |
| verificationMethod | body   | verificationMethod    | no       |                                                |

**Request Body:**

```json
{
    "phone": "string",
    "verified": true,
    "verificationData": {
        "number": "string",
        "dialCode": "string"
    },
    "verificationMethod": "sms_otp"
}
```

**Response 200:**

```json
{
    "ok": true,
    "phone_verified": true,
    "message": "string"
}
```

---

### PUT /phone

> User Change Their Phone Number. Automatically sends OTP to the new number. The user phone number will be updated after OTP verification.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description |
| ------------- | -------- | ------ | -------- | ----------- |
| authorization | header   | string | no       |             |
| phone         | formData | string | no       |             |

**Response 201:**

```json
{
    "success": true,
    "message": "string"
}
```

---

### PUT /profile

> Update User Profile

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name                     | In     | Type    | Required | Description                                    |
| ------------------------ | ------ | ------- | -------- | ---------------------------------------------- |
| authorization            | header | string  | yes      |                                                |
| x-totp-token             | header | string  | no       | Optional TOTP code for inline 2FA verification |
| first_name               | body   | string  | no       |                                                |
| middle_name              | body   | string  | no       |                                                |
| last_name                | body   | string  | no       |                                                |
| display_name             | body   | string  | no       |                                                |
| about                    | body   | string  | no       |                                                |
| country                  | body   | string  | no       |                                                |
| state                    | body   | string  | no       |                                                |
| city                     | body   | string  | no       |                                                |
| street                   | body   | string  | no       |                                                |
| building                 | body   | string  | no       |                                                |
| zip                      | body   | string  | no       |                                                |
| phone                    | body   | string  | no       |                                                |
| avatar                   | body   | string  | no       | aka. User Photo                                |
| email                    | body   | string  | no       |                                                |
| is_new                   | body   | boolean | no       |                                                |
| leaderBoardCampaignShown | body   | boolean | no       |                                                |
| isUpgraded               | body   | boolean | no       |                                                |

**Request Body:**

```json
{
    "first_name": "string",
    "middle_name": "string",
    "last_name": "string",
    "display_name": "string",
    "about": "string",
    "country": "string",
    "state": "string",
    "city": "string",
    "street": "string",
    "building": "string",
    "zip": "string",
    "phone": "string",
    "avatar": "string",
    "email": "string",
    "is_new": true,
    "leaderBoardCampaignShown": true,
    "isUpgraded": true
}
```

**Response 200:**

```json
{
    "success": true,
    "user": {}
}
```

---

### PUT /profile/wallet

> Allows the user to update their wallet address after verifying phone OTP.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description                         |
| ------------- | -------- | ------ | -------- | ----------------------------------- |
| authorization | header   | string | yes      |                                     |
| walletAddress | formData | string | yes      | New Ethereum wallet address (0x...) |
| token         | formData | string | yes      | Phone verification OTP token        |

**Response 200:**

```json
{
    "success": true,
    "message": "string",
    "wallet_address": "string"
}
```

---

### PUT /user/profile/reset-upgrade-flag

> Resets the isUpgraded flag to false after frontend has shown the upgrade notification

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

**Response 200:**

```json
{
    "success": true,
    "message": "string"
}
```

---

## Two-Factor Auth

### GET /profile/2fa/status

> Get 2FA status for user

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

---

### POST /profile/2fa/backup-codes

> Regenerate backup codes (requires TOTP verification)

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                                       |
| ------------- | ------ | ------ | -------- | ------------------------------------------------- |
| authorization | header | string | yes      |                                                   |
| token         | body   | string | yes      | TOTP code from authenticator app for verification |

**Request Body:**

```json
{
    "token": "string"
}
```

---

### POST /profile/2fa/disable

> Disable 2FA (requires TOTP verification)

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                                                      |
| ------------- | ------ | ------ | -------- | ---------------------------------------------------------------- |
| authorization | header | string | yes      |                                                                  |
| token         | body   | string | yes      | TOTP code from authenticator app or backup code for verification |

**Request Body:**

```json
{
    "token": "string"
}
```

---

### POST /profile/2fa/setup

> Setup 2FA for user - generates secret and QR code

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

---

### POST /profile/2fa/verify

> Verify TOTP code and enable 2FA

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                              |
| ------------- | ------ | ------ | -------- | ---------------------------------------- |
| authorization | header | string | yes      |                                          |
| token         | body   | string | yes      | 6-digit TOTP code from authenticator app |

**Request Body:**

```json
{
    "token": "string"
}
```

---

### POST /profile/2fa/verify-token

> Verifies TOTP code and creates a 15-minute session. After successful verification, subsequent requests to 2FA-protected routes will pass through.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                                     |
| ------------- | ------ | ------ | -------- | ----------------------------------------------- |
| authorization | header | string | yes      |                                                 |
| token         | body   | string | yes      | TOTP code from authenticator app or backup code |

**Request Body:**

```json
{
    "token": "string"
}
```

---

## Wallets

### GET /wallets

> List wallets of current login user with optimized MySQL queries

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name             | In     | Type    | Required | Description |
| ---------------- | ------ | ------- | -------- | ----------- |
| authorization    | header | string  | yes      |             |
| include_balances | query  | boolean | no       |             |

**Response 200:**

```json
{
    "success": true,
    "wallets": {
        "topup": {
            "id": 0,
            "balance": 0,
            "currency": "string",
            "withdrawable": true,
            "can_wager": true,
            "last_transaction_at": "string"
        },
        "profit": {
            "id": 0,
            "balance": 0,
            "currency": "string",
            "withdrawable": true,
            "can_wager": true,
            "last_transaction_at": "string"
        },
        "bonus": {
            "id": 0,
            "balance": 0,
            "currency": "string",
            "withdrawable": true,
            "can_wager": true,
            "last_transaction_at": "string"
        }
    },
    "balances": {
        "topup": 0,
        "profit": 0,
        "bonus": 0,
        "total": 0,
        "withdrawable": 0
    },
    "trader_id": "string",
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

### GET /wallets/withdrawable-balance

> Get the user's withdrawable balance from V2 wallet system using formula: MAX(0, TOPUP + PROFIT - CORATIO \* X) where X = BONUS_BALANCE + BONUS_USED_IN_ACTIVE_WAGERS.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

**Response 200:**

```json
{
    "success": true,
    "withdrawable_balance": 0,
    "balances": {
        "topup": 0,
        "profit": 0,
        "bonus": 0,
        "total": 0,
        "withdrawable": 0
    },
    "breakdown": {
        "topup_withdrawable": 0,
        "profit_withdrawable": 0,
        "bonus_withdrawable": 0,
        "coratio_factor": 0,
        "coratio_value": 0,
        "bonus_balance": 0,
        "bonus_used_in_active_wagers": 0,
        "x_value": 0
    }
}
```

---

### GET /wallets/{wallet_type}/transactions

> List transactions for a specific wallet type in the V2 system

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name             | In     | Type   | Required | Description |
| ---------------- | ------ | ------ | -------- | ----------- |
| authorization    | header | string | yes      |             |
| wallet_type      | path   | string | yes      |             |
| from             | query  | string | no       |             |
| to               | query  | string | no       |             |
| transaction_type | query  | number | no       |             |

**Response 200:**

```json
{
    "success": true,
    "transactions": [
        {
            "id": 0,
            "amount": 0,
            "transaction_type": 0,
            "from": 0,
            "to": 0,
            "metadata": {},
            "event_name": "string",
            "event_id": "string",
            "created_at": "string",
            "status": "COMPLETED",
            "is_grouped": true
        }
    ]
}
```

---

### GET /wallets/{wallet_type}/details

> Get detailed information about a specific wallet type in the V2 system

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |
| wallet_type   | path   | string | yes      |             |

**Response 200:**

```json
{
    "success": true,
    "wallet": {
        "id": 0,
        "wallet_type": "string",
        "balance": 0,
        "name": "string",
        "currency": "string",
        "withdrawable": true,
        "can_wager": true,
        "last_transaction_at": "string",
        "is_active": true
    },
    "trader_id": "string"
}
```

---

## Trading

### GET /wagers/preflight

> Returns user balances, bonus limits, and max usable bonus for an event/outcome before placing a wager

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name       | In    | Type   | Required | Description                                                         |
| ---------- | ----- | ------ | -------- | ------------------------------------------------------------------- |
| event_id   | query | string | yes      | Event code                                                          |
| outcome_id | query | string | no       | Outcome code (optional - if omitted, returns data for all outcomes) |

**Response 200:**

```json
{
    "success": true,
    "data": {
        "event_id": "string",
        "outcome_id": "string",
        "user_level": 0,
        "balances": {
            "topup": 0,
            "profit": 0,
            "bonus": 0,
            "available_without_bonus": 0,
            "available_with_bonus": 0
        },
        "bonus_info": {
            "max_bonus_usable": 0,
            "bonus_limit_per_event": 0,
            "bonus_used_on_event": 0,
            "bonus_remaining_for_event": 0,
            "reason": "level_locked",
            "reason_message": "string"
        },
        "outcome_info": {
            "max_trade_size": 0
        }
    },
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

### GET /wagers/events/{event_id}

> Get all wager positions for the authenticated user for a specific event

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description                               |
| ------------- | ------ | ------- | -------- | ----------------------------------------- |
| authorization | header | string  | yes      |                                           |
| event_id      | path   | string  | yes      | Event ID (code) to filter wager positions |
| limit         | query  | integer | no       |                                           |
| page          | query  | integer | no       |                                           |
| pagination    | query  | boolean | no       |                                           |

**Response 200:**

```json
[
    {
        "type": "string",
        "id": "string",
        "trader_id": "string",
        "user_id": "string",
        "event_id": "string",
        "event_status": "string",
        "event_images_url": ["string"],
        "event": {
            "_id": "string",
            "code": "string",
            "ticker": "string",
            "name": "string",
            "description": "string",
            "ends_at": "string",
            "closed_at": "string",
            "status": "string",
            "resolved_outcome_id": "string",
            "outcomes": ["..."],
            "event_images_url": ["..."]
        },
        "wagered_at": "string",
        "pledge": 0,
        "is_leveraged": true,
        "wager": 0,
        "indicative_return": 0,
        "indicative_payout": 0,
        "unrealized_pnl": 0,
        "indicative_pnl": 0,
        "payout_return_for_new_wagers": 0,
        "probability": 0,
        "stop_probability": 0,
        "realized_return": 0,
        "realized_payout": 0,
        "realized_payoff": 0,
        "realized_pnl": 0,
        "final_indicative_return": 0,
        "fill_count": 0,
        "entry_probability": 0,
        "current_probability": 0,
        "probability_change": 0,
        "created_at": "string",
        "last_update_time": "string",
        "last_reason": "string",
        "positions": [
            {
                "type": "...",
                "id": "...",
                "trader_id": "...",
                "user_id": "...",
                "event_id": "...",
                "event_status": "...",
                "event_outcome_id": "...",
                "event": "...",
                "event_outcome": "...",
                "wagered_at": "...",
                "pledge": "...",
                "is_leveraged": "...",
                "leverage": "...",
                "wager": "...",
                "indicative_return": "...",
                "indicative_payout": "...",
                "unrealized_pnl": "...",
                "indicative_pnl": "...",
                "payout_return_for_new_wagers": "...",
                "probability": "...",
                "stop_probability": "...",
                "realized_return": "...",
                "realized_payout": "...",
                "realized_payoff": "...",
                "realized_pnl": "...",
                "final_indicative_return": "...",
                "fill_count": "...",
                "entry_probability": "...",
                "current_probability": "...",
                "probability_change": "...",
                "created_at": "...",
                "last_update_time": "...",
                "last_reason": "...",
                "status": "...",
                "stop_warning": "...",
                "stop_distance": "...",
                "wager_position_id": "...",
                "wager_id": "..."
            }
        ]
    }
]
```

---

### POST /wagers

> Make a wager (bet)

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name             | In       | Type    | Required | Description                                                                                                            |
| ---------------- | -------- | ------- | -------- | ---------------------------------------------------------------------------------------------------------------------- |
| authorization    | header   | string  | yes      |                                                                                                                        |
| wallet_id        | formData | integer | yes      |                                                                                                                        |
| event_id         | formData | string  | yes      |                                                                                                                        |
| event_outcome_id | formData | string  | yes      |                                                                                                                        |
| pledge           | formData | number  | yes      |                                                                                                                        |
| leverage         | formData | number  | yes      |                                                                                                                        |
| wager            | formData | number  | yes      |                                                                                                                        |
| loan             | formData | number  | yes      |                                                                                                                        |
| max_payout       | formData | number  | yes      |                                                                                                                        |
| force_leverage   | formData | boolean | no       | If set to True, then no matter what value is given for leverage the new wager will be added to the leveraged position. |

**Response 201:**

```json
{
    "id": 0,
    "event_id": "string",
    "event_outcome_id": "string",
    "pledge": 0,
    "leverage": 0,
    "wager": 0,
    "created_at": "string"
}
```

---

## Events

### GET /event-groups

> Get list of visible EventGroups (Mention Markets)

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name        | In    | Type    | Required | Description |
| ----------- | ----- | ------- | -------- | ----------- |
| page        | query | integer | no       |             |
| limit       | query | integer | no       |             |
| group_type  | query | string  | no       |             |
| is_featured | query | boolean | no       |             |
| status      | query | string  | no       |             |

**Response 200:**

```json
{
    "success": true,
    "data": [
        {
            "_id": "string",
            "code": "string",
            "slug": "string",
            "group_type": "string",
            "name": "string",
            "description": "string",
            "rules": "string",
            "image_url": "string",
            "banner_image_url": "string",
            "og_image_url": "string",
            "stream_url": "string",
            "status": "string",
            "is_featured": true,
            "starts_at": "string",
            "ends_at": "string",
            "match_date": "string",
            "timezone": "string",
            "trading_gate_open": true,
            "events": ["..."]
        }
    ],
    "pagination": {
        "currentPage": 0,
        "totalPages": 0,
        "totalCount": 0,
        "hasMore": true,
        "pageSize": 0,
        "itemsCount": 0
    },
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

### GET /search-events

> Search for Events using V2 optimized query

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type    | Required | Description                                                                                                                                                                                                                                                                                                                                               |
| --------------- | ------ | ------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| authorization   | header | string  | no       |                                                                                                                                                                                                                                                                                                                                                           |
| accept-language | header | string  | no       |                                                                                                                                                                                                                                                                                                                                                           |
| keyword         | query  | string  | no       | Search keyword                                                                                                                                                                                                                                                                                                                                            |
| tags            | query  | array   | no       | Tags (pass in slugs). Add more `tags` to query when searching more than one tag, for example: ?tags=tag-a&tags=tag-b. Use `&` (%26) to concat a category slug and a sub-category slug.                                                                                                                                                                    |
| categories      | query  | array   | no       | Categories (pass in slugs). Add more `categories` to query when searching more than one category, for example: ?categories=category-a&categories=category-b                                                                                                                                                                                               |
| collections     | query  | array   | no       | Collections (pass in slugs). Add more `collections` to query when searching more than one collection, for example: ?collections=collection-a&collections=collection-b                                                                                                                                                                                     |
| status          | query  | string  | no       | Filter events by status. `open`: currently open events, `closing-soon`: events whose ends_at date is within 24 hours UTC of current time, `pending-resolution`: events with status "closed" but not yet resolved to any outcome, `closed`: all closed events including those pending resolution, `resolved`: events that have been resolved to an outcome |
| sortby          | query  | string  | no       | Order By                                                                                                                                                                                                                                                                                                                                                  |
| limit           | query  | integer | no       |                                                                                                                                                                                                                                                                                                                                                           |
| page            | query  | integer | no       |                                                                                                                                                                                                                                                                                                                                                           |
| pagination      | query  | boolean | no       |                                                                                                                                                                                                                                                                                                                                                           |

**Response 200:**

```json
[
    {
        "_id": "string",
        "code": "string",
        "ticker": "string",
        "group_id": "string",
        "name": "string",
        "description": "string",
        "rules": "string",
        "status": "string",
        "ends_at": "string",
        "match_date": "string",
        "volume": 0,
        "participants_count": 0,
        "closed_at": "string",
        "category": {
            "slug": "string",
            "name": "string",
            "icon": "string",
            "num_events": 0
        },
        "sub_category": {
            "slug": "string",
            "name": "string",
            "icon": "string",
            "num_events": 0
        },
        "featured_tag": {
            "slug": "string",
            "name": "string",
            "icon": "string",
            "num_events": 0
        },
        "collections": [
            {
                "slug": "...",
                "name": "...",
                "icon": "...",
                "image_url": "...",
                "link": "...",
                "target": "...",
                "num_events": "..."
            }
        ],
        "tags": [
            {
                "slug": "...",
                "name": "...",
                "icon": "...",
                "num_events": "..."
            }
        ],
        "outcomes": [
            {
                "_id": "...",
                "code": "...",
                "event_id": "...",
                "name": "...",
                "description": "...",
                "image_url": "...",
                "disabled": "...",
                "trader_info": "...",
                "histories": "...",
                "has_wagered": "...",
                "has_cash_wagered": "...",
                "has_leveraged_wagered": "..."
            }
        ],
        "top_event_images_url": ["string"],
        "event_images_url": ["string"],
        "og_image_url": "string",
        "stream_url": "string",
        "is_top_events": true,
        "is_featured_events": true,
        "recommended_images_url": ["string"]
    }
]
```

---

### GET /event-groups/{slug}

> Get an EventGroup by slug with its associated events. Returns data suitable for og:tags.

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name | In   | Type   | Required | Description     |
| ---- | ---- | ------ | -------- | --------------- |
| slug | path | string | yes      | EventGroup slug |

**Response 200:**

```json
{
    "success": true,
    "data": {
        "_id": "string",
        "code": "string",
        "slug": "string",
        "group_type": "string",
        "name": "string",
        "description": "string",
        "rules": "string",
        "image_url": "string",
        "banner_image_url": "string",
        "og_image_url": "string",
        "stream_url": "string",
        "status": "string",
        "is_featured": true,
        "starts_at": "string",
        "ends_at": "string",
        "match_date": "string",
        "timezone": "string",
        "trading_gate_open": true,
        "events": [
            {
                "_id": "...",
                "code": "...",
                "name": "...",
                "status": "...",
                "visible": "...",
                "ends_at": "...",
                "match_date": "..."
            }
        ]
    },
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

### GET /events/match-event-calendar

> Get events grouped by match_date, returning event codes, count per date, and total count. By default, excludes created events and only includes visible events unless status parameter is specified.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type   | Required | Description                                 |
| --------------- | ------ | ------ | -------- | ------------------------------------------- |
| authorization   | header | string | no       |                                             |
| accept-language | header | string | no       |                                             |
| status          | query  | string | no       | Filter events by status                     |
| category        | query  | string | no       | Category slug to filter events              |
| start_date      | query  | string | no       | Start date for filtering events (inclusive) |
| end_date        | query  | string | no       | End date for filtering events (inclusive)   |
| timezone        | query  | string | no       | Timezone for date formatting                |

**Response 200:**

```json
{
    "success": true,
    "data": [
        {
            "match_date": "string",
            "events": ["..."],
            "count": 0
        }
    ],
    "total": 0
}
```

---

### GET /events/v2

> List events with DB-level sorting and real 24h volume tracking. Uses event_metrics_snapshots for accurate volume data.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type    | Required | Description                                                                                                                                                                                                                                                                                                                                            |
| --------------- | ------ | ------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| authorization   | header | string  | no       |                                                                                                                                                                                                                                                                                                                                                        |
| accept-language | header | string  | no       |                                                                                                                                                                                                                                                                                                                                                        |
| keyword         | query  | string  | no       | Search keyword                                                                                                                                                                                                                                                                                                                                         |
| tags            | query  | string  | no       | Tag slug                                                                                                                                                                                                                                                                                                                                               |
| categories      | query  | string  | no       | Category slug                                                                                                                                                                                                                                                                                                                                          |
| collections     | query  | string  | no       | Collection slug                                                                                                                                                                                                                                                                                                                                        |
| purpose         | query  | string  | no       | Purpose for the event listing. `top`: lists all top events. `featured` lists all featured events. `regular` lists out all non-top and non-featured events. `recommended` lists all events available.                                                                                                                                                   |
| status          | query  | string  | no       | Filter events by status. `open`: currently open events, `closing-soon`: events whose ends_at date is within 24 hours UTC of current time, `pending-resolution`: events with status "closed" or "resolving" but not yet resolved to any outcome, `resolved`: events that have been resolved to an outcome, `cancelled`: events that have been cancelled |
| has_stream_url  | query  | boolean | no       | Filter events that have a stream URL                                                                                                                                                                                                                                                                                                                   |
| sortby          | query  | string  | no       | Order By                                                                                                                                                                                                                                                                                                                                               |
| limit           | query  | integer | no       |                                                                                                                                                                                                                                                                                                                                                        |
| page            | query  | integer | no       |                                                                                                                                                                                                                                                                                                                                                        |
| pagination      | query  | boolean | no       |                                                                                                                                                                                                                                                                                                                                                        |
| match_date      | query  | string  | no       |                                                                                                                                                                                                                                                                                                                                                        |
| timezone        | query  | string  | no       | Timezone for match_date filtering (e.g., 'America/New_York', 'Asia/Tokyo'). Defaults to UTC.                                                                                                                                                                                                                                                           |

**Response 200:**

```json
{
    "success": true,
    "data": [
        {
            "_id": "string",
            "code": "string",
            "ticker": "string",
            "group_id": "string",
            "name": "string",
            "description": "string",
            "rules": "string",
            "status": "string",
            "ends_at": "string",
            "match_date": "string",
            "volume": 0,
            "participants_count": 0,
            "closed_at": "string",
            "category": {
                "slug": "...",
                "name": "...",
                "icon": "...",
                "num_events": "..."
            },
            "sub_category": {
                "slug": "...",
                "name": "...",
                "icon": "...",
                "num_events": "..."
            },
            "featured_tag": {
                "slug": "...",
                "name": "...",
                "icon": "...",
                "num_events": "..."
            },
            "collections": ["..."],
            "tags": ["..."],
            "outcomes": ["..."],
            "top_event_images_url": ["..."],
            "event_images_url": ["..."],
            "og_image_url": "string",
            "stream_url": "string",
            "is_top_events": true,
            "is_featured_events": true,
            "recommended_images_url": ["..."]
        }
    ],
    "tagData": ["string"],
    "pagination": {
        "currentPage": 0,
        "totalPages": 0,
        "totalCount": 0,
        "hasMore": true,
        "pageSize": 0,
        "itemsCount": 0
    },
    "meta": {
        "ts": "string",
        "exec_time": 0,
        "request_id": "string"
    }
}
```

---

### GET /events/{event_id}

> Public event details with outcomes and trading info

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name     | In   | Type   | Required | Description |
| -------- | ---- | ------ | -------- | ----------- |
| event_id | path | string | yes      |             |

---

### GET /user/favourite

> Retrieve a list of the user's favorite events with event details

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name           | In     | Type    | Required | Description                                                                                                                                                                                                                                                                                                                                            |
| -------------- | ------ | ------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| authorization  | header | string  | yes      |                                                                                                                                                                                                                                                                                                                                                        |
| keyword        | query  | string  | no       | Search keyword for the event listing                                                                                                                                                                                                                                                                                                                   |
| tags           | query  | string  | no       | Search by tag(s). Use comma to separate multiple tags.                                                                                                                                                                                                                                                                                                 |
| purpose        | query  | string  | no       | Purpose for the event listing. `top`: lists all top events. `featured` lists all featured events. `regular` lists out all non-top and non-featured events. `recommended` lists all events available.                                                                                                                                                   |
| status         | query  | string  | no       | Filter events by status. `open`: currently open events, `closing-soon`: events whose ends_at date is within 24 hours UTC of current time, `pending-resolution`: events with status "closed" or "resolving" but not yet resolved to any outcome, `resolved`: events that have been resolved to an outcome, `cancelled`: events that have been cancelled |
| has_stream_url | query  | boolean | no       | Filter events that have a stream URL                                                                                                                                                                                                                                                                                                                   |
| sortby         | query  | string  | no       | Order By                                                                                                                                                                                                                                                                                                                                               |
| limit          | query  | integer | no       |                                                                                                                                                                                                                                                                                                                                                        |
| page           | query  | integer | no       |                                                                                                                                                                                                                                                                                                                                                        |
| pagination     | query  | boolean | no       |                                                                                                                                                                                                                                                                                                                                                        |
| format         | query  | string  | no       | Response format: full (default) or ids (only event codes)                                                                                                                                                                                                                                                                                              |

---

### GET /events/{event_id}/recent-wagers

> Event All Outcomes Recent Wagers

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description |
| ------------- | ------ | ------- | -------- | ----------- |
| authorization | header | string  | no       |             |
| event_id      | path   | string  | yes      |             |
| limit         | query  | integer | no       |             |
| page          | query  | integer | no       |             |
| pagination    | query  | boolean | no       |             |

**Response 200:**

```json
[
    {
        "event_id": "string",
        "event_outcome_id": "string",
        "datetime": "string",
        "probability": 0,
        "wager": 0,
        "pledge": 0,
        "leverage": 0,
        "expected_payout": 0,
        "user_email": "string"
    }
]
```

---

### GET /events/{eventId}/comments

> List all comments for a specific event

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type    | Required | Description                    |
| --------------- | ------ | ------- | -------- | ------------------------------ |
| authorization   | header | string  | no       |                                |
| accept-language | header | string  | no       |                                |
| eventId         | path   | string  | yes      | Event ID                       |
| limit           | query  | integer | no       |                                |
| page            | query  | integer | no       |                                |
| pagination      | query  | boolean | no       |                                |
| sortby          | query  | string  | no       | Sort comments by creation time |

**Response 200:**

```json
[
    {
        "id": "string",
        "user_id": "string",
        "display_name": "string",
        "content": "string",
        "timestamp": "2024-01-01T00:00:00Z",
        "eventId": "string"
    }
]
```

---

### POST /events/{eventId}/comments

> Create a new comment for a specific event

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type   | Required | Description     |
| --------------- | ------ | ------ | -------- | --------------- |
| authorization   | header | string | yes      |                 |
| accept-language | header | string | no       |                 |
| eventId         | path   | string | yes      | Event ID        |
| content         | body   | string | yes      | Comment content |

**Request Body:**

```json
{
    "content": "string"
}
```

**Response 201:**

```json
{
    "id": "string",
    "user_id": "string",
    "display_name": "string",
    "content": "string",
    "timestamp": "2024-01-01T00:00:00Z",
    "eventId": "string"
}
```

---

### GET /events/{event_id}/history

> Event Outcomes Market History of Probability, Estimated Payout and Wagers. Supports precision filtering (1hour, 6hour, 1day, 1week, 1month, all) to get consistent data points for each outcome.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name                     | In     | Type    | Required | Description                                                                                                                                                                                                                         |
| ------------------------ | ------ | ------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| authorization            | header | string  | no       |                                                                                                                                                                                                                                     |
| event_id                 | path   | string  | yes      |                                                                                                                                                                                                                                     |
| focused_event_outcome_id | query  | string  | no       | To request for a focused event outcome histories. The response data points would contain two invisible outcomes (`MIN` and `MAX`), representing two range lines.                                                                    |
| precision                | query  | string  | yes      | Precision for datapoints. 1hour=60 points (1min intervals), 6hour=360 points (1min intervals), 1day=288 points (5min intervals), 1week=84 points (30min intervals), 1month=8\*days points (3hour intervals), all=adaptive intervals |
| \_clear_cache            | query  | boolean | no       | Set to true to clear Redis cache for this event before fetching data                                                                                                                                                                |

**Response 200:**

```json
{
    "history": [
        {
            "t": 0,
            "p": 0
        }
    ],
    "metadata": {
        "filter": "string",
        "dataPoints": 0,
        "interval": "string",
        "from": "string",
        "to": "string"
    }
}
```

---

### GET /events/{event_id}/related-events

> Get events that share tags with the given event. Uses optimized V2 query.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name            | In     | Type    | Required | Description                                               |
| --------------- | ------ | ------- | -------- | --------------------------------------------------------- |
| authorization   | header | string  | no       |                                                           |
| accept-language | header | string  | no       |                                                           |
| event_id        | path   | string  | yes      |                                                           |
| sortby          | query  | string  | no       | Order By                                                  |
| limit           | query  | integer | no       |                                                           |
| page            | query  | integer | no       |                                                           |
| pagination      | query  | boolean | no       |                                                           |
| format          | query  | string  | no       | Response format: full (default) or ids (only event codes) |

---

### GET /user/favourite/event-ids

> List Favorite Event IDs

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

---

### GET /user/favourite/{event_id}

> Add an event to the user’s favorite list

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                         |
| ------------- | ------ | ------ | -------- | ----------------------------------- |
| authorization | header | string | yes      |                                     |
| event_id      | path   | string | yes      | ID of the event to add to favorites |

---

### DELETE /user/favourite/{event_id}

> Remove an event from the user’s favorite list

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                              |
| ------------- | ------ | ------ | -------- | ---------------------------------------- |
| authorization | header | string | yes      |                                          |
| event_id      | path   | string | yes      | ID of the event to remove from favorites |

---

### GET /events/{eventId}/comments/{commentId}

> Retrieve a specific comment for an event by its ID

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name      | In   | Type   | Required | Description |
| --------- | ---- | ------ | -------- | ----------- |
| eventId   | path | string | yes      | Event ID    |
| commentId | path | string | yes      | Comment ID  |

**Response 200:**

```json
{
    "id": "string",
    "user_id": "string",
    "display_name": "string",
    "content": "string",
    "timestamp": "2024-01-01T00:00:00Z",
    "eventId": "string"
}
```

---

### DELETE /events/{eventId}/comments/{commentId}

> Delete a specific comment for an event

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name      | In   | Type   | Required | Description |
| --------- | ---- | ------ | -------- | ----------- |
| eventId   | path | string | yes      | Event ID    |
| commentId | path | string | yes      | Comment ID  |

---

### PUT /events/{eventId}/comments/{commentId}

> Update a specific comment for an event

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name      | In   | Type   | Required | Description     |
| --------- | ---- | ------ | -------- | --------------- |
| eventId   | path | string | yes      | Event ID        |
| commentId | path | string | yes      | Comment ID      |
| content   | body | string | yes      | Comment content |

**Request Body:**

```json
{
    "content": "string"
}
```

**Response 200:**

```json
{
    "id": "string",
    "user_id": "string",
    "display_name": "string",
    "content": "string",
    "timestamp": "2024-01-01T00:00:00Z",
    "eventId": "string"
}
```

---

### GET /events/{event_id}/outcomes/{event_outcome_id}/recent-wagers

> Event Outcome Recent Wagers

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name             | In     | Type    | Required | Description |
| ---------------- | ------ | ------- | -------- | ----------- |
| authorization    | header | string  | no       |             |
| event_id         | path   | string  | yes      |             |
| event_outcome_id | path   | string  | yes      |             |
| limit            | query  | integer | no       |             |
| page             | query  | integer | no       |             |
| pagination       | query  | boolean | no       |             |

**Response 200:**

```json
[
    {
        "event_id": "string",
        "event_outcome_id": "string",
        "datetime": "string",
        "probability": 0,
        "wager": 0,
        "pledge": 0,
        "leverage": 0,
        "expected_payout": 0,
        "user_email": "string"
    }
]
```

---

### GET /events/{event_id}/outcomes/{event_outcome_id}/history

> Event Outcome Market History of Probability, Estimated Payout and Wagers

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name             | In     | Type   | Required | Description              |
| ---------------- | ------ | ------ | -------- | ------------------------ |
| authorization    | header | string | no       |                          |
| event_id         | path   | string | yes      |                          |
| event_outcome_id | path   | string | yes      |                          |
| precision        | query  | string | yes      | Precision for datapoints |
| from             | query  | string | yes      |                          |
| to               | query  | string | yes      |                          |

**Response 200:**

```json
[
    {
        "event_id": "string",
        "event_outcome_id": "string",
        "datetime": "string",
        "probability": 0,
        "estimated_payout": 0,
        "indicative_return": 0,
        "num_wagers": 0,
        "sum_wagers": 0
    }
]
```

---

### GET /events/{event_id}/outcomes/{event_outcome_id}/wagers-by-probability

> Event Outcome Market Wager Summary by Probability

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name             | In     | Type   | Required | Description |
| ---------------- | ------ | ------ | -------- | ----------- |
| authorization    | header | string | no       |             |
| event_id         | path   | string | yes      |             |
| event_outcome_id | path   | string | yes      |             |

**Response 200:**

```json
[
    {
        "event_id": "string",
        "event_outcome_id": "string",
        "probability": 0,
        "num_wagers": 0
    }
]
```

---

## Dashboard

### GET /activities

> List Activities of the current login User

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description |
| ------------- | ------ | ------- | -------- | ----------- |
| authorization | header | string  | yes      |             |
| limit         | query  | integer | no       |             |
| page          | query  | integer | no       |             |
| pagination    | query  | boolean | no       |             |

**Response 200:**

```json
[
    {
        "_id": "string",
        "datetime": "string",
        "user_id": "string",
        "activity": "string",
        "target": "string",
        "admin_id": "string"
    }
]
```

---

### GET /dashboard

> User Dashboard Stats

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

**Response 200:**

```json
{
    "user_id": "string",
    "positions": 0,
    "fund_available": 0,
    "profit": 0,
    "best_case_payoff": 0,
    "best_case_cumulative_profit": 0,
    "expected_payoff": 0,
    "withdrawable_balance": 0,
    "topup_balance": 0,
    "bonus_balance": 0,
    "profit_balance": 0,
    "timestamp": "string"
}
```

---

### GET /dashboard/wager-position-events

> List Current User Trader Positions in Events

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description                                                                                                                                      |
| ------------- | ------ | ------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| authorization | header | string  | yes      |                                                                                                                                                  |
| status        | query  | string  | no       | Status filter                                                                                                                                    |
| sortby        | query  | string  | no       | Order By. `most recently traded` and `time to expiration` are applicable to open positions. `recently closed` is applicable to closed positions. |
| limit         | query  | integer | no       |                                                                                                                                                  |
| page          | query  | integer | no       |                                                                                                                                                  |
| pagination    | query  | boolean | no       |                                                                                                                                                  |

**Response 200:**

```json
[
    {
        "type": "string",
        "id": "string",
        "trader_id": "string",
        "user_id": "string",
        "event_id": "string",
        "event_status": "string",
        "event_images_url": ["string"],
        "event": {
            "_id": "string",
            "code": "string",
            "ticker": "string",
            "name": "string",
            "description": "string",
            "ends_at": "string",
            "closed_at": "string",
            "status": "string",
            "resolved_outcome_id": "string",
            "outcomes": ["..."],
            "event_images_url": ["..."]
        },
        "wagered_at": "string",
        "pledge": 0,
        "is_leveraged": true,
        "wager": 0,
        "indicative_return": 0,
        "indicative_payout": 0,
        "unrealized_pnl": 0,
        "indicative_pnl": 0,
        "payout_return_for_new_wagers": 0,
        "probability": 0,
        "stop_probability": 0,
        "realized_return": 0,
        "realized_payout": 0,
        "realized_payoff": 0,
        "realized_pnl": 0,
        "final_indicative_return": 0,
        "fill_count": 0,
        "entry_probability": 0,
        "current_probability": 0,
        "probability_change": 0,
        "created_at": "string",
        "last_update_time": "string",
        "last_reason": "string",
        "positions": [
            {
                "type": "...",
                "id": "...",
                "trader_id": "...",
                "user_id": "...",
                "event_id": "...",
                "event_status": "...",
                "event_outcome_id": "...",
                "event": "...",
                "event_outcome": "...",
                "wagered_at": "...",
                "pledge": "...",
                "is_leveraged": "...",
                "leverage": "...",
                "wager": "...",
                "indicative_return": "...",
                "indicative_payout": "...",
                "unrealized_pnl": "...",
                "indicative_pnl": "...",
                "payout_return_for_new_wagers": "...",
                "probability": "...",
                "stop_probability": "...",
                "realized_return": "...",
                "realized_payout": "...",
                "realized_payoff": "...",
                "realized_pnl": "...",
                "final_indicative_return": "...",
                "fill_count": "...",
                "entry_probability": "...",
                "current_probability": "...",
                "probability_change": "...",
                "created_at": "...",
                "last_update_time": "...",
                "last_reason": "...",
                "status": "...",
                "stop_warning": "...",
                "stop_distance": "...",
                "wager_position_id": "...",
                "wager_id": "..."
            }
        ]
    }
]
```

---

### GET /dashboard/wager-positions

> List Current User Trader Positions

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type    | Required | Description                                                                                                                                      |
| ------------- | ------ | ------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| authorization | header | string  | yes      |                                                                                                                                                  |
| status        | query  | string  | no       | Status filter                                                                                                                                    |
| sortby        | query  | string  | no       | Order By. `most recently traded` and `time to expiration` are applicable to open positions. `recently closed` is applicable to closed positions. |
| limit         | query  | integer | no       |                                                                                                                                                  |
| page          | query  | integer | no       |                                                                                                                                                  |
| pagination    | query  | boolean | no       |                                                                                                                                                  |

**Response 200:**

```json
[
    {
        "type": "string",
        "id": "string",
        "trader_id": "string",
        "user_id": "string",
        "event_id": "string",
        "event_status": "string",
        "event_outcome_id": "string",
        "event": {
            "_id": "string",
            "code": "string",
            "ticker": "string",
            "name": "string",
            "description": "string",
            "ends_at": "string",
            "closed_at": "string",
            "status": "string",
            "resolved_outcome_id": "string",
            "outcomes": ["..."],
            "event_images_url": ["..."]
        },
        "event_outcome": {
            "_id": "string",
            "code": "string",
            "index": 0,
            "event_id": "string",
            "name": "string",
            "description": "string"
        },
        "wagered_at": "string",
        "pledge": 0,
        "is_leveraged": true,
        "leverage": 0,
        "wager": 0,
        "indicative_return": 0,
        "indicative_payout": 0,
        "unrealized_pnl": 0,
        "indicative_pnl": 0,
        "payout_return_for_new_wagers": 0,
        "probability": 0,
        "stop_probability": 0,
        "realized_return": 0,
        "realized_payout": 0,
        "realized_payoff": 0,
        "realized_pnl": 0,
        "final_indicative_return": 0,
        "fill_count": 0,
        "entry_probability": 0,
        "current_probability": 0,
        "probability_change": 0,
        "created_at": "string",
        "last_update_time": "string",
        "last_reason": "string",
        "status": 0,
        "stop_warning": true,
        "stop_distance": 0,
        "wager_position_id": "string",
        "wager_id": "string"
    }
]
```

---

### GET /dashboard/wager-positions/{wager_position_id}

> Get Current User Trader Position

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name              | In     | Type   | Required | Description |
| ----------------- | ------ | ------ | -------- | ----------- |
| authorization     | header | string | yes      |             |
| wager_position_id | path   | string | yes      |             |

**Response 200:**

```json
{
    "type": "string",
    "id": "string",
    "trader_id": "string",
    "user_id": "string",
    "event_id": "string",
    "event_status": "string",
    "event_outcome_id": "string",
    "event": {
        "_id": "string",
        "code": "string",
        "ticker": "string",
        "name": "string",
        "description": "string",
        "ends_at": "string",
        "closed_at": "string",
        "status": "string",
        "resolved_outcome_id": "string",
        "outcomes": [
            {
                "_id": "...",
                "code": "...",
                "event_id": "...",
                "name": "...",
                "description": "...",
                "image_url": "...",
                "disabled": "...",
                "trader_info": "...",
                "histories": "...",
                "has_wagered": "...",
                "has_cash_wagered": "...",
                "has_leveraged_wagered": "..."
            }
        ],
        "event_images_url": ["string"]
    },
    "event_outcome": {
        "_id": "string",
        "code": "string",
        "index": 0,
        "event_id": "string",
        "name": "string",
        "description": "string"
    },
    "wagered_at": "string",
    "pledge": 0,
    "is_leveraged": true,
    "leverage": 0,
    "wager": 0,
    "indicative_return": 0,
    "indicative_payout": 0,
    "unrealized_pnl": 0,
    "indicative_pnl": 0,
    "payout_return_for_new_wagers": 0,
    "probability": 0,
    "stop_probability": 0,
    "realized_return": 0,
    "realized_payout": 0,
    "realized_payoff": 0,
    "realized_pnl": 0,
    "final_indicative_return": 0,
    "fill_count": 0,
    "entry_probability": 0,
    "current_probability": 0,
    "probability_change": 0,
    "created_at": "string",
    "last_update_time": "string",
    "last_reason": "string",
    "status": 0,
    "stop_warning": true,
    "stop_distance": 0,
    "wager_position_id": "string",
    "wager_id": "string"
}
```

---

## Deposits

### GET /deposit/create/wallet

> Create a new wallet

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description |
| ------------- | ------ | ------ | -------- | ----------- |
| authorization | header | string | yes      |             |

**Response 200:**

```json
{
    "message": "string",
    "address": "string"
}
```

---

### POST /deposit/create/transaction

> Create a new transaction

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In       | Type   | Required | Description |
| ------------- | -------- | ------ | -------- | ----------- |
| authorization | header   | string | yes      |             |
| txnHash       | formData | string | yes      |             |
| amount        | formData | string | yes      |             |
| network       | formData | string | yes      |             |

**Response 201:**

```json
{
    "message": "string",
    "transaction_hash": "string",
    "network": "string",
    "status": "string"
}
```

---

## Withdrawals

### POST /withdrawal/preflight

> Advisory endpoint to check OTP requirements before initiating withdrawal. Checks phone verification status and TTL expiry. Does not block withdrawal API - clients should use this for UX optimization.

| Security       |                                           |
| -------------- | ----------------------------------------- |
| Auth           | JWT Bearer Token                          |
| Fingerprint    | Conditional (rule-based)                  |
| Developer Auth | Compatible (bypasses fingerprint/captcha) |

**Parameters:**

| Name          | In     | Type   | Required | Description                                |
| ------------- | ------ | ------ | -------- | ------------------------------------------ |
| authorization | header | string | yes      |                                            |
| amount        | body   | number | no       | Withdrawal amount (optional, for logging)  |
| network       | body   | string | no       | Network identifier (optional, for logging) |

**Request Body:**

```json
{
    "amount": 0,
    "network": "string"
}
```

**Response 200:**

```json
{
    "otp_required": true,
    "verification_method": "string",
    "reason": "string",
    "last_verified_at": "string",
    "verification_expires_at": "string",
    "can_proceed": true,
    "message": "string"
}
```

---

### POST /withdrawal/request

> User can request withdrawal of funds from their wallet using CORATIO-based calculation and sequential wallet deduction

| Security |               |
| -------- | ------------- |
| Auth     | None (public) |

**Parameters:**

| Name    | In   | Type   | Required | Description                      |
| ------- | ---- | ------ | -------- | -------------------------------- |
| amount  | body | number | yes      | Amount to withdraw               |
| network | body | string | yes      | Network type                     |
| address | body | string | yes      | Withdrawal address               |
| token   | body | string | no       | Verification token (OTP or TOTP) |

**Request Body:**

```json
{
    "amount": 0,
    "network": "string",
    "address": "string",
    "token": "string"
}
```

**Response 200:**

```json
{
    "success": true,
    "message": "string",
    "txnHash": "string",
    "status": "string",
    "transactionIds": [0],
    "pendingWithdrawalId": 0,
    "withdrawableBalance": 0,
    "breakdown": {},
    "deductionBreakdown": {}
}
```

---
