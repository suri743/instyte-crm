# 🏢 TenantService – High-Level Design

## 🎯 Purpose

TenantService is the internal, owner-only service that lets you (the SaaS provider) manage:

- ✅ Account (Tenant) onboarding
- ✅ Initial admin user creation (in account schema)
- ✅ Central account registry in platform schema
- ✅ Provisioning account-level PostgreSQL schema
- ✅ Central billing and feature flags ingestion
- ✅ Trial management (e.g., 1-week free period)
- ✅ Invoice generation for each billing cycle
- ❌ Not visible to customers

---

## 🧩 Core Responsibilities

| Feature                  | Description                                  |
|--------------------------|----------------------------------------------|
| 🏢 Account Management     | Create/delete account (e.g., DPS, Oakridge)   |
| 🧱 Schema Provisioning    | Create dedicated schema per account          |
| 👤 Admin User Setup       | Add admin user into account schema           |
| 🔄 Notify Platform Service | Push account metadata, billing, features     |
| 📈 Reporting API          | (Optional) Get summary of accounts + usage   |
| ⏳ Trial Period Tracking   | Manage trial mode and expiry dates           |
| 🧾 Invoicing              | Generate invoices from billing usage data     |

---

## 🗃️ Data Model (platform schema)

### `accounts`

| Field                 | Type      | Description                           |
|-----------------------|-----------|---------------------------------------|
| id                    | UUID      | Unique account ID                     |
| name                  | TEXT      | Full name                             |
| code                  | TEXT      | e.g., `dps`                           |
| schema_name           | TEXT      | PostgreSQL schema name                |
| admin_email           | TEXT      | Main admin contact                    |
| status                | TEXT      | `ACTIVE`, `DISABLED`, `TRIAL_EXPIRED` |
| start_date            | DATE      | Date account was created or activated |
| trial_end_date        | DATE      | Last day of trial period              |
| is_trial              | BOOLEAN   | Indicates if account is in trial mode |
| subscription_end_date | DATE      | Paid plan end date (if applicable)    |
| plan_type             | TEXT      | e.g., `TIER_1`, `CUSTOM`, `TIER_3`    |
| base_cost             | NUMERIC   | Annual or monthly base plan cost      |
| features_cost         | NUMERIC   | Sum of optional feature costs         |
| total_cost            | NUMERIC   | Computed = base_cost + features_cost  |
| currency              | TEXT      | e.g., `INR`, `USD`                    |
| created_at            | TIMESTAMP | Onboarding timestamp                  |
| business_type         | TEXT      | 'SCHOOL', "INSTITUTE', 'COLLEGE'       |


### `billing_usage`

| Field            | Type      | Description                                |
|------------------|-----------|--------------------------------------------|
| id               | UUID      | Unique usage entry ID                      |
| account_id       | UUID      | References `accounts.id`                   |
| service_name     | TEXT      | e.g., `notification`, `ai_engine`          |
| status           | TEXT      | `ACTIVE`, `INACTIVE`                       |
| usage_value      | NUMERIC   | Units used in the period                   |
| start_date       | DATE      | Start of billing cycle                     |
| end_date         | DATE      | End of billing cycle                       |
| amount_billed    | NUMERIC   | Total billed amount for the period         |
| currency         | TEXT      | Billing currency (e.g., `INR`)             |
| payment_status   | TEXT      | `PENDING`, `PAID`, `FAILED`                |
| invoice_id       | TEXT      | Optional reference to external invoice     |
| last_updated     | TIMESTAMP | Time of last sync or update                |


### `account_features`

| Field        | Type      | Description                                |
|--------------|-----------|--------------------------------------------|
| account_id   | UUID      | References `accounts.id`                   |
| feature_code | TEXT      | e.g., `NOTIFICATION`, `AI_ENGINE`          |
| enabled      | BOOLEAN   | true/false                                 |
| updated_at   | TIMESTAMP | Last updated                               |


### `feature_catalog`

| Field         | Type      | Description                                  |
|---------------|-----------|----------------------------------------------|
| id            | UUID      | Unique feature ID                            |
| code          | TEXT      | Feature key (e.g., `NOTIFICATION`)           |
| name          | TEXT      | Friendly name                                |
| description   | TEXT      | Short description of the feature             |
| default       | BOOLEAN   | Enabled by default for new accounts?         |
| group         | TEXT      | Optional logical group (e.g., `CORE`, `ADDON`)|
| pricing_hint  | TEXT      | Optional UI pricing label (e.g., `₹2500/year`)|
| order         | INTEGER   | Display order in admin UI                    |


### `invoices`

| Field               | Type      | Description                                  |
|----------------------|-----------|----------------------------------------------|
| id                   | UUID      | Unique invoice ID                            |
| invoice_number       | TEXT      | e.g., `INV-2025-04-0001`                     |
| account_id           | UUID      | References `accounts.id`                     |
| billing_period_start | DATE      | Start of billing cycle                       |
| billing_period_end   | DATE      | End of billing cycle                         |
| generated_on         | TIMESTAMP | When the invoice was created                 |
| due_date             | DATE      | Payment due date                             |
| total_amount         | NUMERIC   | Total invoice amount                         |
| currency             | TEXT      | e.g., `INR`                                  |
| payment_status       | TEXT      | `PENDING`, `PAID`, `FAILED`, `OVERDUE`       |
| payment_method       | TEXT      | e.g., `Stripe`, `Manual`, `Bank Transfer`    |
| notes                | TEXT      | Optional remarks                             |

---

## 🔌 APIs (Private/Internal Only)

### 🔹 Account Management
- `POST /tenant/accounts` → Creates account schema + metadata + admin user
- `GET /tenant/accounts` → List all accounts
- `DELETE /tenant/accounts/{code}` → (Optional) Remove account + schema

### 🔹 Feature Flag Setup
- `PATCH /tenant/accounts/{id}/features` → Enable/disable features per account
- `GET /tenant/features` → Fetch list of available features from `feature_catalog`

### 🔹 Billing/Usage Sync
- `POST /tenant/accounts/{id}/usage` → Accept usage metric for billing

### 🔹 Trial & Subscription APIs
- `PATCH /tenant/accounts/{id}/trial-end` → Manually end trial or extend
- `PATCH /tenant/accounts/{id}/subscription` → Update paid plan start/end dates
- `GET /tenant/accounts/{id}/status` → Returns `TRIAL`, `ACTIVE`, or `EXPIRED`

### 🔹 Invoices
- `GET /accounts/{id}/invoices` → List past invoices
- `POST /accounts/{id}/invoices/generate` → Generate invoice for current period
- `GET /invoices/{id}` → Get full invoice details
- `PATCH /invoices/{id}/status` → Update payment status

---