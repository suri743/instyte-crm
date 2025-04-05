# 🏢 PlatformService – High-Level Design

## 🎯 Purpose

PlatformService is the internal, owner-only service that lets you (the SaaS provider) manage:

- ✅ Tenant (Customer) onboarding
- ✅ Initial admin user creation
- ✅ Central tenant/user registry
- ✅ Authentication (JWT-based login)
- ✅ Central billing data ingestion
- ❌ Not visible to customers

---

## 🧩 Core Responsibilities

| Feature                  | Description                                  |
|--------------------------|----------------------------------------------|
| 🏢 Tenant Management      | Create/delete tenant (e.g., DPS, Oakridge)    |
| 👤 Admin User Setup       | Add tenant-level admin users                 |
| 🔐 Login + JWT            | Authenticate and issue tenant-aware tokens   |
| 📊 Billing Data Sync      | Pull usage info from tenant services         |
| 📈 Reporting API          | (Optional) Get summary of tenants + services |

---

## 🗃️ Data Model (public schema only)

### `tenants`

| Field       | Type     | Description                     |
|-------------|----------|---------------------------------|
| id          | TEXT     | Unique tenant ID (e.g., `dps`)   |
| name        | TEXT     | Full name                       |
| schema_name | TEXT     | PostgreSQL schema (usually = id)|
| created_at  | TIMESTAMP | Onboarding date                 |

---

### `users`

| Field       | Type     | Description                          |
|-------------|----------|--------------------------------------|
| id          | UUID     | User ID                              |
| email       | TEXT     | Login email                          |
| password    | TEXT     | (hashed in prod)                     |
| role        | TEXT     | `TENANT_ADMIN` or `PLATFORM_ADMIN`   |
| tenant_id   | TEXT     | Foreign key to `tenants.id`          |
| status      | TEXT     | `ACTIVE` / `DISABLED`                |

---

### `billing_usage`

| Field        | Type     | Description                                |
|--------------|----------|--------------------------------------------|
| tenant_id    | TEXT     | e.g., `dps`                                 |
| branch_id    | TEXT     | e.g., `dps_hyd`                             |
| service_name | TEXT     | e.g., `notification`, `feedback`           |
| status       | TEXT     | `ACTIVE`, `INACTIVE`                       |
| last_updated | TIMESTAMP| Time of last sync                          |

---

## 🔌 APIs (Private to YOU)

### 🔹 Tenant Management

- `POST /platform/tenants` → Creates schema + registers tenant
- `GET /platform/tenants` → List all tenants
- `DELETE /platform/tenants/{id}` → Optional removal

---

### 🔹 User Management

- `POST /platform/tenants/{id}/users` → Creates admin user
- `GET /platform/users` → List all users (with tenant info)

---

### 🔐 Auth

- `POST /platform/login` → Login as admin, returns JWT
  ```json
  {
    "token": "jwt-token",
    "tenant": "dps",
    "role": "TENANT_ADMIN"
  }