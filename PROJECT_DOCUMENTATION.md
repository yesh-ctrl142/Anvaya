# Anvaya | Attendance Management System
## Comprehensive Project Documentation & Technical Notes

---

## 1. Executive Summary

**Anvaya** is a next-generation academic ERP and attendance management platform built with an integrated, verifiable cryptographic ledger. It addresses the vulnerabilities of traditional college ERP systems—such as unauthorized retrospective attendance alteration, database tampering, disputed shortage claims, and lack of transparency—by combining an intuitive, accessible college portal with the immutable auditability of blockchain data structures.

---

## 2. Problem Statement & Motivation

### The Challenge with Traditional ERPs
1. **Database Mutability & Tampering**: Conventional attendance tracking stores records in mutable relational database rows (e.g., MySQL or Postgres). A database administrator, privileged staff member, or compromised credential can alter records retrospectively without leaving a tamper-evident audit trail.
2. **Disputed Eligibility Claims**: Students often dispute attendance shortages when hall tickets are barred before exams. Without indisputable timestamped proofs, mediation relies on verbal testimony or paper records.
3. **Single Point of Failure**: Centralized servers can experience downtime, data loss, or internal corruption.
4. **Proxy Attendance & Roll-Call Inaccuracies**: Manual paper sheets lead to proxy marking; disconnected biometric devices suffer from network latency and sync lags.

### The Anvaya Solution
- **Tamper-Evident Ledger**: Every class session's roll call is sealed into a sequentially hashed block containing the SHA-256 hash of the previous block (`prev`).
- **Instant Ledger Verification**: Students and administrators can verify the mathematical integrity of every record at any time. Any retrospective change breaks the cryptographic hash chain.
- **Architectural Flexibility**: Evaluates and demonstrates three distinct operational models:
  - **Client-Server (CS)**: Fast and low cost.
  - **Peer-to-Peer (P2P)**: Fully decentralized and immutable, but higher latency/cost.
  - **Hybrid (HY)**: High-speed permissioned channel (e.g., Hyperledger Fabric) with periodic public mainnet anchoring.

---

## 3. Core Architecture & Storage Models

Anvaya models real-world enterprise infrastructure trade-offs by supporting three operational paradigms:

| Metric | Client-Server (CS) | Peer-to-Peer (P2P) | Hybrid (HY) [Recommended] |
| :--- | :--- | :--- | :--- |
| **Storage Medium** | MySQL Database Row | Public On-chain Tx | Permissioned Fabric Channel + Anchor |
| **Average Latency** | ~60 ms | Seconds to Minutes (~3.5 s sim) | ~1 to 2 s (~1.4 s sim) |
| **Cost per Roll Call** | ₹0.002 | ₹0.66 | ₹0.01 |
| **Tamper Resistance** | Low (Internal Trust) | Absolute (Decentralized Consensus) | Enterprise-Grade (Anchored proofs) |
| **Anchoring Cadence** | N/A | Every Transaction | 1 Public Anchor per 5 local blocks |

### Cryptographic Block Schema
Each attendance submission seals a block with the following data schema:
```typescript
interface AttendanceBlock {
  i: number;              // Sequential Block Index (0 = Genesis)
  course: number;         // Course identifier index (0 to 5)
  session: number;        // Session number for this course (1, 2, 3...)
  date: string;           // Formatted date (e.g. "03 Aug")
  marks: boolean[];       // Attendance bit array (1 = Present, 0 = Absent)
  prev: string;           // 64-character SHA-256 hash of block i - 1
  hash: string;           // SHA-256 hash of this block's contents
  model: 'cs'|'p2p'|'hy'; // Storage model used when sealing
  fee: number;            // Simulated network transaction fee (INR)
  ms: number;             // Processing and confirmation time (ms)
  ok: boolean;            // Verification status (valid / altered)
}
```

