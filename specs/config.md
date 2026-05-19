# Configuration Specification

This project does **not** require a configuration file —
all runtime settings must be provided via **environment variables**.

---

## Core Application Settings

| Variable | Required | Type | Description | Example |
|-----------|-----------|------|-------------|----------|
| `APP_DOMAIN` | Yes | string | Current application domain. | `api.example.com` |
| `VERIFICATION_PAGE_REDIRECT` | Yes | string | Page URL for redirecting the user after email verification. | `http://example.com` |
| `APP_JWT_SECRET_KEY` | Yes | string | Secret key used to validate JWT tokens. | `supersecret123` |
| `APP_JWT_ALGORITHM` | Yes | string | JWT signing algorithm. | `HS256` |
| `APP_JWT_LIFETIME`  | Yes | integer | JWT token lifetime in seconds. | `3600` |
| `APP_ALLOW_ORIGIN` | Yes | comma-separated list | Allowed CORS origins. | `http://example.com` |
| `APP_PORT` | Yes | integer | Port for running the daemon. | `8080` |
| `APP_MAIL_FROM_EMAIL` | Yes | string | Email address used as the sender for outgoing application emails. | `noreply@example.com` |
| `APP_MAIL_FROM_NAME` | Yes | string | Display name used as the sender name for outgoing application emails. | `Example App` |

---

## Database (MySQL) Settings

| Variable | Required | Type | Description | Example |
|-----------|-----------|------|-------------|----------|
| `MYSQL_HOST` | Yes | string | MySQL server hostname or IP. | `127.0.0.1` |
| `MYSQL_PORT` | No | integer | MySQL port. Defaults to `3306`. | `3306` |
| `MYSQL_USER` | Yes | string | Database username. | `db_user` |
| `MYSQL_PWD` | Yes | string | Database password. | `db_pass` |
| `MYSQL_DB` | Yes | string | Database name. | `wooxy_integration_db` |
| `MYSQL_CONNECT_TIMEOUT` | No | integer | Connection timeout in seconds. Defaults to `30`. | `30` |

---

## Wooxy API

| Variable | Required | Type | Description | Example |
|-----------|-----------|------|-------------|----------|
| `WOOXY_API_URL` | Yes | string | Main domain of Wooxy API. | `api.wooxy.com` |
| `WOOXY_API_KEY` | Yes | string | Private key for API. | `******` |
| `WOOXY_CONTACT_LIST_ID` | Yes | string | ID of the Contact list in which we are going to collect users. | 69ecc9229f57d3ac780bdf00 |
