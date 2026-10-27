# FairShift – A Decent Work Platform for Employee-Intensive Organisations
### UN Sustainable Development Goal 8 (SDG 8): Decent Work & Economic Growth

[![Test Suite](https://img.shields.io/badge/tests-16%20passed-brightgreen.svg)]()
[![Languages](https://img.shields.io/badge/languages-English%20%7C%20Tamil%20%7C%20Hindi-blue.svg)]()
[![Accessibility](https://img.shields.io/badge/accessibility-WCAG%20AA%20Compliant-success.svg)]()
[![Repository Size](https://img.shields.io/badge/repo%20size-%3C%201%20MB-blue.svg)]()

---

## 1. Executive Summary & Problem Statement

Large-workforce, frontline organisations (construction projects, colleges/universities, logistics yards, municipal sanitation) rely heavily on fragmented paper registers, informal WhatsApp threads, and opaque supervisor tallies:

- **Worker Vulnerability**: Workers rarely understand complex overtime calculations, lack itemized explanations, fear retaliation when reporting unsafe conditions, and face language barriers on basic smartphones.
- **Operational Opacity**: Employers and foremen lack real-time visibility into excessive shift overtime, pending safety risks, and workforce satisfaction.

**FairShift** is a lightweight, mobile-first, multi-industry platform delivering transparent, dignified, and safe work environments. Out of the box, FairShift adapts via configuration to **Construction Sites** and **Colleges**, providing low-literacy friendly tools for workers and actionable oversight for employers.

---

## 2. Feature-to-Pain-Point Mapping

| Frontline Pain Point | FairShift Feature Solution | Impact (SDG 8 Alignment) |
|---|---|---|
| **Opaque wage math & hidden deductions** | **Transparent Payslip with Itemized Math** (base pay, 1.5x/2.0x overtime multiplier, statutory EPF/taxes) | Ensures fair remuneration; builds mutual wage trust. |
| **Reading & language barriers** | **Plain-Language "Why This Amount" Explanation + Audio Read-Aloud (TTS)** in English & Tamil | Inclusivity for low-literacy and regional language speakers. |
| **Unrecorded hours & buddy punching** | **Site Code & Geofenced Check-In/Out** with automated standard vs overtime hours calculation | Accurate, tamper-resistant timesheets without expensive hardware. |
| **Fear of reporting hazards & accidents** | **Multi-Channel Safety Reporting** (Text or Voice dictation via Web Speech API) + **4-Stage Status Tracker** | Transparent hazard lifecycle (`Submitted` → `Acknowledged` → `In Progress` → `Resolved`). |
| **Retaliation for honest feedback** | **100% Anonymous Feedback** (zero worker ID stored, one-way HMAC rate limit, k-anonymity min group threshold) | Safe voice for workers; genuine morale telemetry for leadership. |
| **Employer blind spots to overtime** | **Employer Operations Dashboard** with real-time shift trends, open hazards, and aggregated sentiment | Actionable workforce welfare oversight. |

---

## 3. System Architecture

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

## 6. Local Setup & Testing

### Prerequisites
- Node.js 20+

### Quickstart
```bash
# 1. Install dependencies
npm install

# 2. Run test suite (Node.js built-in test runner, zero extra test dependencies)
npm test

# 3. Start development server
npm start
```

Visit **`http://localhost:8080`** in your browser.

---

## 7. SDG 8 Impact Notes

- **Target 8.5 (Full & Productive Employment with Equal Pay)**: Transparent, itemized wage breakdowns clarify pay computations in regional languages (English, Tamil, Hindi), eliminating distrust.
- **Target 8.8 (Protect Labour Rights & Safe Working Environments)**: Lowering the reporting barrier through voice dictation and 4-stage tracking ensures safety hazards are recorded and remedied before injuries occur.
- **Privacy & Psychological Safety**: One-way salted hash rate limiting and k-anonymity (k ≥ 3) allow frontline workers to communicate without fear of supervisor reprisal.

---

## 8. Limitations & Future Work

- **Offline-First SQLite / Local Storage Sync**: While the web app is ultra-lightweight (< 25 KB JS uncompressed), expanding local IndexedDB caching will allow shift logging in zero-connectivity tunnels.
- **Biometric QR Verification**: Hardware-assisted thermal/facial check-in kiosk integration for high-throughput gates.
- **WhatsApp Notifications**: Automated WhatsApp message dispatching for payslip PDF receipts.

---

## 9. Self-Audit Checklist

- [x] **Repo Size**: Strictly under 10 MB (`< 1 MB` total excluding `node_modules`).
- [x] **Git Branch**: Single branch only (`main`).
- [x] **Zero Secrets**: All credentials dynamically provisioned; `.env.example` provided.
- [x] **Separated Login Portals**: Independent Construction Site (`METRO-4`) and College Campus (`APEX-1`) portals.
- [x] **Per-Person Tasks Folders**: Individual task folders (`tasks/<person-slug>/tasks.json`) for every worker and supervisor.
- [x] **Attendance & Overtime**: Check-in via site code and geolocation; automated 8-hour cap & overtime calculation.
- [x] **Transparent Payslip**: Itemized breakdown with plain-language explanation and Web Speech TTS read-aloud in English, Tamil & Hindi.
- [x] **Safety Grievance**: Voice/text submission with photo support and 4-stage status tracking.
- [x] **Anonymous Feedback**: Zero worker ID stored; one-way HMAC rate-limit token; k-anonymity threshold (k ≥ 3).
- [x] **Accessibility**: WCAG AA contrast (≥ 4.5:1), 44px minimum touch targets, visible focus states, skip link, prefers-reduced-motion.
- [x] **Passing Tests**: `npm test` runs 16 test suites with 100% pass rate.

---

## 10. LinkedIn Post Draft

> 🏗️ **Introducing FairShift: Reimagining Decent Work for Frontline Organisations (UN SDG 8)**
>
> Millions of workers in construction yards, colleges, and factories still navigate opaque registers, delayed overtime payments, and unaddressed safety hazards. Most speak regional languages and use basic smartphones, leaving them with little visibility into how their pay was calculated or where their complaints went.
>
> That's why we built **FairShift**:
> 
> 🔹 **Site Code & Geofenced Attendance**: Instant clock-in with automated standard vs. overtime hours calculation.  
> 🔹 **Transparent Payslips + Voice Read-Aloud**: Itemized earnings breakdown with a plain-language "Why this amount" explanation in English, Tamil & Hindi with one-tap text-to-speech.  
> 🔹 **Voice-First Safety Reporting**: Dictate safety hazards directly (Web Speech API) and track status in real-time through an accountable 4-stage lifecycle (`Submitted` → `Acknowledged` → `In Progress` → `Resolved`).  
> 🔹 **Cryptographic Anonymous Feedback**: Zero worker IDs stored; one-way rate limiting and k-anonymity guarantee workers can speak up safely without fear of reprisal.  
> 🔹 **Separated Sector Portals & Per-Person Task Folders**: Dedicated Construction Site and College Campus login portals with personal task folders for every worker.
>
> #Hackathon #SDG8 #DecentWork #NodeJS #WebAccessibility #CivicTech