### Content Hash Calculation
$$\text{Content} = i \parallel \text{course} \parallel \text{session} \parallel \text{date} \parallel \text{marks\_string} \parallel \text{prev}$$
$$\text{Hash} = \text{SHA-256}(\text{Content})$$

If any single student's attendance bit is toggled after the block is sealed, recomputing $\text{SHA-256}(\text{Content})$ will not match $\text{hash}$, instantly flagging the block as **Altered**.

---

## 4. Role-Based Access Control (RBAC) & Capabilities

### 🎓 1. Student Portal
- **Dashboard Overview**:
  - Live **Attendance Percentage Ring** with dynamic colour coding (Green for $\ge 75\%$, Red for $< 75\%$).
  - Overall sessions attended vs. held.
  - Alert banner indicating courses with attendance shortage.
  - Today's schedule/timetable with marked/scheduled status.
  - Institutional notices (exam criteria, room changes).
- **Course Matrix View ("My Courses")**:
  - Interactive session grid for each course.
  - Color-coded cells: Green (Present), White (Absent), Red Border (Tampered/Failed Verification).
  - **Inspect Block Mode**: Clicking any session square opens cryptographic details showing Block #, Date, Presence status, Current Block Hash, Previous Hash, and Verification Status.
- **Account Management**: Self-service password change.

### 👩‍🏫 2. Teacher Portal
- **Mark Attendance**:
  - Course selection dropdown showing current session count.
  - Quick roster with student avatars, names, and current attendance %.
  - Bulk actions: **"All Present"** and **"All Absent"** buttons.
  - Sealing indicator showing active storage model, storage target, and simulated confirmation delay.
  - Recent blocks table displaying the last 5 sealed records.
- **Class Reports**:
  - Per-course aggregate statistics: Class Average, Students in Shortage, Sessions Held.
  - Student-by-student attendance progress bar with eligibility status (Eligible vs Shortage).
- **Account Management**: Password change.

### 🛡️ 3. Administrator Portal
- **Storage Model Controller**:
  - Switch active storage engine between **Client-Server**, **Peer-to-Peer**, and **Hybrid**.
  - System-wide ledger analytics: Total Blocks Sealed, Total Marks Recorded, Cumulative Fees Spent, Public Anchors count.
  - Fee and block breakdown table by storage model.
- **Ledger Audit & Verification ("Ledger")**:
  - Chronological block explorer displaying Block #, Course Code, Session, Date, Attendance Ratio, Storage Model, Hash, and Previous Hash.
  - **"Verify Chain" Engine**: Traverses the chain from Genesis to the latest block, recomputes each SHA-256 hash, and verifies link integrity ($b[i].\text{prev} == b[i-1].\text{hash}$). Flags any broken link.
  - Administrative purge option ("Clear all records").
- **User Account Management**:
  - Comprehensive user directory (Admins, Teachers, Students).
  - One-click password reset to institutional default (`anvaya@123`).

---

## 5. Course & User Dataset Specifications

### Courses Configured
| Code | Course Title | Venue | Sessions Seeded |
| :--- | :--- | :--- | :--- |
| **CS401** | Distributed Systems | Room 204 | 14 sessions |
| **CS402** | Database Systems | Room 118 | 14 sessions |
| **CS403** | Computer Networks | Lab 3 | 14 sessions |
| **CS404** | Machine Learning | Room 301 | 14 sessions |
| **CS405** | Software Engineering | Room 120 | 14 sessions |
| **CS406** | Blockchain Technology | Room 214 | 14 sessions |

### Demo Credentials Directory
*Default Password for all accounts:* **`anvaya@123`**

