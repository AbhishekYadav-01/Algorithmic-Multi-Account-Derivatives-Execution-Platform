---

# Algorithmic Multi-Account Derivatives Execution Platform

**PROPRIETARY NOTICE:** This project was architected, engineered, and deployed entirely by me during my tenure at **Softwired**. Complete source code sharing is strictly prohibited by company policy. This documentation serves as a comprehensive technical breakdown of the system's architecture, logic, and operational scale for professional review.

---

## Executive Summary

The **Algorithmic Multi-Account Derivatives Execution Platform** is a high-performance, full-stack trading ecosystem designed to manage a high-value portfolio of **$1,000,000+ Assets Under Management (AUM)**.

In a traditional trading environment, managing 45+ decentralized brokerage accounts requires a massive overhead of manual entry, leading to "execution drift" where fill prices vary wildly between accounts. This platform solves that problem by providing a centralized **"Command and Control"** interface. It allows a single operator to broadcast complex equity and multi-leg derivative strategies (Iron Condors, Vertical Spreads) across **45+ concurrent Charles Schwab accounts** with sub-200ms execution latency, ensuring near-perfect synchronization across the entire $1M+ portfolio.

<img width="1365" height="570" alt="image" src="https://github.com/user-attachments/assets/db68440c-9a43-406d-bb2e-66850db872fa" />

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
<img width="1349" height="602" alt="image" src="https://github.com/user-attachments/assets/ea546785-a3fb-4dfb-bd28-3f5a235f2cdc" />
* **Middle Pane (Saved Placed Order Sets):** A chronological list of strategies currently in the market. Each entry features an **"EXIT"** button for immediate portfolio-wide liquidation.
* **Right Pane (Order Entry & Templates):** Houses the **Saved Order Panel** for one-click entry into frequent trades and the **"OPTION"** button which launches the modal for the Spreads and Iron Condor builders.

<img width="1342" height="581" alt="image" src="https://github.com/user-attachments/assets/3359dbb1-f32d-4fa8-bdf7-2d9e74daf96c" />

### B. Complex Order History & Status Tracking

The **Complex Order History** page provides a granular audit trail for every strategy executed.

