## 📣 Notification Service - Instyte CRM

### 📋 Overview
The Notification Service in Instyte CRM enables institutes to send real-time or scheduled messages to targeted users, such as students, classes, or batches. It supports multiple communication channels and integrates seamlessly with the branch-specific schema model.

### 🎯 Goals
- ✅ Enable targeted and custom notifications
- ✉️ Support multi-channel delivery: Email, SMS, Push, WhatsApp
- 🔄 Support future extensibility (e.g., templating, filters)
- 📈 Scalable to thousands of branches with isolated schema contexts

---

### 🔧 Core Features

#### 1. 🛠️ **Custom Notifications**
Admins or teachers can create custom notifications by:
- 👥 Selecting audience type:
    - 🎓 Specific students
    - 🏫 Specific classes
    - 📚 Batches/sections
    - 🌐 All students
- 🚀 Choosing message delivery medium:
    - 📧 Email, 📱 SMS, 🔔 Push Notification, 💬 WhatsApp
- 🗓️ Scheduling (optional):
    - Immediate send
    - Schedule for a future time/date

#### 📨 NotificationRequest DTO Example:
```json
{
  "title": "Exam Schedule Released",
  "message": "Check your dashboard for the new schedule.",
  "studentIds": ["uuid1", "uuid2"],
  "classCodes": ["X-A", "X-B"],
  "medium": "EMAIL",
  "scheduled": false,
  "customPlaceholders": {
    "studentName": "John"
  }
}
```

#### 2. 🎯 **Target Resolver**
The Notification Service includes logic to resolve actual recipient IDs from:
- 🆔 Student IDs
- 🔤 Class codes
- 🏷️ Section or batch tags

#### 3. 🚚 **Message Dispatcher**
- Dispatches the message to the proper channel service (email, sms, etc.)
- Works asynchronously using an internal job queue
- Uses tenant's current branch schema context

#### 4. 🧾 **Audit and Logging**
- All notifications are stored in a `notification_logs` table with:
    - 🕒 Sent timestamp
    - 🎯 Target group
    - ✅ Status (success/failure)
    - 📡 Channel used
    - 🔁 Retry attempts (if applicable)

---

### 🧱 Additional Design Details

#### 🏗️ Schema Context:
- Notification Service runs in **branch schema context** using `MetadataContext`
- Messages are created and stored per branch for isolation

#### 🤝 Integration with Admin Service:
- Only authenticated users with role `ADMIN` or `TEACHER` can send notifications
- User context and branch schema is resolved via JWT token

---

### 🚀 Future Enhancements
- 🧩 **Templated Notifications** with merge variables like `{{studentName}}`, `{{className}}`
- 🔍 **Filters** like fee defaulters, low attendance, inactive students
- 🔁 **Recurring Notifications** (daily/weekly reminders)
- 🌐 **Multilingual Support** per region or student profile
- 👁️‍🗨️ **Notification Read Receipts**

---

### 📌 Status
- ✅ Design finalized
- 🚧 Implementation in progress
- 🔜 Next step: Integrate channel service adapters and job queue

---

