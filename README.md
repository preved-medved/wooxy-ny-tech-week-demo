# Spec Driven Development: Architect-Guided Approach — Quick Start & Reference Implementation

## Overview

This project demonstrates a practical implementation of the **Spec Driven Development (SDD)** approach — a modern methodology that shifts the center of software development from code to **specifications as the single source of truth**.

Instead of traditional coding-first workflows, this approach enables teams — including QA specialists and non-developers — to actively participate in building microservices through well-defined specifications. This significantly **reduces development cost and time**, while improving consistency and scalability.

### Key ideas behind the project:

- **Specification-first development**. All system behavior is defined through structured specs. Code becomes a derivative artifact.
- **Human-in-the-loop architecture**. Developers focus on high-level decisions — system architecture, APIs, and data models — while implementation details can be generated or derived from specifications.
- **Ideal for microservices**. Architects can design entire systems at a high level, then decompose them into independent, spec-driven components for parallel development.
- **Specification as the source of truth**. Code is not manually controlled in the traditional sense — correctness and behavior are governed by the specification itself.

## Project Structure

This repository contains all the **required documentation to start development** using the Architect-Guided SDD approach.

The core entities of the project are `specs/` and `tests/`.

- `specs/` — written by the architect (human-in-the-loop), defining what should be built and how it should be structured
- `tests/` — also defined by the architect (human-in-the-loop), used to validate that the implementation matches the specification

### `specs/`

Contains the **core specifications** that define the system.
Each file has a clearly scoped responsibility:

| File | Purpose |
|------|---------|
| `agent_policy.md` | Hard constraints for AI-generated code (read first): allowed directories, DB restrictions, security rules. |
| `product.md` | Product overview and business logic: registration, email verification, event tracking, auth, logging, error handling. |
| `openapi.yaml` | API contract — endpoints, request/response schemas, status codes. |
| `tech_stack.md` | Languages, frameworks, engineering rules, and Wooxy integration details. |
| `config.md` | Required environment variables and their meaning. |
| `init_db.sql` | Database schema and seed data. |

> This is the **single source of truth**.
> Everything in the system — including code and tests — is derived from these specifications.

### `tests/`

Contains **validation of the specifications**.

- Functional scenarios based on specs
- Edge cases and negative cases
- Contract validation (API behavior)

Tests are written in **Postman**, which allows even non-developers (e.g. QA specialists) to create and maintain test scenarios without deep programming knowledge.

> Tests ensure that the implementation strictly follows the specifications.
> They validate spec compliance, not handcrafted code, and also enforce security and vulnerability checks as part of the development process.

## Getting Started (Step-by-step)

### 1. Open an AI agent

Use any coding agent:
- Claude Code
- Gemini
- or any similar tool

### 2. Analyze the specification and generate a plan

Ask the agent to read the specs and break down the work.

Prompt:
```
Read the project specifications in the @specs/ folder.

Based on the specifications: describe the architecture and define the implementation plan step by step

Do not write code yet.
```

### 3. Review the plan and generate the implementation

Carefully review the generated implementation plan.

If the plan looks correct and you don’t see any obvious issues, proceed with implementation.

Prompt:
```
Based on the implementation plan and specifications in /specs:

- implement the project
- follow the specifications strictly

Do not invent behavior outside the specification.
```

### 4. Run the project

```
docker-compose up --build
```

### 5. Run tests (Postman / Newman)

Tests can be executed directly from the command line using Newman.

```
newman run stage-1-create-user.postman_collection.json -e wooxy-local.postman_environment.json
newman run stage-2-e2e.postman_collection.json -e wooxy-local.postman_environment.json
```

> Unfortunately, the tests had to be split into two files because the E2E scenario requires a token from the email.
> To keep the test flow simple and easy to understand, the tests were divided into two separate runs.
> You need to extract the `token` from the verification link received by email and add it to the environment variables in Postman.

These are the same tests that are written by the architect / QA (human-in-the-loop) as part of the specification process.

- Validates API behavior against the specification
- Covers functional, edge, and negative scenarios
- Ensures contract compliance

> The goal is to verify that the implementation strictly follows the specification.

### Breakpoint: First Run

At this stage, your project is **theoretically ready** and can already be used.

However, in practice, things rarely work perfectly on the first attempt.

You will likely encounter:
- runtime errors (e.g. 500 responses)
- failing tests
- missing or unclear logic in the specs

> This is expected.

From this point on, the process becomes **iterative**:

- fix errors
- refine the specification
- align implementation with the spec

### 6. Iterate: fix errors and stabilize the system

At this stage, expect failures — this is part of the process.

When you encounter runtime errors (e.g. 500 responses), do not debug manually. Instead, copy the error directly into the agent and ask for a fix.

Prompt:
```
I got the following error while running the project:

<PASTE ERROR LOG>

Analyze the issue and fix it according to the specifications.
```

- Fix issues **one by one**
- Repeat the process if multiple errors appear
- **Run tests after each fix** to verify progress

> The goal is not to manually patch the code, but to guide the system toward stability through iterative corrections.

### 7. Iterate via specification (fix → compare → align)

If the behavior does not match expectations or tests fail:

**Do not fix the code directly. Update the specification first.**

- clarify unclear logic
- fix inconsistencies
- extend or correct functionality

#### Step 1: Compare implementation with updated specs

Ask the agent to detect mismatches.

Prompt:
```
I updated the specifications.

Compare the current implementation with the specs and:
- list all mismatches
- explain what does not comply with the specification
```

#### Step 2: Fix mismatches iteratively

Do not fix everything at once.

Pick one mismatch at a time and resolve it.

Prompt:
```
Fix the following mismatch:

<DESCRIBE ONE ISSUE>

Ensure the implementation strictly follows the specification.
```

- Repeat until all critical mismatches are resolved
- **Run tests after each fix**
- Continue refining the specification if needed

> The system evolves by aligning implementation with the specification — one iteration at a time.

## When to use this approach

This approach works best for:
- Microservices
- API-driven systems
- Rapid prototyping with production potential
- New services built from scratch (greenfield projects)
- Small, well-defined services (the smaller the scope, the better)

It may be less suitable for:
- Existing or legacy projects with established codebases
- Large monolithic systems with high complexity
- Highly dynamic, UI-heavy applications
- Frontend-driven systems where behavior is hard to fully specify