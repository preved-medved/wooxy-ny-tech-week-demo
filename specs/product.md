# Product Overview

## Overview

This project is a REST API microservice designed to be integrated into web applications or websites. It provides core user lifecycle and engagement-tracking capabilities that other products can rely on without having to implement them in-house.

The service has two primary responsibilities:

1. **User registration and email verification** — register new users in the system, issue and validate email confirmation tokens, and expose the resulting verified-user state to client applications.
2. **Event tracking** — capture arbitrary user-driven events that occur on the host site or application (page views, purchases, sign-ups, custom domain events, etc.) and persist them for downstream consumption.

The collected event data is intended to feed downstream systems such as:

- Email marketing campaigns (segmentation, triggered messages, drip flows)
- Transactional and behavioral notifications (push, SMS, in-app)
- Any other notification or automation engine that needs a reliable stream of user activity

## Registration Flow

**User Registration & Email Verification Logic**

When the registration API endpoint is called with all required fields, the service performs the following steps:

### 1. Email Validation via Wooxy API

Before creating a user record in the database, the service must validate the provided email address using the Wooxy Email Validation API.

The request is sent to:
```
POST /v3/validator/email/validate
```

Registration must be rejected if:
- `result != true`
- `data.status != "valid"`

This validation step prevents registration using invalid, disposable, malformed, or non-existent email addresses.

### 2. User Creation

If the email validation succeeds, a new record is created in the `users` table.

At this stage:
- the user account is considered **unverified**
- the `email_verified` field must be set to `0`

### 3. Verification Token Generation

After the user is created, the service generates a unique email verification token.

The generated token is stored in the `email_verification_tokens` table.

The token must:
- be cryptographically secure
- be linked to the corresponding user
- be single-use

### 4. Verification Email Delivery

After the token is generated, the service sends a verification email using the Wooxy Mailer API.

The request is sent to:
```
POST /v3/mailer/send
```

The email template contains the macro:
```
{EMAIL_VERIFICATION_URL}
```

Before sending the email, `{EMAIL_VERIFICATION_URL}` macro must be replaced with the final verification URL constructed as:
```
http: // + APP_DOMAIN + /v1/users/email-verification/ + token
```

## Email Verification Flow

After receiving the verification email, the user can confirm their email address by opening the following URL:
```
GET /v1/users/email-verification/{token}
```
Where:
- `{token}` is the email verification token previously generated during the registration process.

### Verification Process

When the endpoint is called, the service performs the following steps:
- Validates that the token exists in the email_verification_tokens table.
- Retrieves the associated user.
- Updates the corresponding record in the users table:
  - sets email_verified = 1
  - populates the email_verified_at field with the current UTC timestamp
- Deletes the corresponding token record from the email_verification_tokens table.
- Adds this user into the Wooxy Database using the "Add contact" API.

The token becomes permanently invalid after successful verification.

### Redirect Behavior

After the verification flow completes successfully, the user must be redirected to the URL specified in the environment variable:

```
VERIFICATION_PAGE_REDIRECT
```

Example:
```
http://example.com/email-verified
```

## Event Tracking Flow

The service exposes an endpoint for receiving custom user activity events from external applications or websites.

Example request:
```
GET /v1/events/add?user_id=123&event_name=visit_contact_page
```

### Processing Rules

This endpoint must not store any event data in the local database. Its only responsibility is:
- Validate the incoming request
- Enrich the payload with user data
- Forward the event to the Wooxy Custom Event API

### Wooxy Event Payload Mapping

The incoming request must be transformed into the following payload format:
```
{
  "domain": "APP_DOMAIN",
  "customEvent": "visit_contact_page",
  "contact": "test@domain.com"
}
```

### Field Mapping Rules

- `domain` - Loaded from the APP_DOMAIN configuration variable
- `customEvent` - Value from `event_name` after validation. It has to be one of the list `['visit_contact_page','add_to_cart','add_to_cart_123']`
- `contact` - User email loaded from the users table using `user_id`

## Authentication

JWT tokens are signed using `APP_JWT_SECRET_KEY`. The specific signing algorithm is determined by application configuration.

### New JWT payload format

**Example:**

```json
{
  "user_id": 123,
  "iat": 1678886400, // Issued At: Mar 15 2023 00:00:00 GMT
  "exp": 1678890000  // Expiration Time: Mar 15 2023 01:00:00 GMT (1 hour later)
}
```

**Fields:**

- `user_id` — numeric identifier of the authenticated user
- `iat` — the exact time the token was created
- `exp` — the time when the token becomes invalid

## Logging

- All http requests have to logging into strout.
- All requests sent to external third-party APIs, including all Wooxy API integrations, must be logged in case the request was not completed successfully.
- If an external API request fails or returns an unexpected response, the service must write detailed information about the failed request to `stdout` for further investigation.

## Error Handling

**Wooxy upstream failures.** Whenever any endpoint makes a request to the Wooxy API and that request fails (HTTP non-2xx, `result != true`, or network/timeout error), the service must respond with HTTP `424 Failed Dependency` and an `Error` body with `code: upstream_dependency_failed`. The failure must also be logged to stdout per the Logging rules.