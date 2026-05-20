# Quick Tests

To quickly verify that the service works correctly, you can run a few simple command-line requests instead of using Postman.

After starting the application, you can run the following commands to send requests to your service and quickly verify that it responds correctly.

This is useful for a fast smoke test of the main API flow.


## 1. User Registration
```bash
curl --location 'http://localhost:8080/v1/users/registration' \
--header 'Content-Type: application/json' \
--data-raw '{
  "email": "jane.doe@example.com",
  "password": "S3cure!Passw0rd",
  "name": "Jane Doe"
}'

# {
#   "id": 1,
#   "email": "jane.doe@example.com",
#   "name": "Jane Doe",
#   "email_verified": false,
#   "email_verified_at": null,
#   "created_at": "2026-05-19T23:58:43.122849",
#   "updated_at": "2026-05-19T23:58:43.122849"
# }
```

## 2. Email Verification

After completing the request, you will receive an email with a confirmation link. Open this link to verify the email address.

If you are mocking email sending, you can manually generate the verification link and open it yourself. To do this, insert the generated `token` into the verification request.

```bash
curl --location 'http://localhost:8080/v1/users/email-verification/{token}'
```

## 3. User Login
```bash
curl --location 'http://localhost:8080/v1/users/login' \
--header 'Content-Type: application/json' \
--data-raw '{
  "email": "jane.doe@example.com",
  "password": "S3cure!Passw0rd"
}'

# {
#   "access_token": "eyJ***KcM",
#   "token_type": "Bearer",
#   "expires_in": 3600,
#   "user": {
#     "id": 1,
#     "email": "jane.doe@example.com",
#     "name": "Jane Doe",
#     "email_verified": true,
#     "email_verified_at": "2026-05-20T00:00:28",
#     "created_at": "2026-05-19T23:58:43",
#     "updated_at": "2026-05-20T00:00:28"
#   }
# }
```

## 4. Send Event

For this request, replace `access_token` with the token received after registration or login.

```bash
curl --location 'http://localhost:8080/v1/events/add?user_id=1&event_name=add_to_cart' \
--header 'Authorization: Bearer {access_token}'

```
