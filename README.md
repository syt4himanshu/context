# context
Here's your complete project documentation:

---

# PDS Blockchain Project — Complete Documentation

## Project Overview

**Project Name:** Blockchain-Based Public Distribution System (PDS) with IoT Integration

**Developer:** Himanshu Mire

**Institution:** St. Vincent Pallotti College of Engineering and Technology, Nagpur (RTM University Affiliated)

**Degree:** Computer Engineering, VI Semester

**Academic Year:** 2024-25

**Purpose:** Final year college project (Project-I)

---

## What We Are Building

A complete digital infrastructure for India's Public Distribution System — the government program that distributes subsidized food grains (rice, wheat, sugar) to poor households through Fair Price Shops.

The system replaces manual paper-based ration distribution with a fully digital, blockchain-anchored, QR-code-driven platform that eliminates ghost beneficiaries, prevents duplicate claims, and creates a tamper-proof audit trail of every grain dispensing transaction.

---

## The Problem We Are Solving

India's PDS suffers from:
- Ghost and duplicate beneficiaries
- Manual register-based record keeping at shops (easily tampered)
- No real-time audit trail of dispensing transactions
- Unequal entitlement allocation not tied to actual family size
- No cross-verification between shops and beneficiaries
- Corruption at the Fair Price Shop level

---

## Three User Roles

```
ADMIN
  → Government officer
  → Manages everything from admin panel
  → Creates areas, shops, ration cards, assigns shopkeepers
  → Runs monthly entitlement allocation
  → Monitors blockchain readiness

SHOPKEEPER
  → Fair Price Shop operator
  → Assigned to exactly one shop
  → Scans beneficiary QR code
  → Views wallet balance
  → Dispenses ration and confirms transaction

BENEFICIARY
  → Ration card holder (family head or member)
  → Logs in via mobile OTP
  → Views their wallet balance
  → Generates QR code for shopkeeper to scan
  → Uses React Native mobile app
```

---

## Complete Tech Stack

### Backend
```
Runtime:        Node.js (v18+)
Framework:      Express.js
Database:       PostgreSQL (v15)
DB Driver:      node-pg (raw SQL, no ORM)
Auth:           JWT (jsonwebtoken) + bcrypt
OTP:            Twilio Verify API (replaced Firebase Phone Auth)
Scheduling:     node-cron (monthly entitlement reset)
Validation:     Joi
Security:       Helmet + express-rate-limit
Logging:        Winston
Migrations:     node-pg-migrate
QR Generation:  (handled frontend side)
Blockchain:     Solidity + Hardhat + Ethers.js (next phase)
```

### Frontend — Web (pds-frontend)
```
Framework:      React (Vite)
Styling:        Tailwind CSS
Routing:        React Router v6
HTTP Client:    Axios
Forms:          React Hook Form
QR Scanner:     @zxing/browser + @zxing/library
Auth Decode:    jwt-decode
```

### Frontend — Mobile (pds-beneficiary)
```
Framework:      React Native (Expo)
Purpose:        Beneficiary app only
Status:         Separate, untouched during web merging
```

### Blockchain (Next Phase)
```
Language:       Solidity
Dev Tool:       Hardhat
Integration:    Ethers.js (from Node backend)
Network:        Polygon Mumbai Testnet or Ganache (local)
```

### DevOps / Tools
```
IDE:            VS Code + Cursor AI
API Testing:    Postman
DB GUI:         pgAdmin
Version Control: Git + GitHub
Repo:           https://github.com/syt4himanshu/Major-Project
Process Mgr:    concurrently (root package.json)
```

---

## Project Folder Structure

```
PDS/                          ← root folder
├── pds-backend/              ← Node.js REST API (all roles share one backend)
├── pds-frontend/             ← Merged React web app (admin + shopkeeper)
├── pds-beneficiary/          ← React Native Expo app (beneficiary only)
├── package.json              ← root scripts (concurrently)
└── setup.md
```

### Root Commands
```bash
npm run dev          → starts backend + frontend together
npm run backend      → backend only
npm run frontend     → frontend only
npm run beneficiary  → React Native (separate terminal)
```

---

## Database Schema

**Database Name:** `ration_db`
**Connection:** `postgresql://himanshumire@localhost:5432/ration_db`

### Tables (10 total)

