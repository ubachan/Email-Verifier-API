<div align="center">
  <img src="https://cdn-icons-png.flaticon.com/512/3058/3058866.png" alt="Email Verifier API Logo" width="120" />
  <h1>🛡️ Email Verifier API</h1>
  <p><b>A self-hosted, multi-layer email verification engine built with Flask.</b></p>
  <p>
    <img src="https://img.shields.io/badge/Python-3.8+-blue.svg" alt="Python">
    <img src="https://img.shields.io/badge/Framework-Flask-black.svg" alt="Flask">
    <img src="https://img.shields.io/badge/Integration-n8n-ea4b71.svg" alt="n8n">
    <img src="https://img.shields.io/badge/License-Personal_Use-blue.svg" alt="License: Personal Use">
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
```

---

## 🚀 Local Setup

**1. Clone the repository**
```bash
git clone https://github.com/ubachan/Email-Verifier-API.git
cd Email-Verifier-API
```

**2. Create a virtual environment**
```bash
python3 -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

**4. Run the API Engine**
```bash
python verify-app.py
# → API Running on http://localhost:5050
```

**5. Open the UI (in a separate terminal)**
```bash
python3 -m http.server 3000
# → Open http://localhost:3000/index.html in your browser
```

---

## 📡 API Reference

### 1. Upload Leads for Verification
**`POST /verify`**

Upload a CSV file for bulk verification. The CSV must have an `email` column. Other columns are preserved in the final output.

**Request:**
```bash
curl -X POST http://localhost:5050/verify \
  -F "file=@leads.csv"
```

**Response:**
```json
{
  "job_id": "abc-123-def"
}
```

---

### 2. Check Job Progress
**`GET /progress?job_id=<id>`**

```json
{
  "percent": 72,
  "row": 72,
  "total": 100
}
```

---

### 3. Download Results
**`GET /download?job_id=<id>&type=<filter>`**

Download the verified results as a CSV file.

| `type` Parameter | Returns |
| :--- | :--- |
| `all` | Every row with status + reason |
| `valid` | Valid emails only |
| `risky` | Risky emails only |
| `risky_invalid` | Risky + Invalid combined |

**Request:**
```bash
curl "http://localhost:5050/download?job_id=abc-123&type=valid" -o valid-leads.csv
```

---

### 4. Cancel a Job
**`POST /cancel?job_id=<id>`**

Stops a currently running verification process.

---

## ☁️ Deploy to Railway (Production)

Deploy this API to the cloud in 3 minutes to use it as a webhook endpoint.

1. Push this repository to your GitHub.
2. Go to [Railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo**.
3. Select this repository. Railway auto-detects the `Procfile`.
4. You get a permanent public URL (e.g., `https://your-app.up.railway.app`).

> **⚠️ Critical — Update before deploying:**
>
> Change the last line of `verify-app.py` so it binds to Railway's dynamic port:
> ```python
> # REMOVE THIS:
> # app.run(debug=True, port=5050)
>
> # REPLACE WITH THIS:
> if __name__ == '__main__':
    import os
    app.run(debug=False, host='0.0.0.0', port=int(os.environ.get('PORT', 5050)))
> ```

---

## 🤖 n8n Integration

After deploying to Railway, plug this API into your AI automation workflows using the **HTTP Request** node.

**Node Configuration:**
```
Method    : POST
URL       : https://your-app.up.railway.app/verify
Body Type : Form-Data
Parameter : file → (Attach your CSV data)
```

**Recommended High-Performance Flow:**
```
[ Code Node: Syntax + Disposable + Role Checks ]
                      ↓
[ HTTP Request: Google DNS API (Fast MX Check) ]
                      ↓
[ HTTP Request: THIS API (Deep SMTP Check)     ]
                      ↓
[ Switch/If Node: Route to Valid / Invalid     ]
```

---

## 📊 Status Codes & Meanings

**Primary Statuses**

| Status | Meaning | Recommended Action |
| :--- | :--- | :--- |
| 🟢 `valid` | Passed all 4 layers of checks | Safe to send |
| 🟡 `risky` | Role-based or SMTP timeout | Send with caution |
| 🔴 `invalid` | Bad syntax, fake domain, or rejected | Do not send |

**Deep Reason Codes**

| Reason | What happened under the hood |
| :--- | :--- |
| `bad_syntax` | Not a valid email format |
| `disposable_domain` | Temporary/Trash email service detected |
| `role_based` | Generic prefix used (e.g., `info@`, `admin@`, `support@`) |
| `no_mx` | The domain has no active email server |
| `smtp_ok` | The receiving server confirmed the mailbox exists |
| `smtp_reject` | The receiving server explicitly rejected the email (550) |
| `smtp_timeout` | The receiving server took too long to respond |
| `domain_accepts_all` | Catch-all domain — cannot verify individual mailbox |

---

## 🛠️ Tech Stack

- **API Server:** Python 3 + Flask
- **Resolution:** dnspython (MX record routing)
- **Verification:** smtplib (SMTP-level mailbox pinging)
- **Frontend:** Vanilla JS (Browser UI with drag-and-drop & live progress)

---

## 📄 License

Personal Use License — Free to use, modify, distribute, and deploy for personal and educational projects only. Commercial use is strictly prohibited.

---

### 📬 Connect with Me

[<img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />](https://www.google.com/search?q=https://www.linkedin.com/in/uba-chan)
[<img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" />](mailto:aivibe@ubachan.site)


---

