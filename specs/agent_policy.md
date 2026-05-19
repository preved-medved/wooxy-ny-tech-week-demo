# AI Code Generation Rules

These rules define strict limitations for the AI assistant when generating or modifying project code.

---

## 1. Directory Restrictions
- The AI **must not modify, delete, or overwrite any files** inside the `specs/` and `tests/` directories.
- The AI **must not alter** the project’s database schema or make any structural changes to existing tables or models.
- All newly generated code, modules, or services **must be placed inside the `src/` directory**.

---

## 2. General Guidelines
- The AI must respect the existing project architecture, naming conventions, and folder structure.
- The AI must ensure that generated code passes linting, type checking, and follows standards.

---

## 3. Database Restrictions
- The AI must not create or modify database migrations.
- The AI must not change table structures, ORM models, relations, indexes, or constraints unless explicitly requested.
- The AI must not assume that schema changes are allowed.

---

## 4. Security Rules
- The AI must not hardcode secrets, API keys, passwords, or tokens.
- The AI must not expose internal credentials, private URLs, or sensitive configuration values.
