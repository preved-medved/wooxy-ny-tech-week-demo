# Tech Stack & Engineering Rules

## 1. Technology Stack

| Layer | Technology |
|--------|-------------|
| Language | Python 3.12 |
| Frameworks | FastAPI (async), SQLAlchemy (async) |
| Database | MySQL 8.x |
| Deployment | Docker, Docker Compose |
| Logging | Python `logging` module with JSON-formatted logs |

---

## 2. Database Interaction Rules

- All database queries must use the ORM (SQLAlchemy) — avoid raw or native SQL statements.
- When multiple tables are modified within a single operation, the process must be atomic — wrap all such operations in transactions.
- If the database connection is lost, the daemon must automatically attempt to reconnect using a backoff strategy and without requiring a manual restart of the service.

---

## 3. Input Parameters Validation Rules

- Never trust incoming data by default. All external input must be considered potentially unsafe, invalid, or a security risk.
- Always perform strict validation and type enforcement using Pydantic models or equivalent schema-based validation.

---

## 4. Async Rules
- All I/O operations must be asynchronous.
- Do not use blocking calls (e.g., time.sleep, synchronous DB drivers).
- Use async SQLAlchemy sessions.

---

## 5. Password Hashing
- Passwords are stored as SHA-256 hex digests (`hashlib.sha256`) in `users.password_hash`.

---

## 6. Wooxy Integration

External operations (email validation, sending emails, adding contacts, forwarding events) go through `https://WOOXY_API_URL`. The `Access-Token` header is loaded from env; requests use an async client from `src/`.

- **Email validation** — `POST /v3/validator/email/validate`
  ```bash
  curl -X POST 'https://WOOXY_API_URL/v3/validator/email/validate'
    -H "Access-Token: WOOXY_API_KEY" \
    -d '{"email": "kirkillbox@gmail.com"}'
    
  # Example response:
  # {
  #   "result": true,
  #   "data": {
  #     "id": 2,
  #     "email": "user@example.com",
  #     "stage": "done",
  #     "status": "valid",
  #     "comment": ""
  #   }
  # }
  ```
  Registration is rejected if `result != true` or `data.status != "valid"`.
  More details: https://wooxy.com/api-documentation/email-validation/email-validation

- **Send verification email** — `POST /v3/mailer/send`
  ```bash
  curl -X POST "https://WOOXY_API_URL/v3/mailer/send" \
    -H "Access-Token: WOOXY_API_KEY" -H "Content-Type: application/json" \
    -d '{
        "from": {"email": "APP_MAIL_FROM_EMAIL", "name": "APP_MAIL_FROM_NAME"},
        "to":   {"email": "user@example.com"},
        "subject": "Confirm your email",
        "html": "<p>Please confirm your email:</p><a href=\"{EMAIL_VERIFICATION_URL}\">Confirm Email</a>",
        "text": "Please confirm your email: {EMAIL_VERIFICATION_URL}"
    }'

  # Example response:
  # {
  #   "result": true,
  #   "messageId": "6a03a0d13ebb1191a90a2d18"
  # }
  ```
  More details: https://wooxy.com/api-documentation/email/send-email

- **Add contact** — `POST /v3/contacts/add`
  ```bash
  curl -X POST "https://WOOXY_API_URL/v3/contacts/add" \
    -H "Access-Token: WOOXY_API_KEY" -H "Content-Type: application/json" \
    -d '{
        "contactListId": "WOOXY_CONTACT_LIST_ID",
        "contacts": [
          {"email": "user@example.com"}
        ]
    }'

  # Example success response:
  # {
  #   "result": true,
  #   "requestId": "6a0641f24232f056fd023ee4",
  #   "errors": []
  # }
  ```
  Adds a new contact to the Wooxy contacts database so it can be used in further operations (event tracking, email campaigns, etc.). A request is considered successful only if `result == true`.

- **Forward events** — `POST /v3/custom-event/add`
  ```bash
    curl -X POST "https://WOOXY_API_URL/v3/custom-event/add" \
        -H "Access-Token: WOOXY_API_KEY" -H "Content-Type: application/json" \
        -d '{
            "domain": "senderDomain.com",
            "customEvent": "YOUR_REGISTERED_EVENT_ID",
            "contact": "user@example.com"
        }'

  # Example success response:
  # {
  #   "result": true
  # }
  ```
  A request is considered successful only if `result == true`.

  Example error response:
  ```
  {
    "result": false,
    "errors": [
      "Custom event YOUR_REGISTERED_EVENT_ID not found in your account"
    ]
  }
  ```
  