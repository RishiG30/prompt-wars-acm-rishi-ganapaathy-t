# FairShift – A Decent Work Platform for Employee-Intensive Organisations


---

## 1. Executive Summary & Problem Statement

Large-workforce, frontline organisations (construction projects, colleges/universities, logistics yards, municipal sanitation) rely heavily on fragmented paper registers, informal WhatsApp threads, and opaque supervisor tallies:

- **Worker Vulnerability**: Workers rarely understand complex overtime calculations, lack itemized explanations, fear retaliation when reporting unsafe conditions, and face language barriers on basic smartphones.
- **Operational Opacity**: Employers and foremen lack real-time visibility into excessive shift overtime, pending safety risks, and workforce satisfaction.

**FairShift** is a lightweight, mobile-first, multi-industry platform delivering transparent, dignified, and safe work environments. FairShift adapts via configuration to **Construction Sites** and **Colleges**, providing low-literacy friendly tools for workers and actionable oversight for employers.

---

## 2. Feature-to-Pain-Point Mapping

| Frontline Pain Point | FairShift Feature Solution | Impact (SDG 8 Alignment) |
|---|---|---|
| **Opaque wage math & hidden deductions** | **Transparent Payslip with Itemized Math** (base pay, 1.5x/2.0x overtime multiplier, statutory EPF/taxes) | Ensures fair remuneration; builds mutual wage trust. |
| **Reading & language barriers** | **Plain-Language "Why This Amount" Explanation + Audio Read-Aloud (TTS)** in 100% native **English, Tamil (`தமிழ்`), and Hindi (`हिन्दी`)** | Inclusivity for low-literacy and regional language speakers. |
| **Unrecorded hours & buddy punching** | **Site Code & Geofenced Check-In/Out + Live Stopwatch Timer** with automated standard vs overtime hours calculation | Accurate, tamper-resistant timesheets without expensive hardware. |
| **Fear of reporting hazards & accidents** | **Multi-Channel Safety Reporting** (Voice Speech-to-Text dictation, Photo Proof upload & viewer, and **4-Stage Status Tracker** with strict worker anonymity) | Transparent hazard lifecycle (`Submitted` → `Acknowledged` → `In Progress` → `Resolved`). |
| **Retaliation for honest feedback** | **100% Anonymous Feedback & Complaints** (zero worker ID/name stored or shown, one-way HMAC rate limit) | Safe voice for workers; genuine morale telemetry for leadership. |
| **Role-based task accountability** | **Per-Person Tasks Folders (`tasks/<person-slug>/tasks.json`)** with strict RBAC (Admins assign/delete; Workers update completion status) | Clear shift responsibilities without micromanagement. |
| **Employer blind spots to overtime** | **Employer Operations Dashboard** with real-time shift trends, open hazards, and aggregated sentiment | Actionable workforce welfare oversight. |

---

## 3. Tech Stack

- **Frontend**:
  - **HTML5, CSS3 (Mobile-First Responsive UI, WCAG AA Accessible), Vanilla JavaScript (ES6+)** (`public/index.html`, `public/css/style.css`, `public/js/app.js`)
  - **Web Speech API**: Real-time Speech-to-Text (`SpeechRecognition` / `webkitSpeechRecognition`) for voice hazard reporting and Text-to-Speech (`speechSynthesis`) for reading payslips aloud in English (`en-IN`), Tamil (`ta-IN`), and Hindi (`hi-IN`)
  - **HTML5 Geolocation & Canvas APIs**: On-site GPS geofence verification and client-side hazard photo compression & preview
  - **Trilingual i18n Engine**: Complete localization in `public/locales/en.json`, `public/locales/ta.json`, and `public/locales/hi.json`
- **Backend**:
  - **Runtime & Server**: **Node.js (v18+)** + **Express.js** (`src/index.js`, `src/app.js`)
  - **Validation**: **Zod** schema validation (`src/middleware/validation.js`)
  - **Authentication & Cryptography**: Native Node **`crypto`** (`scrypt` password hashing, `HMAC-SHA256` one-way anonymous feedback tokens) + **JWT (`jsonwebtoken`)** role-based bearer authentication (`src/utils/crypto.js`, `src/middleware/auth.js`)
  - **Security & Performance Middleware**: **`helmet`** (CSP & HTTP security headers), **`express-rate-limit`** (brute-force & DDoS protection), **`compression`** (Gzip), **`cors`**
- **Data & Persistence**:
  - **Zero-Setup In-Memory + File-Synced Repository** (`src/repos/inMemoryRepo.js`, `src/data/seedData.json`)
  - **Per-Person File System Tasks Storage**: `tasks/<person-slug>/tasks.json`
  - **Multi-Industry JSON Schema Engine**: `src/config/industries/construction.json` (`METRO-4`) and `src/config/industries/college.json` (`APEX-1`)
- **Testing**:
  - **Node.js Built-in Test Runner (`node --test`)** + **`supertest`** (`tests/*.test.js` — 16 automated unit & integration tests)

---

## 4. System Architecture