```
areas             → 6 areas (Dharampeth, Sadar, Manewada etc.)
policies          → 3 rows (APL/BPL/AAY entitlement rules)
users             → all roles live here (admin, shopkeeper, beneficiary)
shops             → 18 shops (3 per area), each has 1 shopkeeper
ration_cards      → one per family
family_members    → each member links back to users table
wallets           → one per ration card, holds rice/wheat/sugar balance
transactions      → every dispensing event, has blockchain_tx_hash column
qr_sessions       → short-lived QR tokens for beneficiary→shopkeeper flow
otp_verifications → Twilio OTP audit records
```

### Views (3 total)
```
v_beneficiaries    → full beneficiary overview with wallet
v_transactions     → transaction history with human-readable context
v_shop_summary     → shop stats with shopkeeper and transaction counts
```

### Pending Addition (for blockchain)
```sql
blockchain_logs    → tracks on-chain submission status
                     (pending / confirmed / failed)
                     needed to handle retry logic
```

### Seeded Data
```
3 policies     → APL (3kg rice/person), BPL (5kg), AAY (7kg fixed 35kg rice)
1 admin user   → admin@pds.gov / abcd1234
Areas/Shops    → Decision: NOT seeding anymore, create via admin panel
                 Only Nagpur-relevant areas (Dharampeth, Sadar, Manewada)
```

---

## Entitlement Rules (Policy Table)

| Category | Rice | Wheat | Sugar | Note |
|---|---|---|---|---|
| APL | 3kg/person | 2kg/person | 0.5kg/person | Above Poverty Line |
| BPL | 5kg/person | 3kg/person | 1kg/person | Below Poverty Line |
| AAY | 35kg FIXED | 8kg/person | 1kg/person | Poorest of poor — rice is flat 35kg regardless of family size |

---

## API Structure

### Auth Routes (public)
```
POST /auth/login          → email + password → JWT
POST /auth/otp/send       → mobile → Twilio sends OTP
POST /auth/otp/verify     → mobile + OTP → JWT
```

### Admin Routes (JWT + role: admin)
```
POST   /api/admin/ration-cards              → atomic creation (card + members + wallet)
GET    /api/admin/ration-cards              → paginated list
GET    /api/admin/beneficiaries             → filter by category/area/shop
GET    /api/admin/users                     → filter by role
GET    /api/admin/areas                     → with counts
GET    /api/admin/shops                     → with shopkeeper info
GET    /api/admin/shops?unassigned=true     → shops with no shopkeeper
POST   /api/admin/shopkeepers              → create shopkeeper + assign to shop
GET    /api/admin/entitlements/preview      → compute without saving
POST   /api/admin/entitlements/allocate     → reset all wallets (idempotent)
GET    /api/admin/validation/integrity      → 8 DB integrity checks
GET    /api/admin/validation/security       → 6 security checks
```

### Shopkeeper Routes (JWT + role: shopkeeper)
```
GET    /api/shopkeeper/me                          → shop info
GET    /api/shopkeeper/beneficiary/:ration_card_id → wallet + family info
POST   /api/shopkeeper/dispense                    → deduct wallet + save transaction
POST   /api/shopkeeper/transactions                → blockchain-ready stable endpoint
```

---

## Frontend Pages

### Admin Pages (/admin/*)
```
/admin/dashboard          → layout with sidebar
/admin/ration-cards       → table + Add button
/admin/ration-cards/new   → dynamic form (head + N members, area→shop cascade)
/admin/beneficiaries      → table with category/area/shop filters
/admin/users              → table with role toggle + Add Shopkeeper modal
/admin/areas              → summary table with counts
/admin/shops              → table with accordion detail
/admin/entitlements       → preview table + allocate button + cron status
/admin/validation         → integrity checks + security checks + readiness score
```

### Shopkeeper Pages (/shopkeeper/*)
```
/shopkeeper/dashboard     → shop info + Start Scanning button
/shopkeeper/scan          → 3 states: Scanning → Beneficiary Loaded → Confirm → Success
```

### Shared
```
/login                    → single login page, routes by role after JWT decode
/unauthorized             → shown when wrong role hits wrong area
```

---

## Authentication Flow

```
Admin/Shopkeeper:
  Email + Password → POST /auth/login → JWT → stored as 'pds_token'
  JWT decoded → role → redirect to correct dashboard

Beneficiary (mobile app):
  Mobile number → POST /auth/otp/send → Twilio SMS
  Enter OTP → POST /auth/otp/verify → JWT

Role enforcement:
  Every API route: verifyToken middleware + requireRole middleware
  Every frontend route: ProtectedRoute component with allowedRoles prop
```

---

## Key Business Logic

