# Plaza Asset Service — Specification Document

> Stores uploaded images and the published JSON snapshots that the storefront
> reads. The only writer to object storage; delegates auth to `main-api`.

**Service identifier:** `asset-service`
**Last updated:** 2026-06-24 — no change for gift cards (balances are not snapshotted).

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Storage Model](#storage-model)
4. [Authentication](#authentication)
5. [Endpoints](#endpoints)
6. [Environment Variables](#environment-variables)
7. [Known Issues & Gaps](#known-issues--gaps)

---

## Overview

- **Purpose:** accept image uploads and write the per-store JSON snapshot.
- **Owns:** the object-storage layout. Owns no domain data.

## Tech Stack

| Concern   | Choice     |
| --------- | ---------- |
| Framework | NestJS 10  |
| Port      | 4020       |
| Storage   | Object storage fronted by a CDN |

## Storage Model

```text
stores/{storeId}/
├── snapshot.json          ← published catalog/config (read by storefront)
└── assets/
    └── {imageId}.{ext}    ← uploaded images
```

## Authentication

- All write endpoints require a token; validated via `main-api`
  `POST /auth/validate`. Reads are served as static files by the CDN, not this
  service.

## Endpoints

| Method | Path                          | Auth | Purpose                       |
| ------ | ----------------------------- | ---- | ----------------------------- |
| POST   | `/files/:storeId/upload`      | yes  | upload an image               |
| PATCH  | `/files/:storeId/snapshot`    | yes  | write the store JSON snapshot |
| DELETE | `/files/:storeId/:imageId`    | yes  | delete an image               |
| GET    | `/health`                     | —    | health check                  |

## Environment Variables

| Variable          | Required | Default | Controls                |
| ----------------- | -------- | ------- | ----------------------- |
| `PORT`            | no       | 4020    | listen port             |
| `ASSET_BUCKET`    | yes      | —       | object-storage bucket   |
| `MAIN_API_URL`    | yes      | —       | for `/auth/validate`    |

## Known Issues & Gaps

- No cleanup job for orphaned images after product delete.
