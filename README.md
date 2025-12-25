---

# Algorithmic Multi-Account Derivatives Execution Platform

> **Notice:** This project was architected and developed entirely by me at **Softwired**. Due to company policy and proprietary intellectual property, complete code sharing is strictly prohibited. This documentation serves as a high-level technical overview of the system architecture and implementation.

---

## Executive Summary

The **Algorithmic Multi-Account Derivatives Execution Platform** is a high-performance, full-stack trading solution built to manage a high-value portfolio of **$1,000,000+ Assets Under Management (AUM)**. The platform solves the critical challenge of trade synchronization across decentralized brokerage accounts. By integrating directly with the **Charles Schwab API**, it allows a single operator to broadcast complex equity and multi-leg option strategies across **45+ concurrent accounts** with sub-200ms execution latency.

---

## Technical Architecture

The system utilizes a modern decoupled architecture designed for high availability and strict data integrity.

### 1. High-Level Stack

* **Frontend:** React 18 (Vite), Tailwind CSS, React Context API, React-Table.
* **Backend:** Node.js (LTS), Express.js.
* **Database:** SQLite3 (optimized for low-latency local persistence).
* **Caching:** Memcached (session and token caching).
* **Automation:** Puppeteer/Playwright (headless browser integration for OAuth flow).
* **Process Management:** PM2 (Advanced process clustering and monitoring).
* **Infrastructure:** Nginx Reverse Proxy on Ubuntu 22.04 LTS.

---

## Core Functional Modules

### A. Advanced Derivatives Engine

Unlike standard retail platforms, this system treats multi-leg strategies as **atomic units**.

* **Strategy Support:** Native support for **Vertical Spreads** (2-legs) and **Iron Condors** (4-legs).
* **Complex Order Construction:** Dynamically builds `JSON` payloads for the Schwab API, ensuring that multi-leg trades are filled as a single strategy rather than individual legs to prevent "legging-in" risk.
* **The "Reverse Instruction" Logic:** A custom-engineered algorithm that parses existing multi-leg positions and automatically generates opposing instructions (e.g., flipping `BUY_TO_OPEN` to `SELL_TO_CLOSE`) for instant, one-click strategy exits.

### B. The "Fan-Out" Broadcast System

* **Concurrent Execution:** Uses `Promise.all` logic in the backend to broadcast orders to all active accounts simultaneously, ensuring near-identical fill prices across the entire $1M+ portfolio.
* **Account Filtering:** An intelligent toggling system allows the Agent to select specific "Active" accounts for each trade, providing granular control over capital allocation.

### C. Persistent Session Management

* **Automated OAuth Flow:** Integrated Puppeteer-based automation to handle the Schwab login handshake and initial authorization.
* **Token Refresh Service:** A background worker monitors JWT and OAuth token expiration, silently refreshing credentials in the background to ensure 24/7 "ready-to-trade" status.

---

## Backend Logic & Specialized Services

The backend is divided into four distinct service layers to ensure modularity and fault tolerance:

### 1. The API Layer (Express)

Handles all frontend requests, user authentication, and manual trade triggers. All endpoints are protected via JWT-based middleware.

### 2. The Tri-Cron Heartbeat

The system operates three specialized background processes (`Node-Cron`) to synchronize data without blocking the main API thread:

* **Stock Sync Process (5s):** Polls for equity order fills and position updates.
* **Option Sync Process (5s):** Monitors complex multi-leg option strategies.
* **Account Monitor (15s):** Updates account balances, buying power, and total AUM.

### 3. Smart Polling Optimization

To adhere to brokerage API rate limits, I implemented a **Status Terminality Check**. The system identifies orders in "Terminal States" (Filled, Canceled, Rejected, Expired) and automatically excludes them from the polling queue, reducing network overhead by over **60%** during high-volume trading sessions.

---

## Performance & Scale

* **Assets Under Management:** $1,000,000+ USD.
* **Account Scalability:** Currently managing 45+ unique brokerage sessions.
* **Broadcast Latency:** < 200ms from frontend "Submit" to total API broadcast.
* **Daily Data Throughput:** Processing **5,000+ synchronized data points** daily through background cron workers.
* **Uptime:** 99.9% achieved via PM2 auto-restart and Nginx health monitoring.

---

## Deployment & DevOps

The project follows a rigorous production deployment workflow:

1. **Server Environment:** Hardened Ubuntu 22.04 LTS instance.
2. **Web Server (Nginx):** * Configured as a **Reverse Proxy** to forward `/api` requests to the Node.js cluster.
* Serves the React `dist` build as optimized static assets.
* Enforces **SSL/TLS** encryption via Certbot/Let's Encrypt.


3. **Process Management (PM2):**
* `traderswim-api`: The main Express server.
* `traderswim-cron-account`: Dedicated balance/BP monitoring.
* `traderswim-cron-option`: Dedicated options synchronization.
* `traderswim-cron-stock`: Dedicated equity synchronization.


4. **Security:** * Strict **CORS** policies restricted to production domains.
* Encrypted storage for refresh tokens.
* Environment variable isolation for all API secrets.



---

## Frontend Design Philosophy

* **Context-Driven State:** Utilizes multiple React Contexts (`AccountContext`, `StockContext`, `CopyTradingContext`) to prevent "prop-drilling" and ensure real-time UI updates across different tabs.
* **Modular Strategy Builders:** Custom-built entry forms for complex spreads that provide real-time validation of strikes, expirations, and net credit/debit calculations.
* **Strategy History:** A sophisticated "History" view that provides a "Master Status" for strategies, allowing users to drill down into individual account-level fills.

---

## Key Achievements

* Successfully built a system that manages **$1M+ in live capital** without a single synchronization failure.
* Reduced the time required to exit a 4-leg Iron Condor across 15 accounts from **10 minutes** (manual) to **under 5 seconds** (automated).
* Engineered a custom parsing engine for **OCC Option Symbols** to convert raw strings (e.g., `SPXW 251205C07200000`) into human-readable formats for better trader UX.

---
