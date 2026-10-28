# Anvaya | Attendance Management Platform

Anvaya is an ERP-style attendance management portal featuring a verifiable cryptographic ledger (simulating Client-Server, Peer-to-Peer, and Hybrid storage models).

---

## 📁 Project Structure

```
anvaya/
├── index.html     # Complete client application (UI, logic, cryptographic verification)
├── server.js      # Zero-dependency Node.js HTTP server
├── package.json   # NPM scripts and project metadata
└── README.md      # Setup, architecture, and credentials guide
```

---

## 🚀 Running Locally

### Option 1: Using Node.js
```bash
node server.js
```
or
```bash
npm start
```
Then visit: **http://localhost:3000**

### Option 2: Standalone Browser File
Because `index.html` is self-contained with Vanilla CSS and JS, you can also double-click `index.html` to open it directly in any browser.

---

## 🔑 Demo Accounts

All demo accounts use the default password: **`anvaya@123`**

| Role | User ID | Name | Capabilities |
| :--- | :--- | :--- | :--- |
| **Administrator** | `ADM-001` | Registrar Office | Set storage model (CS / P2P / Hybrid), audit ledger integrity, reset user passwords |
| **Teacher** | `FAC-117` | Dr. Meera Iyer | Take roll call, seal attendance blocks to ledger, view class reports |
| **Student** | `STU-2401` | Ananya Reddy | Attendance percentage ring, course sessions breakdown, verify block hashes |

*(Additional student accounts: `STU-2402` through `STU-2412`)*

---

## 🌐 Deploying to the Public Web (Free)

- **Netlify Drop**: Drag and drop the `anvaya/` folder to [app.netlify.com/drop](https://app.netlify.com/drop).
- **GitHub Pages**: Push this directory to a GitHub repository and turn on GitHub Pages under **Settings > Pages**.
- **Vercel**: Run `npx vercel` inside this folder.
