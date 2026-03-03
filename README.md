# 🩸 HemoConnect

### Smart Blood Donation & Request Management Platform

## 🌍 The Story Behind BloodBridge

Every 2 seconds, someone in the world needs blood.

In emergencies, families run from hospital to hospital searching for blood units.
Meanwhile, willing donors don’t know **where** or **when** they are needed.

💡 **BloodBridge was built to connect the gap** — between patients in urgent need, verified donors, and hospitals managing inventory.

This is not just a web app.
It’s a **real-time, role-based blood ecosystem platform**.

---

## 🚀 What Problem Does It Solve?

✔ Patients struggle to find available blood units quickly
✔ Hospitals manually manage donor verification
✔ No centralized real-time request tracking
✔ Donors don’t get structured scheduling
✔ Blood inventory visibility is limited

🔴 Emergency situations demand speed, trust, and coordination.
🟢 BloodBridge delivers exactly that.

---

# ✨ Core Modules & Features

---

## 🔴 1. Blood Request System

### 🧭 How It Works

1. User navigates to **Find Blood**
2. Selects hospital using map or search
3. Fills request form:

   * Patient Name
   * Blood Group
   * Units Required
   * Upload Prescription (Required)
4. Submit request
5. Track status in real-time dashboard

### 🔄 Request Lifecycle

| Status      | Meaning                  |
| ----------- | ------------------------ |
| 🟡 Pending  | Awaiting hospital review |
| 🟢 Approved | Request accepted         |
| 🔴 Rejected | Rejected with reason     |

🧠 Smart Feature:
Dashboard automatically polls every 3 seconds for status updates.

---

## 🧑‍⚕️ 2. Donor Management & Health Screening

### Step-by-Step Flow

1️⃣ Basic Information
2️⃣ Medical History & Health Questions
3️⃣ Lab Test Values (Hemoglobin, BP)
4️⃣ ID & Employment Verification Upload

Hospital reviews → Approves/Rejects

### Donor Status Flow

`Pending → Screening → Approved → Scheduled`

---

## 🗓️ 3. Smart Donation Scheduling

* Choose date (0–90 days ahead)
* 16 time slots (8:00 AM – 5:30 PM)
* Reschedule capability
* Appointment visible in hospital panel

---

## 🏥 4. Hospital Admin Panel

Hospital Staff Can:

✔ Review blood requests
✔ Preview uploaded prescriptions
✔ Approve / Reject with reason
✔ View approved donors
✔ Track scheduled appointments
✔ Manage blood inventory

---

## 🗺️ 5. Hospital Locator (Map Based)

* Built using **Leaflet.js**
* Search by name or blood type
* Live blood inventory display
* Interactive map markers

---

## 📚 6. Education Hub

Educates users about:

* Blood types
* Donation eligibility
* Myths vs Facts
* Importance of regular donation

---

# 👥 User Roles

| Role              | Capabilities                               |
| ----------------- | ------------------------------------------ |
| 🧑 Donor          | Apply, upload documents, schedule donation |
| 🏥 Hospital Staff | Approve/reject requests, manage donors     |
| 🧑‍💼 Requestor   | Request blood, track status                |
| 👀 General User   | Browse hospitals & education               |

---

# 🏗️ Tech Stack

## 💻 Frontend

* React (TypeScript)
* Vite
* Tailwind CSS
* Radix UI
* Material UI
* Leaflet.js
* date-fns

## 🖥️ Backend

* Node.js
* Express.js
* MongoDB
* JWT Authentication
* Multer (File Upload)
* Role-based Middleware

---

# 🔐 Security Architecture

✔ JWT-based Authentication
✔ Password hashing (bcrypt)
✔ Role-based access control
✔ Secure file upload validation
✔ Data isolation per user

---

# 🔄 Real-Time Architecture

Instead of complex WebSockets:

🔁 3-second polling system
⚡ Lightweight & scalable for small-mid deployments
📈 Easily upgradeable to WebSockets in future

---

# 🗄️ Database Design (MongoDB)

### Collections

* Users
* Blood Requests
* Donor Applications
* Hospitals
* Inventory

Structured references:

* `userId`
* `hospitalId`
* Real-time status tracking

---

# 🛠️ Installation Guide

## 📌 Prerequisites

* Node.js (v16+)
* MongoDB (Local/Atlas)
* npm

---

## 🔧 Backend Setup

```bash
cd backend
npm install

# Create .env file
MONGODB_URI=mongodb://localhost:27017/blood-donation
NODE_ENV=development

node server.js
```

---

## 🎨 Frontend Setup

```bash
npm install
npm run dev
```

---

# 📊 Project Highlights

* ✔ Multi-role architecture
* ✔ Real-time request tracking
* ✔ Medical document handling
* ✔ Appointment scheduling logic
* ✔ Role-based dashboards
* ✔ Map-based hospital finder
* ✔ Modular scalable backend
* ✔ Proper RESTful APIs

---

# 🧠 System Scalability

Current:

* Polling based updates
* Local file storage

Future Upgrade Path:

* WebSocket integration
* Redis caching
* Cloud file storage (AWS S3)
* Email + SMS integration
* Push notifications
* AI-based donor matching

---

# 🌟 Why This Project Stands Out

This is not just a CRUD application.

It demonstrates:

* Full MERN architecture
* Complex user-role workflows
* Real-time UI logic
* File upload handling
* Secure authentication
* Production-ready design
* Real-world healthcare problem solving

---

# 🎯 Ideal Use Case

* Smart City Healthcare Systems
* NGO Blood Donation Platforms
* Hospital Digital Infrastructure
* Emergency Response Systems

---

# ❤️ Vision

> "No patient should lose their life due to blood unavailability."

BloodBridge aims to create a **digitally connected blood ecosystem** where supply meets demand instantly.

---

# 📌 Future Roadmap

* 📲 SMS Alerts
* 📧 Email Confirmations
* 📊 Donation Analytics
* 🤖 AI Blood Matching Engine
* 🧾 Donation History Records
* ⭐ Hospital Feedback System

---

# 🏁 Final Note

* Built with the intention to save lives through technology.
* Designed using clean architecture, scalable backend, and real-world workflow logic.