```mermaid
flowchart TD
    subgraph Client["Frontend Client Layer (Mobile-First PWA)"]
        WApp["👷 Worker Interface<br/>• Site Code & Geofence Check-in<br/>• Itemized Payslip + Web Speech TTS<br/>• Voice & Text Hazard Reporting<br/>• Anonymous Feedback Form"]
        AApp["👨‍💼 Admin Dashboard<br/>• Shift & Overtime Trends<br/>• Grievance Status & Reply Manager<br/>• K-Anonymity Sentiment Telemetry"]
    end

    subgraph Server["Backend API (Node.js 20 + Express)"]
        MW["Security & Compliance Layer<br/>Helmet, Strict CORS, Rate Limiting, Zod Validation"]
        
        subgraph DomainServices["Core Domain Services"]
            ATT["Attendance & Overtime Service<br/>(Site Code, Geofence, 8h cap + OT)"]
            PAY["Transparent Payslip Service<br/>(Gross, OT multiplier, Deductions, Net, EN/TA TTS)"]
            GRV["Grievance & Safety Service<br/>(4-Stage Lifecycle, Photo, Voice Transcription)"]
            FDB["Anonymous Feedback Engine<br/>(Zero worker ID, One-Way Salted Hash, k ≥ 3)"]
        end
        
        CONF["Industry Config Engine<br/>(Construction: METRO-4, College: APEX-1)"]
    end

    subgraph Storage["Storage Abstraction Layer"]
        REPO["In-Memory Repository & Per-Person Tasks Folders<br/>(Zero Setup, tasks/<person>/tasks.json)"]
    end

    Client --> MW
    MW --> DomainServices
    DomainServices --> CONF
    DomainServices --> REPO
```

---

## 4. Multi-Industry Configuration (Zero Code Modification)

FairShift does **not** hardcode roles, wage multipliers, hazards, or workplace terminology. Industry configurations are driven entirely by JSON schemas in `src/config/industries/`:

### 1. Construction Site (`src/config/industries/construction.json`)
- **Site Code**: `METRO-4`
- **Workplace**: Metro Rail Station Sector 4 Construction Yard
- **Roles & Wages**: Mason (₹650/day), Electrician (₹800/day), Welder (₹750/day), Scaffolder (₹700/day)
- **Overtime Multiplier**: **2.0x** (Double-time for hazardous physical construction overtime)
- **Hazards**: High-elevation scaffolding falls, electrical exposure, crane blindspots, heat stress

### 2. College & University (`src/config/industries/college.json`)
- **Site Code**: `APEX-1`
- **Workplace**: Academic Quad & Central Laboratories
- **Roles & Wages**: Adjunct Lecturer (₹150/hr), Lab Demonstrator (₹90/hr), Library Assistant (₹75/hr)
- **Overtime Multiplier**: **1.5x** (Extra duty exam invigilation / lab grading)
- **Hazards**: Chemical reagent spills, lab electrical short circuits, honorarium delays

### How to Onboard a Third Industry (e.g. Factory or Hospital) via Config Only:
Simply add `src/config/industries/factory.json`:
```json
{
  "id": "factory",
  "name": "Sriperumbudur Auto Component Assembly",
  "siteCode": "AUTO-FAC-2",
  "roles": [{ "id": "cnc_op", "title": "CNC Operator", "baseHourlyRate": 95 }],
  "wageRules": { "standardDayHours": 8, "overtimeMultiplier": 2.0 },
  "hazardCategories": [{ "id": "press_guard", "label": "Press Machine Sensor Failure" }],
  "geofence": { "name": "Assembly Unit 2", "latitude": 12.98, "longitude": 79.96, "radiusMeters": 400 }
}
```
FairShift dynamically loads the new schema with **zero code modifications**.

---

## 5. Demo Personas & Unique Credentials

You can view all emails and unique passwords via the **"🎭 Demo Personas"** menu in the top header and sign in through the separated **Worker Login Portal** or **Employer (Admin) Login Portal**:

| Persona | Portal Type | Sector | Email | Unique Password | Tasks Folder |
|---|---|---|---|---|---|
| **Murugan Selvam** | Worker (Mason) | Construction (`METRO-4`) | `murugan@metrobuild.org` | `murugan123` | `tasks/murugan-selvam/` |
| **Ramesh Sharma** | Worker (Electrician) | Construction (`METRO-4`) | `ramesh@metrobuild.org` | `ramesh123` | `tasks/ramesh-sharma/` |
| **David Fernandez** | Employer / Admin (Supervisor) | Construction (`METRO-4`) | `admin@metrobuild.org` | `david123` | `tasks/david-fernandez/` |
| **Dr. Kavitha Raman** | Worker / Staff (Lecturer) | College (`APEX-1`) | `kavitha@apexcollege.edu` | `kavitha123` | `tasks/kavitha-raman/` |
| **Arjun Nair** | Worker / Staff (Lab Demonstrator) | College (`APEX-1`) | `arjun@apexcollege.edu` | `arjun123` | `tasks/arjun-nair/` |
| **Prof. S. Ranganathan**| Employer / Admin (Dean) | College (`APEX-1`) | `dean@apexcollege.edu` | `dean123` | `tasks/s-ranganathan/` |

---

## 7. Local Setup Steps & How to Run It

### Prerequisites
- **Node.js v18+** (or v20+) and **npm**
- **No external database or third-party API keys required**

### Step-by-Step Setup & Execution
```bash
# 1. Clone the repository and navigate into the project folder
git clone https://github.com/RishiG30/prompt-wars-acm-rishi-ganapaathy-t.git
cd prompt-wars-acm-rishi-ganapaathy-t

# 2. (Optional) Copy environment template — safe defaults are built-in
cp .env.example .env

# 3. Install dependencies
npm install

# 4. Run the automated test suite (16 unit & security tests)
npm test

# 5. Start the FairShift server
npm start
```

### Accessing the Application
- Open **`https://prompt-wars-acm-rishi-ganapaathy-t.vercel.app`** in your browser.
- **Step 1**: Choose your sector (**Construction Site `METRO-4`** or **College Campus `APEX-1`**).
- **Step 2**: Log in via the separated **Worker / Staff Login Portal** or **Employer (Admin) Login Portal** (click **🎭 Demo Personas** in the top navigation bar to view reference credentials).
- **Language Switching**: Use the top-right language selector to switch live between **🌐 English**, **🌐 தமிழ் (Tamil)**, and **🌐 हिन्दी (Hindi)**.

---
