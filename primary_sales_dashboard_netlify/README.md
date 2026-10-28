# Primary Sales Performance Dashboard — Netlify Deployment

## Historical data is baked in, automatically, every time you open it

Your April 2025 – March 2026 historical data (`historical-seed.json`,
~50,964 rows) loads automatically on every page load — no upload, no click,
nothing to do. It's a separate file shipped alongside `index.html`, not
stored in your browser, so it isn't subject to browser storage limits and
works identically for everyone who opens the dashboard.

### How new data adds up on top of it — nothing is ever lost

- Uploading or Drive-refreshing a **new month** (one not already in the
  historical file) simply adds alongside it — both are included in every
  YTD/MTD calculation and view from then on.
- **Re-uploading a month that's already in the historical file** (e.g. a
  corrected extract for a month you already have) cleanly **replaces just
  that month** — the newer version always wins, and nothing gets duplicated
  or double-counted.
- This was tested directly: re-uploading a 50-row correction for April 2025
  replaced that month's original 4,045 rows exactly, with zero duplicates.

### When you have a new/updated historical file

Since it's a static file baked into this build (for performance — a dataset
this size is too large for browser storage, which silently drops data past
~5MB), updating it means sending me the new file for a rebuild — the same
way we did today. It isn't something to hand-edit.

## KAM, Platform, AOP Division & AOP Portfolio — fully automatic

These four fields are **no longer needed in your daily transaction file at all.**
The dashboard derives them automatically, invisibly, from two master mappings
that are baked directly into this build:

- **KAM & Platform** — derived from each transaction's Customer Code
- **AOP Division & AOP Portfolio** — derived from each transaction's Material Code

There is no upload button for these — nothing to manage day to day. Your
daily file only needs: Customer Code (BILL TO PARTY), Material Code
(MATERIAL), Channel, City, Billing Qty, Qty in CV, Sales Value, SKU, and
Billing Date.

### When the master mapping data changes (~every 3 months)

Since this data is baked into the code rather than uploaded, updating it
means rebuilding this file. When you have an updated master sheet, send it
back for a new build — this isn't something to edit by hand in this file.

### Rows with no AOP match

Some transactions' Material Codes won't be found in the AOP mapping (this is
a data coverage gap in the master sheet itself, not a bug). These rows are
clearly flagged and excluded **only** from the AOP Division/Portfolio views —
they're still fully counted in every other total, platform, KAM, city, and
SKU view.

## ⚠️ Important: data storage is NOT shared across your team on this version

This build uses your browser's `localStorage` to save uploaded report data.

- Storage is **per browser, per device**.
- If you upload a report on your laptop, **only your browser** sees that data.
- A teammate opening the same Netlify link on their own computer will see an
  **empty dashboard** — they are not seeing your uploads.
- Clearing your browser data / cache will also erase the dashboard's history.

This is different from the version running inside Claude.ai, which used
Claude's shared storage so one upload was visible to your whole team.

### If you need genuine "one upload, whole team sees it" on Netlify

You'd need a real backend to hold the data centrally — for example:
- [Supabase](https://supabase.com) (free tier available, Postgres-based)
- [Firebase](https://firebase.google.com) (free tier available, Firestore-based)
- A small custom API of your own

That's a real (if fairly small) build — ask if you want help scoping it.

### Reasonable ways to use this Netlify version today

- **Single shared machine**: e.g. deploy it and always open it from one
  dedicated office computer/browser that the team looks at together.
- **Personal use**: each person deploys/uses their own copy.
- **Manual redistribution**: one person uploads, then exports and shares
  screenshots/numbers manually with the team (not automated).

---

## How to deploy

### Option A — Drag and drop (easiest, no account setup needed beyond Netlify)

1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag this whole folder (containing `index.html` and `xlsx.full.min.js`)
   onto the page
3. Netlify gives you a live URL immediately (e.g. `random-name-123.netlify.app`)
4. Share that URL

### Option B — Netlify CLI

```bash
npm install -g netlify-cli
cd this-folder
netlify deploy --prod
```

### Option C — Connect to a Git repo (for ongoing updates)

1. Push this folder to a GitHub repo
2. In Netlify: **Add new site → Import an existing project** → connect the repo
3. Build command: none needed (it's static) — publish directory: `/`

---

## What's inside

- `index.html` — the full dashboard (self-contained, includes all styling/logic)
- `xlsx.full.min.js` — the Excel-parsing library (SheetJS), bundled locally so
  the site doesn't depend on an external CDN staying available

## OneDrive auto-fetch — what you see vs. what's happening

The dashboard shows **one button**: "↻ Refresh from OneDrive". Your OneDrive
folder link is already built into the code — nobody ever needs to paste it.

Unlike Google Drive, OneDrive genuinely has no "just an API key" option —
Microsoft Graph API always requires a real sign-in. So the flow is: the
**first time** anyone clicks Refresh, a one-time popup asks for two values
(Azure App Client ID and Tenant ID, see setup below) — saved permanently in
that browser afterward. Then **every click, every session, shows a Microsoft
sign-in popup** — this cannot be skipped or automated away; it's how
Microsoft's security model works, not a limitation of this dashboard.

### ⚠️ Real risk worth knowing before you invest time here

This exact organization previously blocked SharePoint/OneDrive access to
Claude at the admin level. Since OneDrive for Business runs on the same
SharePoint infrastructure, a custom Azure app for this dashboard may hit the
identical block — particularly the "admin consent" step below. If it does,
that's a genuine IT policy decision, not a bug to fix in this file.

### 1. Register an app in Azure

1. Go to [portal.azure.com](https://portal.azure.com) → **Azure Active Directory** → **App registrations** → **New registration**
2. Name it anything (e.g. "HRI Ecommerce Dashboard")
3. **Supported account types**: "Accounts in this organizational directory only" (single tenant)
4. **Redirect URI**: platform = **Single-page application (SPA)**, value = your live Netlify URL (e.g. `https://your-site.netlify.app`)
5. Click **Register**

### 2. Note two values from the Overview page

- **Application (client) ID** — a GUID
- **Directory (tenant) ID** — a GUID, or you can use your domain (e.g. `hriindia.com`) instead

### 3. Grant API permission

1. Left menu → **API permissions** → **Add a permission** → **Microsoft Graph** → **Delegated permissions**
2. Search and check **Files.Read** → **Add permissions**
3. If you see a **"Grant admin consent"** button and you're not an admin, this needs your IT/Azure admin to click it — **this is the step most likely to hit the same wall as before**

### 4. Use it in the dashboard

1. Open the dashboard, click **"↻ Refresh from OneDrive"**
2. Paste the **Client ID** when prompted, then the **Tenant ID**
3. A Microsoft sign-in window opens — sign in with your HRI account, approve the permission
4. It finds the most recently modified `.xls`/`.xlsx` file in the folder and loads it

### Notes
- If step 3's admin consent is denied or unavailable, this integration cannot work regardless of any other setting — that's the same organizational restriction encountered before, not something to debug further in code
- Requires being served over `https://` (a real deployed URL) — will not work opened as a local `file://` page
- Manual upload always remains available as a fallback regardless of how this goes



