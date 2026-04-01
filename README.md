# Monthly 1-on-1 Review Dashboard · 10x Growth

A fully self-contained, browser-based performance review tool built for the leadership team at **TranZact / 10x Growth**. No backend required — runs entirely from a single HTML file, syncs to Google Sheets, and is deployed via GitHub Pages.

---

## Live Links

| Page | URL |
|---|---|
| Landing Page | https://harikeshtiwari12.github.io/1on1-review/landing.html |
| Dashboard (base) | https://harikeshtiwari12.github.io/1on1-review/ |

### Personal Dashboard Links (direct login — no password required)

| Person | Role | Link |
|---|---|---|
| Ritesh Kumar | Admin · CEO | https://harikeshtiwari12.github.io/1on1-review/#ritesh |
| Sharad Sen Sharma | Admin · Co-Founder | https://harikeshtiwari12.github.io/1on1-review/#sharad |
| Rohan Sen Sharma | Admin · Co-Founder | https://harikeshtiwari12.github.io/1on1-review/#rohan |
| Atul Naharwara | Channel Partnerships | https://harikeshtiwari12.github.io/1on1-review/#atul |
| Adarsh Sikaria | Sales | https://harikeshtiwari12.github.io/1on1-review/#adarsh |
| Shabib Karjikar | Customer Experience | https://harikeshtiwari12.github.io/1on1-review/#shabib |
| Mudit Sharma | Growth Marketing | https://harikeshtiwari12.github.io/1on1-review/#mudit |

---

## What It Does

Leaders conduct monthly 1-on-1 performance reviews for every member of their team. Each review scores the employee on **Performance** (3 role-specific questions) and **Culture** (2 common questions), producing one of four outcomes:

| Outcome | Criteria | Action |
|---|---|---|
| 🟢 Rockstar | Good Perf + Good Culture | Recognition & Responsibilities |
| 🔵 WIP | Good Perf + Weak Culture | 3-Month Plan to Rockstar |
| 🟡 PIP | Weak Perf + Good Culture | 1-Month Warning |
| 🔴 Exit | Weak Perf + Weak Culture | Immediate Action |

> Good = average score ≥ 3.5 / 5 · Weak = average score < 3.5 / 5

---

## Architecture

```
index.html          ← The entire app (single file, no dependencies)
landing.html        ← Marketing/access landing page
```

**Data flow:**
1. Leader selects employee → rates 5 questions (1–5 scale) → adds notes → saves
2. Record saved to `localStorage` (instant, offline-safe)
3. Record simultaneously POSTed to Google Apps Script → written to Google Sheet
4. All data persists across sessions on the same browser

**Google Sheet:** `1EWdBpT0MhXIVibwsYKLb-5pbL01twTQxpFFNoZeseGI`
**Apps Script URL:** `https://script.google.com/macros/s/AKfycbyiWdeYAB9OjP7hZ8AkjDFIVSONWFoqMmw82QuT8y55fgXSr0ANETbjnHySm4PN8f6C/exec`

---

## Departments & Teams

| Department | Leader | Members |
|---|---|---|
| Channel Partnerships | Atul Naharwara | 5 members |
| Sales | Adarsh Sikaria | 9 members |
| Product | Sharad Sen Sharma | 2 members |
| Engineering | Rohan Sen Sharma | 12 members |
| Customer Experience | Shabib Karjikar | 13 members |
| Growth Marketing | Mudit Sharma | 4 members |
| Leadership Team | Ritesh Kumar | 6 members |

---

## Access Control

### URL-hash based auto-login
Each leader gets a unique URL (`#slug`) that:
- Bypasses the login screen entirely
- Locks the session to their department only
- Hides the "Switch user" button — they cannot access other teams

### Admin vs Leader permissions

| Capability | Leader | Admin |
|---|---|---|
| Rate own team | ✅ | ✅ |
| View own team records | ✅ | ✅ |
| View all teams | ❌ | ✅ |
| Export CSV (own team only) | ✅ | ✅ (all) |
| Edit records (own team) | ✅ | ✅ |
| Delete records (own team) | ✅ | ✅ |
| Department filter in Saved | ❌ hidden | ✅ |

---

## Features

### Review Flow
- **Period selector** — month + year dropdowns
- **Department selector** — locked for leaders, free for admins
- **Employee grid** — visual cards with done/active states
- **Role-specific questions** — each job role has 3 tailored performance questions
- **Common culture questions** — 2 questions asked to all employees
- **1–5 rating scale** — Needs Improvement → 10x Ready
- **Live score calculation** — Perf and Culture averages update in real time
- **Outcome display** — Rockstar / WIP / PIP / Exit with description
- **Manager notes** — mandatory text field (minimum 20 characters, blocks save if empty)
- **Duplicate detection** — warns if the employee was already reviewed this period
- **Edit existing records** — click a reviewed employee to load and update their record

### Saved Records
- **Filter by** period, department (admin only), outcome, name search
- **Outcome summary pills** — counts of Rockstar / WIP / PIP / Exit at top
- **Trajectory arrows** — `↑ from WIP`, `↓ from Rockstar`, `→ same` vs previous period
- **Export CSV** — scoped to the logged-in user's team