### Atomic Ration Card Creation
One API call does 7 things in a single DB transaction:
1. Validate card number uniqueness
2. Fetch policy for category
3. Create head user row
4. Get area from shop
5. Create ration card row
6. Create family member rows + user rows for each member
7. Create wallet with pre-computed balance

If anything fails → full rollback.

### Entitlement Engine
```
Runs: manually (admin button) OR automatically (cron, 1st of month, 6AM IST)
Idempotent: running twice same day → second run skipped automatically

Logic:
  AAY → rice = 35kg fixed, wheat/sugar = policy × family_size
  BPL → all = policy × family_size
  APL → all = policy × family_size
```

### Double Claim Prevention
```
Before every dispense:
  Check transactions table for current month
  If found → 400 "Already claimed this month"
  Wallet unchanged
```

### Cross-Shop Prevention
```
Shopkeeper can only serve beneficiaries assigned to their shop
If shop_id mismatch → 403 Forbidden
Logged as warning in Winston
```

---

## Security Layers

```
1. Helmet          → 11 HTTP security headers
2. Rate limiting   → /auth: 10 req/15min, /api: 100 req/min
3. Joi validation  → all input validated before hitting controller
4. bcrypt          → passwords hashed with cost factor 10
5. JWT             → 7 day expiry, role embedded in payload
6. Role middleware → verifyToken + requireRole on every route
7. Winston logging → all auth events, warnings, errors logged
8. OTP cleanup     → hourly cron expires stale OTPs
```

---

## Automated Jobs (node-cron)

```
Entitlement allocation:  '0 6 1 * *'   → 1st of every month, 6AM IST
OTP cleanup:             '0 * * * *'    → every hour
```

---

## Pre-Blockchain Validation System

Before any on-chain recording, the admin runs these checks:

### 8 Integrity Checks
```
1. Ration cards without wallet
2. Ration cards without family members
3. Orphaned family members
4. Shops without area
5. Negative wallet balances
6. Suspicious transactions (>50kg)
7. Duplicate card numbers
8. Unlinked beneficiary users
```

### 6 Security Checks
```
1. Helmet headers active
2. Rate limiting configured
3. OTP reuse prevention
4. Admin/shopkeeper without password hash
5. Duplicate beneficiary mobiles
6. Anonymous transactions (no served_by)
```

### Readiness Score
```
3 green dots required:
  ✅ All 8 integrity checks pass
  ✅ All 6 security checks pass (0 failures)
  ✅ All 10 manual checklist items ticked

→ "🚀 Ready for blockchain integration"
```

---

## Blockchain Plan (Next Phase)

### What Goes On-Chain vs Off-Chain
```
ON-CHAIN (immutable proof):        OFF-CHAIN (PostgreSQL):
─────────────────────────          ──────────────────────
card_number (hashed)               Beneficiary personal details
shop_id                            Family member names/ages
rice_qty_kg                        Mobile numbers
wheat_qty_kg                       Live wallet balance
timestamp                          Admin logs
tx_hash (self-reference)           Transaction history UI
```

### Flow
```
Shopkeeper dispenses
      ↓
Save to PostgreSQL (transactions table)
      ↓
POST to blockchain smart contract (async)
      ↓
Save tx_hash to blockchain_logs (status: pending)
Update transactions.blockchain_tx_hash
      ↓
Blockchain confirms
      ↓
Update blockchain_logs (status: confirmed)
```

### Missing Table to Add
```sql
CREATE TABLE blockchain_logs (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id   UUID NOT NULL REFERENCES transactions(id),
  tx_hash          TEXT,
  status           VARCHAR(20) DEFAULT 'pending',
  block_number     BIGINT,
  submitted_at     TIMESTAMP DEFAULT NOW(),
  confirmed_at     TIMESTAMP
);
```

---

## What's Completed vs Pending

### ✅ Completed
```
PostgreSQL schema (10 tables, 3 views)
Auth module (email/password + OTP)
Admin backend routes (all)
Shopkeeper backend routes (all)
Admin frontend (all 9 pages)
Shopkeeper frontend (scan + dispense flow)
Entitlement engine (manual + cron)
Security hardening (Helmet, rate limit, Joi)
Winston logging
Pre-blockchain validation dashboard
Merged pds-frontend (admin + shopkeeper in one app)
Root package.json (one command startup)
Synopsis document (college submission)
```

