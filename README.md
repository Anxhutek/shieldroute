# 🛡️ ShieldRoute: Enterprise-Grade Link Protection & Anti-Bypass Proxy Gate

<p align="center">
  <img src="https://capsule-render.vercel.app/render?type=waving&color=auto&height=200&section=header&text=ShieldRoute&fontSize=50&desc=Enterprise-Grade%20Link%20Protection%20%26%20Anti-Bypass%20Proxy%20Gate&descAlignY=70&descSize=20" alt="ShieldRoute Banner" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="NodeJS" />
  <img src="https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white" alt="Express" />
  <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" alt="Redis" />
  <img src="https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Razorpay-0B1E47?style=for-the-badge&logo=razorpay&logoColor=white" alt="Razorpay" />
  <img src="https://img.shields.io/badge/PM2-2B037A?style=for-the-badge&logo=pm2&logoColor=white" alt="PM2" />
  <img src="https://img.shields.io/badge/EJS-B35F3B?style=for-the-badge&logo=ejs&logoColor=white" alt="EJS" />
</p>

---

ShieldRoute is a high-performance link-gating and security proxy system designed to protect monetized destination links from automated web scrapers, bypass extensions, and bot extraction.

> [!NOTE]
> This repository contains the system architecture, database design patterns, and engineering documentation for ShieldRoute. The core source code is proprietary.

---

## ⚙️ System Architecture & Redirection State Machine

ShieldRoute ensures that final destination URLs are never exposed in frontend scripts, server logs, or intermediate page HTML.

```mermaid
sequenceDiagram
    autonumber
    actor User as "Client / Bot"
    participant S as "Server (/s/:rid)"
    participant R as "Redis Cache"
    participant B as "Bridge (/bridge)"
    participant D as "MongoDB (Analytics)"

    User->>S: Access Protected Link (/s/:rid)
    Note over S: Validate Request Origin 
 (Referrer & UA Validation)
    S->>R: Generate & Store Temp Ticket (TTL 5 mins)
    S-->>User: Set Temporary Signed Session & Redirect to /bridge
    User->>B: Access Gate / Countdown Page
    B->>R: Verify Temporary Ticket Validity
    Note over B: Client-Side Anti-Tampering Checks
    User->>B: Finish Gate Timer (/api/bridge/finish)
    B->>R: Invalidate Temp Ticket & Issue Final One-Time Ticket
    B-->>User: Secure Token Signed Redirect (/go/:ticket)
    User->>S: Call Redirect Resolver (/go/:ticket)
    S->>R: Consume & Delete Ticket (Single-Use Enforcement)
    S->>D: Log Access Metrics & IP Hash Logs
    S-->>User: 302 Redirect to Destination URL
```

---

## 🔑 Special Authentication & Onboarding Flow

To prevent fake registrations and ensure high platform integrity, ShieldRoute employs several specialized authentication flows:

### 1️⃣ Dual-Method Authentication (Password & Passwordless OTP)
* **Password Login:** Standard credential validation matching cryptographically hashed passwords (`sha256` with custom `HASH_SALT` configuration).
* **Passwordless Email OTP:** Users can request a one-time 6-digit verification code sent to their registered email. The OTP is stored in Redis with a 5-minute TTL to enforce fast expiry.

### 2️⃣ Razorpay Automated Payment Gateway & Billing
* **Checkout Flow:** Integrated **Razorpay API** to handle automated subscriptions, generating dynamic payment orders and opening standard checkout forms.
* **Webhook Signature Verification:** Uses secure webhooks with SHA-256 HMAC verification to process payment success/failure events dynamically, updating the user's subscription tier on callback.
* **Transaction Logging:** Integrates directly with the billing engine to transition pending accounts to active instantly, logging Razorpay payment IDs, order IDs, and payment responses to MongoDB.
* **Fallback Mailer Notifications:** Fires automated SMTP receipts and trial activation confirmations instantly upon successful payment completion.

### 3️⃣ Dynamic Credential Pre-filling & Auto-routing
* Upon successful registration and OTP confirmation, the frontend dynamically switches the active panel state and pre-fills the login form with the verified username, maximizing user experience.

---

## 📈 Engineering Achievements & Portfolio Highlights

* 🛡️ **State-Machine Redirect Validation:** Built a multi-step token validation flow (`/s/:rid` ➔ `/bridge` ➔ `/go/:ticket`) that completely removes destination URLs from the DOM, thwarting automated bypass browser extensions.
* ⚡ **High-Speed Caching Layer:** Selected **Redis** for transient state (caching short-lived redirect tickets, rate-limiting, and email OTPs) ensuring sub-millisecond response times.
* 💾 **Dual-Database Strategy:** Paired Redis with **MongoDB** for long-term analytics logs, audit tracking, referrer configurations, and billing history.
* 🕵️ **IP-Fingerprint Tampering Detection:** Checks request headers (User-Agent, IP hashes) at both validation gates to catch and block ticket reuse or spoofing.

---

## 💻 Tech Stack & Deployment

* **Backend Runtime:** Node.js (Express)
* **Caching & State Storage:** Redis
* **Data Persistence:** MongoDB
* **View Engine:** EJS (Dynamic server-rendered HTML layouts)
* **Email Engine:** SMTP (Nodemailer)
* **Production Hosting:** Hosted on Ubuntu Linux VPS, managed via PM2 process supervisor, with dynamic robots/sitemap compliance.

---
<p align="center">
  *Vibes are temporary, good system design is forever. Let's build!* 🚀
</p>
