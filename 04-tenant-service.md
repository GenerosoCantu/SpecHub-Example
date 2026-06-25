# Plaza Tenant Service — Specification Document

> Resolves a store from an incoming domain and owns the `stores` collection.
> Lightweight and precise; delegates auth validation to `main-api`.

**Service identifier:** `tenant-service`
**Last updated:** 2026-06-24 — no change for gift cards (not affected).

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Architecture](#architecture)
4. [Authentication](#authentication)
5. [Data Model](#data-model)
6. [Endpoints](#endpoints)
7. [Environment Variables](#environment-variables)
8. [Known Issues & Gaps](#known-issues--gaps)

---

## Overview

- **Purpose:** map a domain/subdomain to a `storeId`, and manage store records.
- **Owns:** the `stores` collection (the tenant record).
- **Does not own:** products, orders, or assets.

## Tech Stack

| Concern   | Choice                          |
| --------- | ------------------------------- |
| Framework | NestJS 10                       |
| Port      | 4010                            |
| Datastore | MongoDB Atlas — database `plaza`|

## Architecture

- `StoresModule` (lookup + CRUD), `AppModule`, `ApiAuthGuard`.
- Global `HttpExceptionFilter` and `ValidationPipe`.

## Authentication

- **Public:** the domain lookup endpoint is unauthenticated.
- **Protected:** store CRUD requires a valid token; this service calls
  `main-api` `POST /auth/validate` and checks the returned `role`.

## Data Model

### Store (collection `stores`)

| Field      | Type     | Required | Default | Notes / Index            |
| ---------- | -------- | -------- | ------- | ------------------------ |
| `_id`      | ObjectId | yes      | auto    | PK = `storeId`           |
| `name`     | string   | yes      | —       |                          |
| `domains`  | string[] | yes      | —       | indexed; unique per entry|
| `active`   | boolean  | yes      | `true`  | inactive → storefront 404 |

## Endpoints

### Public

#### `GET /stores/resolve?domain=` — resolve a store by domain

- **Guard:** none
- **Response:** `{ storeId, name, active }` or `404` if no match / inactive.

### Protected (token required)

| Method | Path           | Purpose            |
| ------ | -------------- | ------------------ |
| POST   | `/stores`      | create a store     |
| PATCH  | `/stores/:id`  | update a store     |

## Environment Variables

| Variable           | Required | Default | Controls                |
| ------------------ | -------- | ------- | ----------------------- |
| `PORT`             | no       | 4010    | listen port             |
| `TENANT_MONGO_URI` | yes      | —       | Atlas connection string |
| `MAIN_API_URL`     | yes      | —       | for `/auth/validate`    |

## Known Issues & Gaps

- Domain changes are not historically tracked.