* **Visual Status Grid:** Instead of a single status, each strategy entry shows a grid of icons representing every account. A quick glance tells the Agent if the strategy is `FILLED`, `WORKING`, or `REJECTED` across the entire 45-account fleet.
* **Leg & Price Parsing:** The **Legs & Fill Prices** column uses a custom-built parser to convert raw OCC symbols (e.g., `SPXW 251205C07200000`) into human-readable text (e.g., `SPXW 12/05/2025 7200.00 C`), significantly reducing human error.
![8bc765ff-dbd9-4e87-a58d-e7bf5ba4019f](https://github.com/user-attachments/assets/0aca93fa-4ac3-4682-bf54-00ea7b7c43a4)

### C. Brokerage Account Management

The **Account Management** page is the foundation of the platform's multi-tenant capability.

* **Secure Credentialing:** Stores the *Account Hash*, *API Key*, and *API Secret* for each Schwab account.
* **Operational Status:** Features a "Status" toggle to instantly activate or deactivate an account. Inactive accounts are automatically bypassed by the "Fan-Out" engine, allowing for precise capital management.
* **Real-Time Sync:** The **"REFRESH"** button triggers an immediate balance and buying power update across all accounts via the background `AccountProcess` worker.

<img width="1353" height="507" alt="image" src="https://github.com/user-attachments/assets/d7947ab9-906d-4193-93c6-166336e58f1a" />

---

## Backend Engineering: The "Invisible Engine"

While the frontend provides the command interface, the backend is a high-availability environment designed to manage thousands of concurrent data points and maintain persistent connections with the Charles Schwab brokerage infrastructure.

### 1. The "Tri-Cron" Heartbeat Architecture

To ensure the system remains responsive, the background synchronization logic is decoupled from the main API thread. The system runs three specialized, independent **Node-Cron** services:

* **Stock Synchronization Service (`cronStockCopyTradingProcess.js`):**
* **Frequency:** Executes every **5 seconds**.
* **Logic:** It iterates through all "Working" stock orders across 45+ accounts. It queries the Schwab API for execution updates and synchronizes the local SQLite database. This ensures that if a stock order fills on the brokerage side, the UI reflects it within seconds.


* **Option Strategy Synchronization Service (`cronOptionCopyTradingProcess.js`):**
* **Frequency:** Executes every **5 seconds**.
* **Logic:** This is the most complex worker. It monitors multi-leg option strategies (Verticals and Iron Condors). Because these strategies involve up to 4 legs, this worker verifies that *every leg* is filled before marking the strategy as `FILLED` in the **Complex Order History** view.


* **Account & Capital Monitor (`cronAccountProcess.js`):** * **Frequency:** Executes every **15 seconds**.
* **Frequency:** Executes every **15 seconds**.
* **Logic:** It fetches the "Total Account Value," "Cash Balance," and "Buying Power" for every linked account. This data is what drives the **Account Management Dashboard**, allowing the Agent to see a real-time aggregate of the **$1,000,000+ AUM**.



### 2. Smart Polling & "State Terminality" Optimization

A critical engineering challenge was avoiding "API Rate Limiting" from Charles Schwab while still providing real-time updates. I implemented a **Smart Polling** algorithm:

* **Terminal State Detection:** The system identifies orders that have reached a "Terminal State" (e.g., `FILLED`, `CANCELED`, `REJECTED`, `EXPIRED`, or `FAILED_TO_PLACE`).
* **Exclusion Logic:** Once an order is terminal, it is mathematically impossible for its status to change again. The backend and frontend logic automatically exclude these orders from the polling queue.
* **Result:** This optimization reduces unnecessary API calls by over **60%**, ensuring that the system's bandwidth is strictly dedicated to "Working" or "Pending" orders that require active monitoring.

### 3. Puppeteer-Based Authentication Orchestration

Standard OAuth2 flows often require manual user redirection to a browser. For a multi-account system, this is a major bottleneck. I engineered a **Headless Auth Engine**:

* **Browser Automation:** Using **Puppeteer**, the system launches a headless browser instance to navigate the Schwab login and consent screens automatically.
* **Persistent Session Caching:** Once the `access_token` and `refresh_token` are retrieved, they are cached in **Memcached**.
* **Silent Refresh:** A dedicated worker monitors the 30-minute expiration window of the access tokens and uses the refresh tokens to silently generate new credentials, ensuring the Agent never has to re-authenticate manually during market hours.


### 4. Relational Database Schema (SQLite3)

The data layer is designed for relational integrity and fast lookups. Key tables include:

* **`complexOrderHistory`:** The master record for strategies. It stores the strategy type (Iron Condor/Vertical) and the "Global Status" across all accounts.
* **`optionContractVerticalCopyTradingAccount`:** Stores the individual leg details for every account involved in a strategy.
* **`placedOrderSets`:** Acts as the "Trade Bookmark," storing the original JSON payload of a filled trade. This is what allows the **"EXIT"** button to function—it reads this table to know exactly which legs to reverse.
* **`accounts`:** Stores the encrypted API credentials and the "Active/Inactive" toggle state for each of the 45+ managed accounts.

---

## Visual Data Evidence (From Screenshots)

### The "Status (per Account)" Intelligence

In the **Vertical Copy Trading** and **History** screenshots, you can see the "Status" column. This is not a simple text field. It is a live-updating component driven by the **Tri-Cron** workers.

* If an Iron Condor fills in Account A but is still "Working" in Account B, the UI reflects this divergence instantly.
* The **Overall Status** component aggregates these account-level statuses into a single visual health indicator for the strategy.

### Human-Readable Strategy Parsing

The screenshots show legs like `SPXW 12/05/2025 7200.00 C`.

* **Backend Reality:** The Schwab API provides raw strings like `SPXW  251205C07200000`.
* **Parsing Logic:** I implemented a regex-based parser that breaks down these 21-character strings into Root, Expiration, Strike, and Option Type. This ensures that the Agent can read and verify trades at a glance, preventing catastrophic errors in high-stakes trading.


---

## Performance Metrics & System KPIs

The platform is engineered for institutional-grade reliability, handling significant capital with high execution precision.

### 1. Execution & Latency

* **Broadcast Latency:** Measured at **< 200ms** from the moment the "Place Order" button is clicked to the final API request sent for the 45th account. This is achieved through highly optimized asynchronous "Fan-Out" logic using `Promise.all`.
* **Sync Frequency:** Background workers process state updates every **5 seconds**, ensuring the dashboard remains a "live" representation of the brokerage environment.

### 2. Operational Scale

* **Assets Under Management (AUM):** Successfully managing **$1,000,000+ USD** across active user portfolios.
* **Transaction Volume:** Processing **10,000+ daily data points** (status checks, balance updates, and fill synchronizations).
* **High Availability:** Maintained **99.9% system uptime** through PM2's process monitoring and Nginx's resilient request handling.

---

## Security & Data Integrity

Given the high-value nature of the platform ($1M+ AUM), security is the primary architecture pillar.

### 1. Multi-Layered Authentication

* **Application Level:** State-of-the-art **JWT (JSON Web Tokens)** for secure Agent logins, with short-lived session windows and secure HTTP-only cookie storage.
* **Brokerage Level:** Implementation of **OAuth2 with PKCE** (Proof Key for Code Exchange) where possible, ensuring that brokerage credentials never touch the client-side code.
* **Headless Isolation:** The Puppeteer-based auth engine runs in a sandboxed environment on the server, preventing cross-site scripting (XSS) or session hijacking.

### 2. Infrastructure Hardening

* **Encryption in Transit:** Mandatory **SSL/TLS 1.3** encryption via Let’s Encrypt (Certbot), ensuring all order payloads and account balances are encrypted between the user and the server.
* **Environment Isolation:** All sensitive API keys, database paths, and Super-Admin credentials are kept in strictly isolated `.env` files, never hardcoded into the version control.
* **CORS Protection:** Cross-Origin Resource Sharing is strictly locked down to the production domain (`tradeswim.org`), preventing unauthorized API access from external sources.

---

## Key Engineering Achievements

* **The Reversal Algorithm:** Developed a custom logic engine that parses complex multi-leg JSON payloads from past trades to generate "Perfect Opposing Orders," effectively automating the liquidation of 4-leg Iron Condors in a single click.
* **Zero-Drift Synchronization:** Built a reliable Tri-Cron system that ensures 45+ accounts stay synchronized within a 5-second window, a feat usually reserved for high-frequency institutional software.
* **OCC Symbol Parser:** Solved the "Readability Gap" by engineering a regex-based parser that translates cryptic brokerage strings into human-readable text, reducing the risk of trader "fat-finger" errors.
* **Headless Auth Orchestrator:** Created a self-healing authentication system that uses Puppeteer to bypass manual login hurdles, allowing for truly automated 24/7 account management.

---

## Conclusion

The **Algorithmic Multi-Account Derivatives Execution Platform** represents a significant milestone in retail-integrated trading technology. By bridging the gap between high-level React/Node.js architecture and professional brokerage APIs, I have delivered a tool capable of managing over **$1 Million in live capital** with the precision and speed of institutional software.

This project demonstrates a mastery of the full-stack lifecycle: from the high-stakes logic of derivatives execution to the DevOps complexity of managing persistent, secure brokerage sessions at scale.

---
