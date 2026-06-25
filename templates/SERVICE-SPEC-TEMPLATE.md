<!--
  SERVICE SPEC TEMPLATE
  =====================
  Copy this file to `##-{service}.md` at the workspace root and fill it in.
  The numeric prefix sets reading order in the Spec Document Index.

  This is the fill-in skeleton. For the rules on WHAT each section must
  contain, read SPEC-GUIDELINES.md (§2 "Service Spec Files") — that is the
  canonical checklist; this template is just the scaffold to populate.

  How to generate one with an agent:
    1. Tell the agent which service repo it is, and point it at the repo.
    2. Have it read SPEC-GUIDELINES.md §2 and this template.
    3. Ask it to fill every section from the actual source — exact names,
       real endpoints, real schema fields. No speculation.
    4. Anything unresolved goes in "Known Issues & Gaps", not inline guesses.

  Rules of thumb (from SPEC-GUIDELINES.md):
    - Describe the IMPLEMENTED state, not the idea or a design draft.
    - Use exact identifiers (file names, DTO names, route paths).
    - Prefer tables and request/response examples over prose alone.
    - Delete this comment block and every <!-- guidance --> note before committing.
    - Remove sections that genuinely do not apply; do not pad them.
-->

# Plaza {Service Name} — Specification Document

> One-sentence description of what this service is and the single
> responsibility it owns.

**Service identifier:** `{service-id}`  <!-- e.g. main-api, tenant-service, cdn-service -->
**Last updated:** {YYYY-MM-DD} — {brief note on the most recent change}

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Architecture](#architecture)
4. [Authentication](#authentication)
5. [Data Model(s)](#data-models)
6. [Endpoints](#endpoints)
7. [Endpoint Summary](#endpoint-summary)
8. [Environment Variables](#environment-variables)
9. [Cross-Cutting Concerns / Patterns](#cross-cutting-concerns--patterns)
10. [Key Design Notes](#key-design-notes)
11. [Known Issues & Gaps](#known-issues--gaps)

<!-- Add/renumber entries to match the sections you actually keep. -->

---

## Overview

- **Purpose:** what the service does and why it exists.
- **Owns:** the data and responsibilities that live here and nowhere else.
- **Does not own:** responsibilities that look related but belong elsewhere
  (link to the spec that does own them).

---

## Tech Stack

| Concern        | Choice                                  |
| -------------- | --------------------------------------- |
| Language       | {e.g. TypeScript}                       |
| Framework      | {e.g. NestJS 10}                        |
| Datastore      | {e.g. MongoDB Atlas — database `xxx`}   |
| Default port   | {e.g. 3001}                             |
| Hosting        | {e.g. VPS + pm2 / cloud / local}        |

---

## Architecture

Internal module/component layout. Include a component tree or ASCII diagram
showing controllers → services → modules → schemas → datastore.

```text
{service-id}/
├── src/
│   ├── {feature}/
│   │   ├── {feature}.controller.ts
│   │   ├── {feature}.service.ts
│   │   ├── {feature}.module.ts
│   │   └── {feature}.schema.ts
│   └── main.ts
```

### Global Middleware / Bootstrap
<!-- Interceptors, filters, pipes, CORS, validation registered app-wide. -->

---

## Authentication

- **Strategy:** {e.g. JWT bearer, validated by delegating to Main API `/auth/validate`}.
- **Guard(s):** {e.g. `ApiAuthGuard`} — where it is applied.
- **Public vs protected:** which endpoint groups require auth and which do not.
- **Tenant claims:** how tenant isolation is enforced from the token.

---

## Data Model(s)

Field-by-field definition for each schema/domain entity this service owns.
Duplicate the table per entity.

### {EntityName} (collection `{collection_name}`)

| Field        | Type     | Required | Default | Notes / Index            |
| ------------ | -------- | -------- | ------- | ------------------------ |
| `_id`        | ObjectId | yes      | auto    | PK                       |
| `tenantId`   | string   | yes      | —       | indexed; tenant isolation|
| `{field}`    | {type}   | {y/n}    | {value} | {meaning, constraints}   |

---

## Endpoints

Group by purpose; separate **Public** from **Protected**. For each endpoint
document: method, path, guard, request shape/DTO, response shape, and any
validation rules or side effects.

### Public Endpoints

#### `GET /{path}` — {what it does}

- **Guard:** none
- **Request:** {params / query / body or DTO name}
- **Response:** 

  ```json
  { "example": "response" }
  ```

- **Notes:** {validation, side effects, error cases}

### Protected Endpoints (auth required)

#### `POST /{path}` — {what it does}

- **Guard:** `{GuardName}`
- **Request DTO:** `{CreateXDto}` — { key fields }
- **Response:** { shape or status }
- **Side effects:** { e.g. writes JSON to CDN, emits event }

---

## Endpoint Summary

Quick-reference table of every endpoint.

| Method | Path            | Auth | Purpose            |
| ------ | --------------- | ---- | ------------------ |
| GET    | `/{path}`       | —    | {summary}          |
| POST   | `/{path}`       | JWT  | {summary}          |

---

## Environment Variables

| Variable        | Required | Default | Controls                       |
| --------------- | -------- | ------- | ------------------------------ |
| `PORT`          | no       | {value} | listen port                    |
| `{VAR_NAME}`    | yes      | —       | {what it configures}           |

---

## Cross-Cutting Concerns / Patterns

- **Interceptors / filters:** {e.g. `GeneralInterceptor`, `HttpExceptionFilter`}.
- **Validation strategy:** {global pipes, DTO validation}.
- **Error/response envelope:** {standard success/error shape}.
- **Logging:** {what is logged, where}.

<!--
  FRONTEND SPECS ONLY — replace this section with the frontend additions
  from SPEC-GUIDELINES.md §6:
    - Base Request Layer (apiService.makeRequest semantics, tenantUrl resolution)
    - Redux Store Structure (key slice names)
    - Enumerations & Constants
    - Views & Pages
    - Frontend Patterns & Implementation Conventions (create/edit view pattern, etc.)
-->

---

## Key Design Notes

Important implementation decisions worth remembering: ID generation strategy,
active/inactive lifecycle rules, cross-service delegation, caching, etc.

---

## Known Issues & Gaps

- {Known bug, caveat, legacy behavior, or unresolved question.}
- {Anything a `[PENDING]` feature still depends on.}

<!-- Speculative / unresolved items live here, never inline as if implemented. -->
