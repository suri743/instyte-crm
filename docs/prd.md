# 📘 Instyte CRM – Product Requirements Document (PRD) – Version 2

## 🧭 Version Info
**Version**: 2.0
**Date**: April 2025
**Author**: Surish

---

## 🎯 Product Vision
Instyte CRM is a scalable, modular, and multi-tenant SaaS CRM platform designed for educational institutions such as schools, colleges, and franchise-based organizations. The product enables full lifecycle management of students — from lead generation to post-admission services, attendance, feedback, and reporting.

The goal is to provide institutions with:
- Complete visibility across branches
- Custom service selection
- Transparent billing
- Scalable infrastructure
- Performance-optimized architecture

---

## 🧱 Architecture Overview
- **Multi-Tenant SaaS** using **PostgreSQL** schema-per-tenant model
- **Schema-per-branch** model to isolate data at the institution’s sub-unit level
- **Role-Based Access Control (RBAC)** enforced using **JWT tokens**
- **Spring Boot Modular Monolith** with clear domain-driven design
- **React-based UI** for Admins and Platform Owner
- **API-First Design** using RESTful standards
- **Cloud-Ready Deployment** (Docker, Kubernetes)
- **High Performance with Horizontal Scaling Support**

### 🌐 High-Level Design (HLD) Enhancements

#### 🔀 Load Balancing (LB)
- Global Load Balancer for all tenants at ingress
- Dedicated Load Balancer per Tenant (Org) to isolate traffic, throttle, and rate-limit as needed

#### 🌐 API Gateway
- Central API Gateway for all platform services
- Tenant-level API gateway routing for scoped microservices (e.g., admin, leads, notifications)
- Future support for GraphQL aggregation layer

#### 🗃️ Schema Design
- Separate PostgreSQL **schema per tenant** (e.g., `dps`, `oakridge`)
- Within each tenant, a **separate schema per branch** (e.g., `dps_hyd`, `dps_blr`) for ultimate isolation

#### ⚙️ Performance-Oriented Enhancements
- Connection pooling per schema using HikariCP
- Async request handling for non-blocking flows (Spring WebFlux-ready)
- Caching layer (e.g., Redis) for repeated read queries
- Horizontal scaling of services behind gateway
- CDN support for serving static assets (React UI)
- Rate limiting per tenant at API Gateway level
- Background jobs using Quartz/Scheduler for syncs, reports, expiry reminders

---

## 🌍 Subdomain Routing (Multi-Tenant UX Enhancement)

Instyte CRM supports subdomain-level routing to offer a white-labeled, isolated experience for each tenant.

| Tenant Name       | Subdomain URL                     |
|-------------------|------------------------------------|
| Delhi Public School | https://dps.instyte.com          |
| Oakridge School     | https://oakridge.instyte.com     |

### How it works:
- Subdomains are resolved at the gateway (NGINX, API Gateway, or frontend router)
- Subdomain is extracted from the `Host` header and mapped to a tenant ID (e.g., `dps` → `dps` schema)
- JWT still includes the tenant ID for backend validation

**Benefits**:
- Professional, branded experience
- Easier scaling for white-labeling
- Improved user trust

---

## 🧩 Core Modules in Detail

### 🏢 PlatformService (Internal Only)
Handles:
- Onboarding of new tenants (schools/orgs)
- Creation of `schema_name`
- Admin user creation
- JWT issuance
- Billing sync from all tenants

**APIs**:
- `POST /platform/tenants`
- `POST /platform/tenants/{id}/users`
- `POST /platform/login`
- `POST /platform/sync-usage`
- `GET /platform/billing-summary`

**Tables**:
- `public.tenants`
- `public.users`
- `public.billing_usage`

---

### 🛠 AdminService (Per-Tenant Admin Panel)
Allows tenant admins to:
- Create/manage branches
- Add users with roles per branch
- Subscribe to optional services
- View pricing info

**APIs**:
- `POST /branches`
- `GET /branches`
- `POST /branches/{id}/subscribe`
- `POST /admin/login`
- `GET /pricing`
- `GET /internal/billing-summary` (for PlatformService)

**Tables (inside tenant schema)**:
- `branches`
- `users`
- `branch_subscriptions`

---

### 👨‍🎓 Pre-Admission Service
Manages:
- Lead creation
- Follow-up scheduling
- AI-based scoring (based on source, interest)

**APIs**:
- `POST /leads`
- `POST /leads/{id}/followup`
- `GET /leads/{id}`
- `GET /leads/branch/{branchId}`

**Tables**:
- `leads`
- `followups`

---

### 👥 User Service
Manages all users within a tenant:
- User profiles, emails, passwords (hashed)
- Role assignment
- Audit trail of creation and updates

**APIs**:
- `POST /users`
- `GET /users`
- `PUT /users/{id}`
- `DELETE /users/{id}`

**Tables**:
- `users`
- `user_logs`

---

### 📅 Attendance Service
Captures daily attendance across branches:
- By student, date, subject
- Attendance summary/reporting

**APIs**:
- `POST /attendance`
- `GET /attendance/{studentId}`
- `GET /attendance/summary`

**Tables**:
- `attendance`

---

