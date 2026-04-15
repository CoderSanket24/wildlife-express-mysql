# 🌿 Wildlife Express

A full-stack web application for managing a wildlife sanctuary — tracking animals, staff, safari zones, visitor bookings, and medical records. Built as a **DBMS course project** using **Node.js + Express + MySQL** with server-side rendered EJS templates.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Design](#database-design)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [NPM Scripts](#npm-scripts)
- [API Routes](#api-routes)
- [Database Concepts Used](#database-concepts-used)
- [Team Contributions](#team-contributions)

---

## Overview

Wildlife Express is a sanctuary management system that allows:
- **Admins / Rangers** to manage animals, zones, and staff
- **Visitors** to register, book safari tickets, and submit feedback
- **Analysts** to view dashboards with revenue, booking trends, and visitor demographics

The project was built to demonstrate real-world application of core **Database Management System** concepts including stored procedures, triggers, views, joins, transactions, and indexes.

---

## Features

- 🐾 **Animal Management** — Add and track animal species, counts, habitat zones, and survey dates (with image uploads)
- 🗺️ **Zone Management** — Create and view safari zones with camera traps, climate, access levels, and species data
- 👥 **Visitor Management** — View visitor profiles, booking history, and demographics
- 🎟️ **Safari Booking** — Book safari tickets with optional add-ons (guide, camera, lunch, transport); GST auto-calculated via a database trigger
- ⭐ **Feedback System** — Collect and display visitor feedback and ratings
- 🏥 **Medical Records** — Log medical checkups, treatments, and feeding logs for animals
- 👮 **Ranger / Staff Management** — Add and manage sanctuary staff and rangers
- 📊 **Visitor Analytics Dashboard** — View real-time stats — age distribution, zone popularity, daily revenue, booking trends — powered by MySQL Views and indexed queries
- 🔐 **Authentication** — Session-based auth with JWT cookies, password hashing via Argon2, and flash messages

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js (ES Modules) |
| Framework | Express.js v5 |
| Database | MySQL 8 via `mysql2` |
| Templating | EJS |
| Auth | JWT + Argon2 + express-session |
| Validation | Zod |
| File Uploads | Multer |
| Styling | Vanilla CSS + EJS partials |

---

## Project Structure

```
wildlife-express-mysql/
│
├── app.js                   # Express app entry point
├── exampl.env               # Example environment configuration
├── package.json
│
├── config/                  # Environment & DB config
├── routes/
│   ├── wildlife.routes.js   # All sanctuary routes
│   └── auth.routes.js       # Login / register routes
│
├── controller/
│   ├── wildlife.controller.js  # Business logic for all features
│   └── auth.controller.js      # Auth logic
│
├── services/
│   ├── wildlife.service.js         # DB queries for core features
│   ├── visitor-analytics.service.js # Analytics queries (views-based)
│   └── auth.service.js             # Auth DB queries
│
├── middlewares/
│   ├── auth-verify-middleware.js   # JWT/session auth check
│   └── upload-middleware.js        # Multer image upload config
│
├── validators/              # Zod input validation schemas
├── views/                   # EJS templates
│   ├── index.ejs            # Home / dashboard
│   ├── animals.ejs
│   ├── zones.ejs
│   ├── visitors.ejs
│   ├── medical.ejs
│   ├── staff.ejs
│   ├── user-profile.ejs
│   ├── visitors-feedback.ejs
│   ├── auth/                # Login / register templates
│   └── forms/               # Add animal, zone, staff forms
│
├── public/                  # Static assets (CSS, JS, images)
│
└── database/
    ├── README.md                        # Database setup guide
    └── visitor_analytics_compatible.sql # Analytics SQL setup
```

---

## Database Design

### Core Tables

| Table | Description |
|-------|-------------|
| `visitors` | Registered visitors (name, email, age, etc.) |
| `tickets` | Safari bookings with add-ons and cost breakdown |
| `feedbacks` | Visitor ratings and comments |
| `animals` | Animal species with zone and survey info |
| `zones` | Safari zones with climate, area, and access level |
| `rangers_staff` | Rangers and staff with role and shift info |
| `medical_checkups` | Animal health records |
| `medical_treatments` | Treatment logs per animal |
| `feeding_logs` | Animal feeding schedule records |

### Analytics Views

| View | Purpose |
|------|---------|
| `v_age_distribution` | Visitor age group statistics |
| `v_zone_popularity` | Bookings and revenue per zone |
| `v_daily_revenue` | Daily revenue and visitor count |
| `v_booking_status` | Booking status distribution |
| `v_time_slot_preferences` | Popular safari time slots |
| `v_monthly_trends` | Month-over-month trends |
| `v_dashboard_stats` | Aggregated dashboard stats |
| `v_zones_enhanced` | Zone info joined with booking and animal stats |

---

## Getting Started

### Prerequisites

- Node.js v18+
- MySQL 8.0+
- npm

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/CoderSanket24/wildlife-express-mysql.git
cd wildlife-express-mysql

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp exampl.env .env
# Edit .env with your MySQL credentials and JWT secret

# 4. Set up the database (create tables, views, indexes)
npm run setup-db

# 5. Start the development server
npm run dev
```

Open your browser at **http://localhost:\<PORT\>**

---

## Environment Variables

Create a `.env` file in the root directory (refer to `exampl.env`):

```env
PORT=""                   # Server port (e.g. 3000)
DATABASE_HOST=""          # MySQL host (e.g. localhost)
DATABASE_USER=""          # MySQL username
DATABASE_PASSWORD=""      # MySQL password
DATABASE_NAME=""          # Database name
DATABASE_PORT=3306        # MySQL port (default: 3306)
SECREATE_KEY=""           # JWT secret key
```

---

## NPM Scripts

| Script | Command | Description |
|--------|---------|-------------|
| `dev` | `npm run dev` | Start dev server with hot-reload (`--watch`) |
| `start` | `npm start` | Start production server |
| `setup-db` | `npm run setup-db` | Run DB setup (views, indexes, stored procedures) |
| `update-analytics` | `npm run update-analytics` | Manually refresh analytics summary table |
| `cleanup-test-data` | `npm run cleanup-test-data` | Remove test/seed data from DB |

---

## API Routes

### Auth Routes

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/login` | Show login page |
| POST | `/login` | Authenticate visitor |
| GET | `/register` | Show register page |
| POST | `/register` | Register a new visitor |
| GET | `/logout` | Log out and clear session |

### Sanctuary Routes

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/` | Home / dashboard |
| GET | `/visitors` | All visitors list |
| GET | `/animals` | Animals list |
| GET/POST | `/addAnimal` | Add new animal (with image upload) |
| GET | `/zones` | Safari zones list |
| GET/POST | `/addZone` | Add a new zone |
| GET | `/staff` | Staff list |
| GET/POST | `/addStaff` | Add new ranger/staff |
| GET | `/medical` | Medical records |
| GET | `/addMedicalCheckup` | Log a medical checkup |
| GET | `/addTreatment` | Log a treatment |
| GET | `/addFeedingLog` | Log feeding data |
| GET/POST | `/booking` | Safari booking |
| GET/POST | `/feedback` | Submit feedback |
| GET | `/user-profile` | Logged-in visitor profile |
| GET | `/visitors-feedback` | View all visitor feedback |

### REST API

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/animals` | JSON list of all animals |
| GET | `/api/staff` | JSON list of all staff |
| GET | `/api/zones/details/:zoneId` | Zone details by ID |
| POST | `/api/medical-checkups` | Submit medical checkup |
| POST | `/api/medical-treatments` | Submit treatment record |
| POST | `/api/feeding-logs` | Submit feeding log |

---

## Database Concepts Used

This project was built to demonstrate core DBMS concepts:

### ✅ Stored Procedures
- `BookSafariTicket(...)` — Book a safari with duplicate booking check, transaction, and rollback on error *(Sanket)*
- `sp_HireRanger(...)` — Hire a ranger with FK validation and error handling *(Chaitanya)*
- `sp_AddZone_V2(...)` — Add a zone with transaction and error handling *(Pravin)*
- `sp_AddCheckup(...)` — Add a medical checkup with FK constraint handling *(Krishna)*
- `sp_LogAnimalSurvey(...)` — Upsert animal survey data with `ON DUPLICATE KEY UPDATE` *(Neha)*

### ✅ Triggers
- `before_ticket_insert_calculate_cost` — Auto-calculate `services_cost`, `gst_amount`, and `total_amount` before ticket insert *(Sanket)*
- `trg_Feedback_BeforeInsert` — Validate rating range and enforce comments for low ratings *(Sanket)*
- `trg_staff_before_insert` — Validate age, experience, and gender before inserting ranger *(Chaitanya)*
- `trg_staff_before_update` — Same validations on ranger updates *(Chaitanya)*

### ✅ Views
- `v_zones_enhanced` — Zone info enriched with booking stats and animal counts using LEFT JOINs *(Pravin)*
- Multiple analytics views (`v_age_distribution`, `v_daily_revenue`, etc.) for dashboard *(Sanket)*

### ✅ Joins
- INNER JOIN on `feedbacks` and `visitors` to show named feedback lists *(Sanket)*
- JOIN on `visitors` and `tickets` to retrieve booking history per visitor *(Sanket)*

### ✅ Indexes
Performance indexes on `tickets` and `visitors` for date, zone, status, and age-based queries.

### ✅ Transactions
All stored procedures use `START TRANSACTION` / `COMMIT` / `ROLLBACK` for data safety.

### ✅ Aggregate Functions
- `SUM(area)`, `SUM(camera_traps)` on zones *(Pravin)*
- `COUNT(*)`, `SUM(total_amount)` for revenue analytics

---

## Team Contributions

| Team Member | Contribution |
|-------------|-------------|
| **Sanket** | Booking system, ticket trigger (cost calc), feedback trigger, visitor analytics views, JOINs (tickets + feedback), dashboard |
| **Chaitanya** | Staff/ranger management, `sp_HireRanger` procedure, before-insert and before-update triggers for staff |
| **Pravin** | Zone management, `sp_AddZone_V2` procedure, `v_zones_enhanced` view, aggregate functions |
| **Krishna** | Medical checkup module, `sp_AddCheckup` procedure |
| **Neha** | Animal survey module, `sp_LogAnimalSurvey` procedure with upsert logic |

---

## License

This project is built for academic purposes as part of the **SY SEM-I DBMS course**.
