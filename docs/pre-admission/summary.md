# 📘 Pre-Admission Module – High Level Design

## 🧠 Overview

The Pre-Admission module is responsible for handling leads, tracking follow-ups, and converting interested individuals into enrolled students. It is the first step in the student lifecycle for any educational institution using the Instyte CRM platform.

---

## 🎯 Objectives

- Capture inquiries (leads) from potential students
- Assign counselors to handle leads
- Schedule and track follow-ups
- Convert leads into students or mark as dropped

---

## 🧩 Key Components

| Component          | Responsibility                                      |
|-------------------|------------------------------------------------------|
| **UI (Frontend)** | Lead submission form and counselor dashboard         |
| **PreAdmissionService** | Business logic for leads and follow-ups         |
| **UserService**    | Counselor data management                           |
| **PostgreSQL DB**  | Data storage (per-branch schema)                    |

---

## 🔄 Process Flow

### 📝 Creating a Lead
1. User submits a form via the website/app
2. `PreAdmissionService` receives the request
3. Lead is validated and saved to the database
4. Assigned counselor is notified

### 🔁 Follow-Up Handling
1. Counselor schedules follow-up
2. System sends reminder notifications (if enabled)
3. Counselor updates status after call/meeting

### ✅ Conversion
1. Lead is marked as "Converted"
2. Data is passed to the Student Management module

---

## 📊 Database Entities (Simplified)

### `leads`
| Column      | Type       | Description              |
|-------------|------------|--------------------------|
| id          | UUID       | Primary Key              |
| name        | TEXT       | Name of the lead         |
| email       | TEXT       | Email ID                 |
| phone       | TEXT       | Contact number           |
| status      | ENUM       | [New, InProgress, Converted, Dropped] |
| created_at  | TIMESTAMP  | Creation time            |

### `follow_ups`
| Column       | Type      | Description              |
|--------------|-----------|--------------------------|
| id           | UUID      | Primary Key              |
| lead_id      | UUID      | Foreign Key (leads.id)   |
| notes        | TEXT      | Notes from counselor     |
| next_contact | TIMESTAMP | Scheduled next contact   |

---

## 🖼️ Diagrams

- **[Component Diagram](./component-diagram.png)** – Shows how services connect
- **[Sequence Diagram](./sequence-diagram.png)** – Illustrates the "Create Lead" flow

---

## 💡 Future Enhancements

- Lead Scoring using AI
- Automated follow-up reminders
- Integration with WhatsApp for instant responses

---

> 🧠 This module is the entry point of the Instyte platform and sets the foundation for the complete student lifecycle.