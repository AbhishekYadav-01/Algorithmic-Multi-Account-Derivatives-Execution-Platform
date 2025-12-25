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