### 🔲 Pending
```
blockchain_logs table (add to schema)
Solidity smart contract
Hardhat setup
Ethers.js integration in backend
Blockchain submission on dispense
Transaction confirmation listener
pds-beneficiary React Native app (separate)
Seed data cleanup (remove Gujarat areas, use Nagpur only)
Production deployment
```

---

## Current Issues / Decisions Made

| Decision | What Was Chosen | Why |
|---|---|---|
| Firebase Phone Auth | Replaced with Twilio | Firebase requires Blaze paid plan for OTP |
| UI Library | Tailwind CSS only | Best with AI tools, no abstraction issues |
| ORM | None — raw SQL with pg | Full control, no hidden queries |
| Seed data | Policies + admin only | Areas/shops created via admin panel for realism |
| Frontend | Merged into one app | One command to run, cleaner structure |
| Blockchain storage | Hybrid (PG + chain) | Performance + auditability balance |

---

## Running The Project

```bash
# Start everything
cd PDS
npm run dev
# → Backend on port 5055
# → Frontend on port 5174

# Beneficiary app (separate terminal)
npm run beneficiary

# Database
psql -U himanshumire -d ration_db

# Admin login
Email:    admin@pds.gov
Password: abcd1234
```

---

This is the complete picture of everything built so far. Next step is blockchain integration — add the `blockchain_logs` table then write the Solidity contract. 🙂

Yes. Based on all our conversations over the past few weeks, I have enough context to produce a comprehensive project document. It will consolidate everything we've discussed rather than just summarize the latest chat.

The document will cover:

* Project Overview

  * Problem statement
  * Why this project exists
  * Objectives
  * Scope
  * Stakeholders (Admin, Shopkeeper, Beneficiary)

* Business Workflow

  * Current PDS process
  * Problems in the traditional system
  * Proposed digital workflow
  * End-to-end architecture

* System Architecture

  * Complete architecture diagram (described)
  * Three-module architecture

    * Admin Dashboard
    * Shopkeeper Dashboard
    * Beneficiary Mobile App
  * Backend architecture
  * Database architecture

* Technology Stack

  * Frontend

    * React
    * Vite
    * React Router
    * Axios
  * Mobile

    * React Native
    * Expo
  * Backend

    * Node.js
    * Express.js
    * JWT
    * Bcrypt
    * Node Cron
  * Database

    * PostgreSQL
  * QR System
  * OTP (Twilio with local fallback)
  * Blockchain (planned MVP)

* Database Design

  * Every table
  * Relationships
  * Why each table exists
  * ERD explanation

* Current Modules

  * Authentication
  * Admin
  * Shopkeeper
  * Beneficiary
  * Wallet
  * Policies
  * QR Session
  * Transactions
  * OTP

* Current Project Status

  * What is complete
  * What is partially complete
  * What remains

* Development Timeline

  * Initial architecture
  * Dashboard merge
  * Render deployment attempts
  * Neon migration
  * Recovery decision
  * Blockchain planning

* Major Technical Decisions

  * Why PostgreSQL
  * Why QR
  * Why Wallet model
  * Why RBAC
  * Why React Native
  * Why blockchain only for audit

* Problems Faced

  * Render
  * Neon
  * CORS
  * Environment variables
  * Twilio
  * Deployment artifacts
  * Broken local setup
  * Database recreation

* Recovery Strategy

  * Why we stopped deployment
  * Localhost-first approach
  * Database rebuild
  * Cleanup strategy

* Blockchain Roadmap

  * MVP architecture
  * Smart contract
  * Hardhat
  * Transaction recording
  * Verification endpoint
  * Dashboard

* Future Scope

  * Real Aadhaar integration
  * GPS verification
  * Face recognition
  * Analytics
  * Government APIs
  * Production deployment

* Presentation Notes

  * How to explain the architecture
  * How to explain blockchain
  * Expected viva questions
  * Strong technical points

* Development Roadmap

  * Phase 1: Restore localhost
  * Phase 2: Finish modules
  * Phase 3: Blockchain
  * Phase 4: Testing
  * Phase 5: Documentation
  * Phase 6: Optional deployment

* Lessons Learned

  * Architectural decisions
  * Engineering practices
  * Deployment experience
  * Database design improvements

This will not be a chat summary. It will read like a professional **Software Design & Development Document** for your major project, incorporating all of our discussions, decisions, architectural reasoning, implementation choices, issues encountered, and the current roadmap.

Given the amount of material we've covered, the document will be roughly **30–40 pages** in length when formatted, making it suitable as a project reference throughout the remainder of development.
