---

# Algorithmic Multi-Account Derivatives Execution Platform

> **PROPRIETARY NOTICE:** This project was architected, engineered, and deployed entirely by me during my tenure at **Softwired**. Complete source code sharing is strictly prohibited by company policy. This documentation serves as a comprehensive technical breakdown of the system's architecture, logic, and operational scale for professional review.

---

## Executive Summary

The **Algorithmic Multi-Account Derivatives Execution Platform** is a high-performance, full-stack trading ecosystem designed to manage a high-value portfolio of **$1,000,000+ Assets Under Management (AUM)**.

In a traditional trading environment, managing 45+ decentralized brokerage accounts requires a massive overhead of manual entry, leading to "execution drift" where fill prices vary wildly between accounts. This platform solves that problem by providing a centralized **"Command and Control"** interface. It allows a single operator to broadcast complex equity and multi-leg derivative strategies (Iron Condors, Vertical Spreads) across **45+ concurrent Charles Schwab accounts** with sub-200ms execution latency, ensuring near-perfect synchronization across the entire $1M+ portfolio.

---

## Technical Architecture & Stack

The system is built on a decoupled, service-oriented architecture designed for high availability, low-latency data synchronization, and strict financial data integrity.

### 1. The Frontend (Client Layer)

* **React 18 (Vite):** Utilized for a lightning-fast development cycle and optimized production builds.
* **Tailwind CSS:** For a high-density, "Bloomberg-style" professional trading interface.
* **React Context API:** Implemented a modular state management system (`AccountContext`, `StockContext`, `CopyTradingContext`) to ensure real-time data updates across the dashboard without unnecessary re-renders.
* **React-Table (TanStack):** Powering the high-performance data grids capable of sorting and filtering hundreds of live orders and positions.

### 2. The Backend (Server Layer)

* **Node.js & Express.js:** A non-blocking, event-driven architecture perfectly suited for handling concurrent API requests to brokerage servers.
* **SQLite3:** Chosen for its low-latency local persistence, handling over **5,000+ daily data points** with minimal overhead compared to traditional client-server databases.
* **Memcached:** Integrated for high-speed caching of OAuth access tokens and session data, reducing database hits and improving API response times.

### 3. Brokerage & Automation Layer

* **Charles Schwab API (v1):** Deep integration for order placement, account polling, and option chain data.
* **Puppeteer/Playwright:** Engineered a sophisticated headless browser automation suite to handle the complex Schwab OAuth2 handshake, including automated login and consent flows to maintain 24/7 connectivity.
* **OAuth2 / JWT:** Dual-layered security. JWT manages Agent-to-Platform sessions, while OAuth2 (with automated refresh logic) manages Platform-to-Brokerage sessions.

---

## Infrastructure & DevOps

The platform is deployed on a hardened **Ubuntu 22.04 LTS** server environment using a modern CI/CD mindset.

### 1. Nginx Reverse Proxy

Nginx acts as the "Front Door" of the system:

* **SSL/TLS Termination:** Secured via Certbot (Let's Encrypt) to ensure all financial data is encrypted in transit.
* **Static File Serving:** Optimized delivery of the React `dist` build.
* **API Routing:** Transparently proxies `/api` requests to the Node.js backend cluster running on internal ports.

### 2. PM2 Process Management

To ensure 99.9% uptime, the system uses PM2 to manage a cluster of four independent processes:

* `traderswim-api`: The main Express.js server handling user interactions.
* `traderswim-cron-account`: A background worker dedicated to balance and buying power synchronization.
* `traderswim-cron-option`: A specialized engine for monitoring multi-leg option fills.
* `traderswim-cron-stock`: A worker for real-time equity position tracking.

---

## Core Functional Modules & Dashboard Logic

The platform is divided into specialized modules that handle the lifecycle of a trade—from construction and "Fan-Out" broadcasting to real-time status tracking.

### 1. The "Fan-Out" Multi-Account Broadcast Engine

The core innovation of the platform is the **Broadcast Engine**. When an Agent initiates a trade through the UI, the system does not place a single order; instead, it performs a **Synchronized Fan-Out**:

* **Active Account Filtering:** The backend queries the SQLite database to identify all accounts marked as "Active" by the user.
* **Payload Serialization:** A single, standardized order payload is generated (e.g., "BUY 10 AAPL" or "IRON CONDOR SPY").
* **Concurrent Execution:** Using Node.js asynchronous `Promise.all` logic, the system broadcasts the order to the Charles Schwab API for every active account simultaneously. This minimizes "Execution Slippage" and ensures the entire $1M+ portfolio enters the market at nearly the same price point.

### 2. Specialized Derivatives Strategy Builder

As seen in the **Vertical Option Trading Dashboard**, the system provides a high-fidelity interface for constructing complex, multi-leg derivative strategies.

* **Vertical Spreads (2-Legs):** Automated construction of Bull/Bear Call and Put spreads.
* **Iron Condors (4-Legs):** A specialized builder that allows users to define "Wings" and "Body" strikes simultaneously.
* **Atomic Unit Treatment:** The system treats these as single "Complex Orders" in the Schwab API, ensuring all legs are filled together, which eliminates the risk of "Legging-In" (where one side fills but the other does not).

### 3. Smart Exit Logic & Strategy Reversal

One of the most powerful features of the platform is the **Saved Placed Order Sets** (Middle Pane of the Vertical Trading Dashboard).

* **One-Click Liquidation:** When a complex strategy is filled, the system "bookmarks" it. The user can simply click the **"EXIT"** button next to a strategy set.
* **Automated Instruction Flipping:** The backend parses the original order legs and automatically generates the exact opposite instructions (e.g., flipping `BUY_TO_OPEN` to `SELL_TO_CLOSE` and `NET_CREDIT` to `NET_DEBIT`) to close the position instantly across all accounts.

---

## Dashboard Breakdown (Visual Architecture)

Based on the production screenshots, the platform features a "mission-control" style layout designed for rapid data consumption.

### A. Vertical Option Trading Dashboard

This high-density view is split into three functional panes to manage the entire lifecycle of a derivatives trade:

* **Left Pane (Live Orders):** Displays active orders with real-time status. Key columns include *Account*, *Strategy Type*, *Legs & Fill Prices*, and the critical *Status (per account)* indicator, which shows the progress of the trade across the entire portfolio.
* **Middle Pane (Saved Placed Order Sets):** A chronological list of strategies currently in the market. Each entry features an **"EXIT"** button for immediate portfolio-wide liquidation.
* **Right Pane (Order Entry & Templates):** Houses the **Saved Order Panel** for one-click entry into frequent trades and the **"OPTION"** button which launches the modal for the Spreads and Iron Condor builders.

### B. Complex Order History & Status Tracking

The **Complex Order History** page provides a granular audit trail for every strategy executed.

* **Visual Status Grid:** Instead of a single status, each strategy entry shows a grid of icons representing every account. A quick glance tells the Agent if the strategy is `FILLED`, `WORKING`, or `REJECTED` across the entire 45-account fleet.
* **Leg & Price Parsing:** The **Legs & Fill Prices** column uses a custom-built parser to convert raw OCC symbols (e.g., `SPXW 251205C07200000`) into human-readable text (e.g., `SPXW 12/05/2025 7200.00 C`), significantly reducing human error.

### C. Brokerage Account Management

The **Account Management** page is the foundation of the platform's multi-tenant capability.

* **Secure Credentialing:** Stores the *Account Hash*, *API Key*, and *API Secret* for each Schwab account.
* **Operational Status:** Features a "Status" toggle to instantly activate or deactivate an account. Inactive accounts are automatically bypassed by the "Fan-Out" engine, allowing for precise capital management.
* **Real-Time Sync:** The **"REFRESH"** button triggers an immediate balance and buying power update across all accounts via the background `AccountProcess` worker.

---
