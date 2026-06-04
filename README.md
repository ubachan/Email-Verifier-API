Email Verifier API
A self-hosted email verification API built with Flask. Verifies emails through multiple layers — syntax, disposable domain detection, role-based prefix filtering, and live MX/SMTP checks — before sending any campaign emails.
Built to replace paid services like ZeroBounce or NeverBounce for personal and freelance use.
---
What It Does
Checks every email through 4 layers before it reaches your sending queue:
Layer	Check	Example catch
1	Syntax validation	`user@` → invalid
2	Disposable domain	`mailinator.com` → invalid
3	Role-based prefix	`info@`, `admin@` → risky
4	MX + SMTP check	Non-existent mailbox → invalid
Returns `valid`, `risky`, or `invalid` with a reason for each email.
---
Files
```
├── verify-app.py     # Flask API — core verification logic
├── index.html        # Browser UI — drag-and-drop CSV uploader
├── requirements.txt  # Python dependencies
├── Procfile          # Railway deployment config
└── README.md
```
---
Local Setup
```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/email-verifier.git
cd email-verifier

# 2. Create virtual environment
python3 -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the API
python verify-app.py
# → Running on http://localhost:5050

# 5. Open the UI (separate terminal)
python3 -m http.server 3000
# → Open http://localhost:3000/index.html
```
---
API Reference
`POST /verify`
Upload a CSV file for bulk verification.
Request:
```bash
curl -X POST http://localhost:5050/verify \
  -F "file=@leads.csv"
```
Response:
```json
{
  "job_id": "abc-123-def"
}
```
CSV must have an `email` column. Other columns are preserved in output.
---
`GET /progress?job_id=<id>`
Check verification progress.
```json
{
  "percent": 72,
  "row": 72,
  "total": 100
}
```
---
`GET /download?job_id=<id>&type=<filter>`
Download results as CSV.
`type` value	Returns
`all`	Every row with status + reason
`valid`	Valid emails only
`risky`	Risky emails only
`risky_invalid`	Risky + invalid combined
```bash
curl "http://localhost:5050/download?job_id=abc-123&type=valid" -o valid-leads.csv
```
---
`POST /cancel?job_id=<id>`
Cancel a running job.
---
Deploy to Railway
Push this repo to GitHub
Go to railway.app → New Project → Deploy from GitHub repo
Select this repo — Railway auto-detects the `Procfile`
Done. You get a permanent public URL like `https://your-app.up.railway.app`
> **Note:** Update `verify-app.py` last line before deploying:
> ```python
> # Change this:
> app.run(debug=True, port=5050)
>
> # To this:
> import os
> app.run(debug=False, host='0.0.0.0', port=int(os.environ.get('PORT', 5050)))
> ```
---
Use with n8n
After deploying to Railway, call your API from an n8n HTTP Request node:
Single email verify:
```
Method : POST
URL    : https://your-app.up.railway.app/verify
Body   : form-data → file: (your CSV)
```
Recommended n8n flow:
```
Code Node (syntax + disposable + role)
    ↓
HTTP Request → Google DNS API (MX check)
    ↓
HTTP Request → This API (SMTP check)
    ↓
IF Node → valid / invalid branch
```
---
Email Status Meanings
Status	Meaning	Action
`valid`	Passed all checks	Safe to send
`risky`	Role-based or SMTP uncertain	Send with caution
`invalid`	Bad syntax, fake domain, or rejected by server	Do not send
Reason codes
Reason	What happened
`bad_syntax`	Not a valid email format
`disposable_domain`	Temp email service detected
`role_based`	Generic prefix like info@, admin@
`no_mx`	Domain has no email server
`smtp_ok`	Server confirmed mailbox exists
`smtp_reject`	Server explicitly rejected (550)
`smtp_timeout`	Server didn't respond
`domain_accepts_all`	Catch-all domain — can't verify individual mailbox
---
Tech Stack
Python 3 + Flask — API server
dnspython — MX record resolution
smtplib — SMTP-level mailbox verification
Vanilla JS — Browser UI with drag-and-drop and live progress
---
License
MIT — free to use, modify, and deploy.