| Role | User ID | Name / Description |
| :--- | :--- | :--- |
| **Admin** | `ADM-001` | Registrar Office |
| **Teacher** | `FAC-117` | Dr. Meera Iyer |
| **Student** | `STU-2401` | Ananya Reddy |
| **Student** | `STU-2402` | Karthik Rao |
| **Student** | `STU-2403` | Fatima Sheikh |
| **Student** | `STU-2404` | Rohan Mehta |
| **Student** | `STU-2405` | Priya Nair |
| **Student** | `STU-2406` | Aditya Verma |
| **Student** | `STU-2407` | Sneha Kulkarni |
| **Student** | `STU-2408` | Vikram Singh |
| **Student** | `STU-2409` | Meghana Pillai |
| **Student** | `STU-2410` | Arjun Das |
| **Student** | `STU-2411` | Lakshmi Iyer |
| **Student** | `STU-2412` | Imran Qureshi |

---

## 6. Technical Stack & Implementation Details

- **Frontend Core**: HTML5 Semantic markup, JavaScript (ES6+).
- **Styling Architecture**:
  - Vanilla CSS design system using CSS Custom Properties (`--paper`, `--panel`, `--line`, `--ink`, `--top`, `--brand`, `--cs`, `--p2p`, `--hy`).
  - Automatic and manual dark/light theme switching (`color-scheme: light dark`).
  - Responsive layouts: Grid and Flexbox with media queries down to mobile viewports.
  - Typography: Google Fonts (`Inter` for display and body, `IBM Plex Mono` for hashes and data labels).
- **Cryptography Engine**:
  - Native browser `crypto.subtle.digest('SHA-256', buffer)` for fast, hardware-accelerated cryptographic hashing.
  - Fallback deterministic hashing implementation for offline/legacy environments.
  - Password security: Salted SHA-256 hash (`id + ":" + password`).
- **Data Persistence**:
  - `localStorage` under namespace `anvaya.v1.` (`anvaya.v1.chain`, `anvaya.v1.acc`, `anvaya.v1.model`).
  - `sessionStorage` for active session token authentication.
- **Backend Server**:
  - Lightweight Node.js server (`server.js`) using built-in `http`, `fs`, and `path` modules (zero external npm dependencies).
  - Port forwarding and CORS compatibility for local and tunneled network environments.

---

## 7. Viva & Technical Interview Questions (Q&A)

### Q1: Why use a cryptographic ledger for college attendance?
> **Answer**: Traditional databases allow anyone with administrative access to silently update rows. In a collegiate setting where a 75% attendance threshold determines eligibility for final examinations, disputes frequently arise. A cryptographic ledger ensures that every attendance submission is sealed with a cryptographic hash linked to previous sessions. Retroactive modification becomes mathematically impossible without invalidating the entire subsequent chain.

### Q2: What is the benefit of the Hybrid Model over pure Peer-to-Peer?
> **Answer**: Pure Peer-to-Peer (like Ethereum or Bitcoin mainnet) has high transaction fees (₹0.66 per mark) and latency (3.5+ seconds). In contrast, the Hybrid model uses a private permissioned consortium channel (e.g., Hyperledger Fabric) for instant roll-call recording (₹0.01, 1.4s) and periodically anchors state roots (1 anchor per 5 blocks) to a public blockchain for third-party auditing.

### Q3: How does the "Verify Chain" feature detect tampered records?
> **Answer**: The verification engine iterates through all blocks. For each block, it recomputes $\text{SHA-256}(\text{block\_content})$ and checks if it matches `block.hash`, and verifies that `block[i].prev` equals `block[i-1].hash`. If an attacker changes an absent mark to present, the recomputed hash diverges from the stored hash, immediately marking the block as **Altered**.

### Q4: How is user privacy handled?
> **Answer**: Passwords are never stored in plaintext; they are hashed via SHA-256 with the User ID as a distinct salt. Attendance records store boolean status bits and student index references rather than sensitive personal identifiers on-chain.

---

## 8. Summary Checklist & Usage Commands

```bash
# To run the local server:
node server.js

# To view the app:
Open http://localhost:3000 in your browser

# Default Login:
User ID: ADM-001 (Admin) or FAC-117 (Teacher) or STU-2401 (Student)
Password: anvaya@123
```