## 🧩 Optional Services (Per Branch)
These are modular services that tenants can subscribe to on a **per-branch basis**. PlatformService tracks all subscription and billing logic centrally.

### 🟡 Notification Service
- WhatsApp, SMS, email alerts
- Automated reminders for follow-ups, leads, and attendance
- Supports per-template customization

**APIs**:
- `POST /notifications/send`
- `POST /notifications/template`
- `GET /notifications/history`

### 🟡 Feedback Service
- Student/parent feedback forms
- Star ratings, NPS, open comments
- Analytics dashboard per branch

**APIs**:
- `POST /feedback/form`
- `POST /feedback/submit`
- `GET /feedback/summary`

### 🟡 Reporting Service
- Visual dashboards (attendance, lead conversion, user activity)
- Export reports to PDF, Excel

**APIs**:
- `GET /reports/attendance`
- `GET /reports/leads`
- `GET /reports/usage`

### 🟡 Payment Service
- Manage student fee collection
- Track dues, payments, receipts
- Optional Razorpay/Stripe integration

**APIs**:
- `POST /payments/collect`
- `GET /payments/history`
- `POST /payments/reminder`

**Pricing (Add-ons)**:
| Service       | Description                              | Price (INR)     |
|---------------|------------------------------------------|------------------|
| Notification  | WhatsApp/email alerts/reminders         | ₹2,500/month     |
| Feedback      | Collect feedback + analytics             | ₹2,000/month     |
| Reporting     | Dashboard + report downloads             | ₹1,500/month     |
| Payment       | Fee tracking + optional gateway setup    | ₹2,500/month     |

---

## 🔐 Role Matrix
| Role             | Access Scope                    | Can Create     | Who Assigns      |
|------------------|----------------------------------|----------------|------------------|
| PLATFORM_ADMIN   | All tenants, all data           | Tenants        | System Owner     |
| TENANT_ADMIN     | Their schema & branches         | Branches/users | Platform Admin   |
| COUNSELOR        | Assigned branch only            | Leads, followups | Tenant Admin  |
| STAFF            | Limited branch data             | View Only      | Tenant Admin     |

---

## 🔄 JWT Token Format
```json
{
  "sub": "admin@dps.com",
  "tenant": "dps",
  "role": "TENANT_ADMIN",
  "iat": 1712330000,
  "exp": 1712416400
}
```
Used by all services for:
- Dynamic schema routing
- Role validation

---

## 💰 Pricing Model

### Base Package (per branch)
| Tier          | Student Count     | Base Price (₹) | Full Bundle (₹) |
|---------------|-------------------|----------------|------------------|
| Tier 1 – Small | 100–500           | ₹15,000/year   | ₹25,000/year     |
| Tier 2 – Med   | 500–5,000         | ₹20,000/year   | ₹32,500/year     |
| Tier 3 – Large | 5,000–10,000+     | ₹24,000/year   | ₹34,500/year     |

### Optional Add-ons (Flat Rate – Per Branch)
- Notification: ₹2,500/month
- Feedback: ₹2,000/month
- Reporting: ₹1,500/month
- Payment: ₹2,500/month

---

## 📊 Reporting Dashboard (Platform Owner)
Accessible only via PlatformService
- Total tenants onboarded
- Branches per tenant
- Active services per branch
- Estimated monthly revenue
- Most used services
- Billing-ready export (CSV)

---

## 🧪 Tech Stack
| Layer              | Tech                           |
|--------------------|----------------------------------|
| Backend            | Spring Boot (Java)              |
| Frontend           | React + Tailwind UI             |
| Database           | PostgreSQL (schema-per-tenant, schema-per-branch)  |
| Auth               | JWT (HS256)                     |
| Caching            | Redis (planned)                 |
| Messaging (future) | Kafka or RabbitMQ               |
| Deployment         | Docker + Kubernetes             |
| Monitoring         | Prometheus + Grafana            |
| Gateway            | NGINX / Spring Cloud Gateway    |
| Load Balancer      | Cloud LB per tenant             |

---

## 🛠 Sprint Execution – 4-Week MVP

### ✅ Week 1 – PlatformService
- Tenant onboarding
- Admin creation
- Login & token generation
- Schema creation logic
- Billing usage table

### ✅ Week 2 – AdminService
- Branch & user APIs
- Optional service subscriptions
- Role enforcement via JWT

### ✅ Week 3 – Core Modules
- Pre-Admission full flow
- User profile management
- Attendance tracker

### ✅ Week 4 – Reporting + Dashboard
- Billing summary aggregation
- Internal dashboard APIs
- Export to CSV/PDF
- UI scaffolding (React dashboard)

---

## 🛣 Future Roadmap
- IAM service as separate module
- Payment Gateway integration
- Custom white-label branding
- Audit logging and history
- Email/SMS config per tenant
- Public self-onboarding portal
- Subdomain routing (e.g., dps.instyte.com)
- Global + Tenant-Level Load Balancers
- Tenant-scoped API Gateway with circuit breakers

---

✅ This Version 2 PRD reflects the **complete modular SaaS product model**, with detailed pricing, core + optional services (including Payment Service), platform owner visibility, subdomain routing, dedicated LB/API gateway per tenant, schema-per-branch support, and performance optimizations for enterprise readiness.

