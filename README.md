---
# Algorithmic Multi-Account Derivatives Execution Platform
This Project completely done by me in the company Softwired, complete code sharing is not allowed

A professional-grade, full-stack trading solution designed for synchronized multi-account management across the Charles Schwab brokerage ecosystem. This platform enables "Agents" to manage a high-value portfolio (currently **$1M+ AUM**) by broadcasting complex equity and derivative strategies across **45+ accounts** simultaneously with sub-200ms execution latency.

## System Overview

The platform is architected to solve the "latency and synchronization" problem in manual copy-trading. By consolidating multiple brokerage accounts into a single interface, it allows for atomic execution of complex multi-leg options and equity trades.

* **Frontend:** React 18 (Vite), Tailwind CSS, React-Table.
* **Backend:** Node.js, Express, SQLite3.
* **Authentication:** JWT (Agent-level) & Puppeteer-driven OAuth2 (Brokerage-level).
* **Process Management:** PM2 (API + 3 specialized Cron Services).
* **Infrastructure:** Nginx Reverse Proxy on Ubuntu 22.04 with SSL (Certbot).

---

## Core Features

### 1. Multi-Account Copy Trading

* **Direct Broadcast:** Execute a single trade that is instantly mirrored across all "Active" accounts.
* **Account Management:** Real-time toggling of accounts to include/exclude them from specific trade sets.
* **Session Persistence:** Automated token refresh logic ensuring 24/7 API connectivity without manual login.

### 2. Advanced Derivatives Engine

* **Atomic Multi-Leg Orders:** Support for **Vertical Spreads** (2-legs) and **Iron Condors** (4-legs).
* **Strategy Exit Logic:** Intelligent "one-click" exit system that automatically calculates the reversing instructions (e.g., Buy-to-Close vs Sell-to-Close) for entire strategies.
* **Saved Order Sets:** Templates for complex strategies to enable rapid re-entry into high-probability trades.

### 3. Real-Time Synchronization

* **Tri-Cron Architecture:** Three dedicated background processes sync Stock orders, Option orders, and Account balances every 5–15 seconds.
* **Smart Polling:** Frontend optimization that detects "Terminal States" (Filled, Canceled, Rejected) to stop unnecessary API calls and reduce network overhead.

---

## Technical Architecture

### Backend Structure (`/backend/src`)

* **`controllers/`:** Business logic for order construction and account synchronization.
* **`data-access/`:** Abstracted SQL layer for SQLite, handling 5,000+ daily data points.
* **`models/`:** Schema definitions for 15+ relational tables.
* **`cron*.js`:** Decoupled background workers for high-availability data fetching.
* **`tradingAccountPuppeteer.js`:** Sophisticated browser automation for managing the Schwab OAuth flow and session caching with Memcached.

### Frontend Structure (`/frontend/src`)

* **`context/`:** Global state management for live orders, positions, and account balances.
* **`pages/optionContractVerticalTrading/`:** The high-complexity strategy builder and history tracker.
* **`shared/`:** Reusable UI components including a highly optimized data table.

---

## Installation & Local Setup

### Prerequisites

* Node.js v20+
* SQLite3
* Charles Schwab Developer API Credentials

### 1. Backend Setup

```bash
cd backend
npm install
# Configure your .env file with API keys and Super Admin credentials
npm start

```

### 2. Frontend Setup

```bash
cd frontend
npm install
npm run dev

```

---

## 🌐 Production Deployment

The platform is optimized for deployment on Ubuntu servers (via MobaXterm/SSH) using the following stack:

1. **Process Management:** * Main API: `pm2 start src/index.js --name traderswim-api`
* Cron Workers: `pm2 start src/cronAccountProcess.js`, etc.


2. **Web Server:** Nginx configured as a reverse proxy for the `/api` route and a static file server for the React `dist` build.
3. **Security:** SSL termination via Certbot (Let's Encrypt) and CORS restriction to `https://tradeswim.org`.

---

## Performance Metrics

* **Current AUM:** $1,000,000+ USD.
* **Managed Accounts:** 45+ Active Brokerage Accounts.
* **Execution Speed:** < 200ms from frontend "Place Order" to Backend API Broadcast.
* **Data Throughput:** 5,000+ synchronized daily data points.
* **Availability:** 99.9% System Uptime via PM2 auto-restart policies.

---
