<div align="center">
  <!-- Replace the 'src' link below with your actual logo URL -->
  <img src="https://cdn-icons-png.flaticon.com/512/3058/3058866.png" alt="Email Verifier API Logo" width="120" />

  <h1>🛡️ Email Verifier API</h1>
  
  <p><b>A self-hosted, multi-layer email verification engine built with Flask.</b></p>

  <p>
    <img src="https://img.shields.io/badge/Python-3.8+-blue.svg" alt="Python">
    <img src="https://img.shields.io/badge/Framework-Flask-black.svg" alt="Flask">
    <img src="https://img.shields.io/badge/Integration-n8n-ea4b71.svg" alt="n8n">
    <img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License">
  </p>
  <p>Built to replace expensive paid services like ZeroBounce or NeverBounce for personal, freelance, and automation workflows.</p>
</div>

---

## ⚡ What It Does

This API checks every email through **4 distinct layers** before it ever reaches your sending queue, protecting your SMTP server and domain reputation. It returns `valid`, `risky`, or `invalid` with a specific reason for each email.

| Layer | Verification Check | Example Catch | Action |
| :--- | :--- | :--- | :--- |
| **1** | **Syntax Validation** | `user@` | ❌ Invalid |
| **2** | **Disposable Domain** | `mailinator.com` | ❌ Invalid |
| **3** | **Role-Based Prefix** | `info@`, `admin@` | ⚠️ Risky |
| **4** | **Live MX + SMTP Check** | Non-existent mailbox | ❌ Invalid |

---

## 📂 File Structure

```text
├── verify-app.py     # Flask API — core verification logic
├── index.html        # Browser UI — drag-and-drop CSV uploader
├── requirements.txt  # Python dependencies
├── Procfile          # Railway deployment config
└── README.md         # Documentation
