[README.md](https://github.com/user-attachments/files/27219648/README.md)
# Tenstorrent Supplier Qualification Form

A globally accessible supplier onboarding and qualification system built for **Tenstorrent Global Strategic Sourcing**. Designed to replace geo-blocked Google Forms — works for any supplier worldwide with no Google account required.

---

## Overview

| Component | Technology | Purpose |
|---|---|---|
| `index.html` | HTML / JavaScript | Supplier-facing form (hosted on GitHub Pages) |
| `Code.gs` | Google Apps Script | Backend — receives submissions, writes to Sheet, sends emails, manages follow-up |
| Google Sheet | Google Sheets | Response database in shared Supply Chain Drive |
| Google Drive | Drive API | Per-supplier document folders |

---

## Form Features

### Short Form vs. Full Qualification
The form opens in **Short Form** mode by default (~5 min) for initial vendor onboarding. A toggle at the top switches to **Full Qualification** mode (~15 min) for strategic or sole-source suppliers.

| | Short Form | Full Qualification |
|---|---|---|
| Time to complete | ~5 min | ~15 min |
| Sections | 1–7 | 1–12 |
| Use case | New supplier first contact | Strategic / sole-source qualification |
| Quality questions | 6 core yes/no | 6 core + 8 extended |
| Manufacturing & Capacity | — | ✅ 13 questions |
| Schedule Management | — | ✅ 5 questions |
| Commercial & Contract | — | ✅ 4 questions |
| Customer Support | — | ✅ 5 questions |

### Sections Covered
1. Company Information
2. Contacts (submitter + GM, PM, Quality, AP)
3. Business Profile (capacity, customers, ownership)
4. Quality System (certifications + yes/no checklist)
5. Regulatory & Ethical Compliance (RoHS, REACH, FAR 52.222-50, UFLPA)
6. Supply Chain & Audit (serialization, CoC, audit access)
7. Documents & File Uploads
8. *(Full only)* Quality Extended
9. *(Full only)* Manufacturing & Capacity
10. *(Full only)* Schedule Management
11. *(Full only)* Commercial & Contract
12. *(Full only)* Customer Support & Partnership

### Compliance Flags
The following responses are automatically flagged in the Sheet (red row highlight + cell note) and called out in the notification email:
- FAR 52.222-50 — Non-compliant
- UFLPA — Non-compliant
- RoHS/REACH — Non-compliant
- Site audit access declined

---

## Backend — Google Apps Script (`Code.gs`)

### Key Functions

| Function | When to Run | Purpose |
|---|---|---|
| `setupSpreadsheetAndFolder()` | **Once, on first deploy** | Creates `Vendor Information` folder in shared Drive + response Sheet with styled headers |
| `setupTriggers()` | **Once, on first deploy** | Installs daily 8am cron job for pending qualification checks |
| `doPost(e)` | Auto (on form submit) | Receives form data, writes to Sheet, creates supplier folder, sends emails |
| `doGet(e)` | Auto (on action link click) | Handles `?action=remind` and `?action=ignore` from digest emails |
| `checkPendingQualifications()` | Auto (daily cron) | Scans for short-form submissions past the 30-day threshold |
| `refreshHeaders()` | As needed | Re-writes header row if columns are changed |
| `testSubmission()` | Testing | Simulates a full form submission end-to-end |
| `testPendingCheck()` | Testing | Runs the pending qualification scan immediately |

### Email Flows

```
Vendor submits Short Form
        │
        ├─► Warren receives: New submission notification + risk flags + Drive folder link
        │
        └─► Vendor receives: Confirmation email with Ref ID + full qualification note

After 30 days (if Full Qualification not received)
        │
        └─► Warren receives: Weekly pending digest with action buttons
                    │
                    ├─► [Send Reminder to Vendor] → Vendor gets reminder email with form link
                    │                              → Sheet updated: status = "Reminder Sent", count +1
                    │
                    └─► [Ignore] → Sheet updated: status = "Ignored — No Follow-up Required"
                                 → No further reminders sent
```

### Qualification Status Values (Sheet Column)

| Status | Meaning |
|---|---|
| `Short — Pending Full Qualification` | Short form received, full qualification not yet done |
| `Full Qualification Complete` | Full qualification form submitted |
| `Reminder Sent` | Warren clicked "Send Reminder"; vendor notified |
| `Ignored — No Follow-up Required` | Warren clicked "Ignore"; no further alerts |

---

## Setup & Deployment

### First-Time Setup (run once)

1. Open [script.google.com](https://script.google.com) → your project
2. Paste `Code.gs` content
3. Run **`setupSpreadsheetAndFolder()`** — authorise when prompted
4. Run **`setupTriggers()`** — installs daily cron
5. **Deploy → New deployment → Web App**
   - Execute as: `Me`
   - Who has access: `Anyone` ← must be Anyone, not "Anyone at Tenstorrent"
6. Copy the deployment URL
7. Paste into `index.html` where it says:
   ```javascript
   const APPS_SCRIPT_URL = 'YOUR_APPS_SCRIPT_DEPLOYMENT_URL_HERE';
   ```
8. Update `FORM_URL` in `Code.gs` CONFIG with your GitHub Pages URL

### Re-deploying After Changes
**Deploy → Manage deployments → Edit (pencil) → New version → Deploy**  
Then hard refresh your browser (`Ctrl+Shift+R` / `Cmd+Shift+R`).

### GitHub Pages Hosting
1. Upload `index.html` to this repo (root directory)
2. **Settings → Pages → Source: main branch / root**
3. Your public URL: `https://[org].github.io/tenstorrent-supplier-form/`

---

## Configuration

All key settings are in the `CONFIG` object at the top of `Code.gs`:

```javascript
const CONFIG = {
  PARENT_FOLDER_ID:       '1XRmw1oVbbJtiXUb3BrwxtOXnVL7osavP', // Supply Chain - EXT shared drive
  SUBFOLDER_NAME:         'Vendor Information',
  SHEET_NAME:             'Supplier Responses',
  NOTIFY_EMAIL:           'wvandrine@tenstorrent.com',           // Update to add team members
  TIMEZONE:               'America/Toronto',
  FORM_URL:               'YOUR_GITHUB_PAGES_URL',               // Update after GitHub Pages setup
  DAYS_BEFORE_FIRST_FLAG: 30,                                    // Days before first pending alert
  FLAG_FREQUENCY_DAYS:    7,                                     // Weekly re-flag cadence
};
```

To add additional notification recipients, change `NOTIFY_EMAIL` to a comma-separated string:
```javascript
NOTIFY_EMAIL: 'wvandrine@tenstorrent.com, colleague@tenstorrent.com',
```

---

## Drive Folder Structure

```
Supply Chain - EXT (shared drive)
└── Vendor Information/
    ├── Supplier Responses (Google Sheet)
    ├── TestVendorCo_TT-ABC123/
    ├── AnotherSupplier_TT-DEF456/
    └── ...
```

Each supplier submission creates a dedicated subfolder named `SupplierName_RefID` for document uploads and correspondence.

---

## Owned By

**Warren Van Drine** — Sr. Manager, Global Strategic Sourcing  
Tenstorrent Inc. · Operations: Supply Chain  
`wvandrine@tenstorrent.com`
