# 🛡️ AdminService – High-Level Design (with RBAC)

## 🧠 Purpose

AdminService is the central platform module responsible for:

- ✅ User authentication (Login + JWT token)
- ✅ Branch onboarding (creates PostgreSQL schemas)
- ✅ User management (admin, counselor, staff)
- ✅ Role-Based Access Control (RBAC) to secure routes and actions

---

## 🧩 Core Responsibilities

| Feature               | Description                                              |
|------------------------|----------------------------------------------------------|
| 🔑 Login               | Authenticates users and returns JWT                      |
| 🏢 Branch onboarding   | Creates a new branch and schema                          |
| 👤 User creation       | Adds new users with roles under branches                 |
| 🔐 Role Enforcement    | Controls access to APIs based on user roles              |
| 📦 JWT Token Generation| Encodes user ID, branch ID, and role in every request    |

---

## 📦 Entities (Stored in `public` schema)

### `users`

| Field       | Type     | Description                |
|-------------|----------|----------------------------|
| id          | UUID     | Primary key                |
| email       | TEXT     | Login ID                   |
| password    | TEXT     | Hashed password            |
| role        | TEXT     | `ADMIN`, `COUNSELOR`, etc. |
| branch_id   | TEXT     | Linked schema/tenant       |

---

### `branches`

| Field        | Type     | Description                      |
|--------------|----------|----------------------------------|
| id           | TEXT     | Unique key (`muzigal_hyd`)       |
| name         | TEXT     | Display name (e.g., Hyderabad)   |
| schema_name  | TEXT     | Schema name (usually same as ID) |

---

## 🔌 API Endpoints

### 1. `POST /admin/login`
> Authenticates and returns a JWT

```json
Request:
{
  "email": "admin@muzigal.com",
  "password": "muzigal@123"
}

Response:
{
  "token": "JWT-TOKEN",
  "branch": "muzigal_hyd",
  "role": "ADMIN"
}