### UX & Micro-interactions
- **Rating button pop animation** — buttons bounce on selection
- **Score card glow** — Perf/Culture card pulses in its colour when rated
- **Outcome badge animation** — slides in each time the outcome changes
- **Toast notifications** — replaces all browser `alert()` dialogs:
  - ✓ Green on save/update
  - ⚠ Amber for warnings
  - ✕ Red for validation errors
- **Notes character counter** — live feedback: `47 characters ✓` / `8 more needed`
- **Review progress bar** — `5 / 9 reviewed this month` with animated fill
- **Full team badge** — `✅ Full team reviewed this month!` when complete

### Employee History Badges
Badges appear on employee cards based on their review history:

| Badge | Trigger |
|---|---|
| 🏆 Consistent Rockstar | Rockstar 3 months in a row |
| 📈 On the Rise | Improved outcome vs previous month |
| ⚠ Needs Attention | PIP or Exit two months running |
| 🔄 Turnaround | Was PIP/Exit, now Rockstar |

---

## Google Apps Script (Backend)

The Apps Script receives POST requests from the HTML and writes rows to the Google Sheet.

### Sheet columns (15 fields)
`period` · `employee` · `role` · `department` · `perfQ1` · `perfQ2` · `perfQ3` · `cultureQ1` · `cultureQ2` · `avgPerf` · `avgCulture` · `outcome` · `notes` · `dateReviewed` · `submittedBy`

### Full script

```javascript
const SPREADSHEET_ID = '1EWdBpT0MhXIVibwsYKLb-5pbL01twTQxpFFNoZeseGI';

const HEADERS = [
  'period','employee','role','department',
  'perfQ1','perfQ2','perfQ3',
  'cultureQ1','cultureQ2',
  'avgPerf','avgCulture',
  'outcome','notes','dateReviewed','submittedBy'
];

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
    const sheet = ss.getSheets()[0];

    if (sheet.getLastRow() === 0) {
      const hr = sheet.getRange(1, 1, 1, HEADERS.length);
      hr.setValues([HEADERS]);
      hr.setFontWeight('bold');
      hr.setBackground('#1a73e8');
      hr.setFontColor('#ffffff');
    }

    const row = HEADERS.map(key => {
      const val = data[key];
      return (val === undefined || val === null) ? '' : val;
    });

    sheet.appendRow(row);

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### Redeployment (when code changes)
1. Apps Script editor → **Deploy** → **Manage deployments**
2. Find the existing deployment → click **pencil icon**
3. Version dropdown → **New version**
4. Click **Deploy** — URL stays the same

---

## Landing Page

`landing.html` is a separate marketing page built with TranZact branding:
- Dark theme (`#0e0f0c`) with orange accent (`#da5d37`) and purple (`#df7afe`)
- TranZact logo and images from `framerusercontent.com`
- Hero tagline, stats, feature cards, photo strip
- 6 leadership/people quotes (Zig Ziglar, Sheryl Sandberg, Andy Grove, Simon Sinek, Ken Blanchard, Robin Sharma)
- Personal dashboard access cards — one per leader, each linking to their `#slug` URL

---

## Deployment

Hosted on **GitHub Pages** from the `main` branch.

### To update the live site after any code change:

```bash
# Copy updated file(s) into the repo folder
cp ~/Desktop/1on1-review.html /tmp/1on1-review/index.html
cp ~/Desktop/landing.html /tmp/1on1-review/landing.html

# Commit and push
cd /tmp/1on1-review
git add .
git commit -m "describe your change"
git push
```

Changes go live within ~60 seconds.

### First-time setup (already done)
```bash
brew install gh
gh auth login
gh repo create 1on1-review --public --source=. --remote=origin --push
gh api repos/Harikeshtiwari12/1on1-review/pages -X POST \
  --field 'source[branch]=main' --field 'source[path]=/'
```

---

## Local Files

| File | Location |
|---|---|
| App | `~/Desktop/1on1-review.html` |
| Landing page | `~/Desktop/landing.html` |
| Git repo (working copy) | `/tmp/1on1-review/` |

---

## Adding a New Leader

1. Add them to the `users` array in `index.html`:
```javascript
{name:'New Leader', title:'Department Name', deptIndex:N, slug:'slug'}
```
2. Make sure `deptIndex` matches their department's position in the `depts` array (0-based)
3. Add a card to `landing.html` with their link
4. Push to GitHub

## Adding a New Department

1. Add an entry to the `depts` array in `index.html` with `name`, `leader`, `members[]`, and `qs[]`
2. Add the leader to the `users` array with the correct `deptIndex`
3. Add role-specific questions to `roleQuestionsMap` for each new role
4. Push to GitHub

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML / CSS / JavaScript (no frameworks) |
| Storage | `localStorage` (browser) + Google Sheets (cloud) |
| Backend | Google Apps Script (serverless) |
| Hosting | GitHub Pages |
| Auth | URL hash-based session (`#slug`) |
| Analytics | None |

---

*Built for 10x Growth · TranZact · April 2026*
