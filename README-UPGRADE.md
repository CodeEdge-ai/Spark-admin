# SPARK Admin — Upgrade Guide

Complete step-by-step instructions for deploying the upgraded dashboard with backend proxy, Supabase, PWA, and charts.

---

## What's New

| Feature | Details |
|---------|---------|
| **Backend Proxy** | Anthropic + Google Sheets API calls routed through `/api/` — API keys never in the browser |
| **Supabase** | Contacts and RSVPs stored in PostgreSQL with real-time sync; localStorage still works as fallback |
| **Export / Backup** | One-click JSON export and restore from Settings |
| **Debounced Search** | 300ms debounce on Contacts search |
| **Skeleton Loaders** | Animated skeletons for stats, follow-up, upcoming sessions, contacts table |
| **FullCalendar RSVP** | Interactive calendar view in the RSVP tab (toggle between list and calendar) |
| **Chart.js Reports** | Revenue trend line, session type doughnut, practitioner revenue bars |
| **PWA + Offline** | Service worker, manifest, offline banner, IndexedDB cache |
| **Mobile Bottom Nav** | Always visible on screens < 900px with correct active states |

---

## Option A — Use index.html Locally (No Backend)

1. Open `index.html` in any browser
2. Enter your Anthropic API key in the setup screen
3. Everything works with localStorage + direct API calls (same as before)
4. No Supabase or proxy required — graceful degradation built in

---

## Option B — Full Production Deploy (Recommended)

### Step 1 — Google Sheets Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project (or use existing)
3. Enable **Google Sheets API** (APIs & Services → Enable APIs)
4. Go to **IAM & Admin → Service Accounts → Create Service Account**
5. Name it `spark-admin`, grant no roles, click Done
6. Click the service account → **Keys → Add Key → Create New Key → JSON**
7. Download the JSON file
8. Open your Google Sheet → **Share** → paste the service account email (`spark-admin@your-project.iam.gserviceaccount.com`) → Viewer
9. Note your spreadsheet ID from the URL

### Step 2 — Supabase Setup

1. Go to [supabase.com](https://supabase.com) → New Project
2. Once created, go to **SQL Editor** → paste the contents of `supabase-migration.sql` → Run
3. Go to **Settings → API** → copy `Project URL` and `anon public` key

### Step 3 — Deploy to Vercel

```bash
# Install dependencies for the API routes
cd spark-admin-upgraded
npm init -y
npm install googleapis

# Copy env file and fill in your values
cp .env.example .env.local
# Edit .env.local with your actual keys

# Install Vercel CLI
npm i -g vercel

# Deploy
vercel --prod
```

### Step 4 — Configure the Dashboard

1. Open your deployed URL
2. Complete the setup screen (API key optional if using proxy)
3. Open **Settings** in the top nav
4. Fill in:
   - **Proxy URL**: `https://your-vercel-app.vercel.app` (your deployed URL)
   - **Supabase URL**: from Step 2
   - **Supabase Anon Key**: from Step 2
5. Click Save & Sync
6. Your contacts and RSVPs will now sync to Supabase

### Step 5 — PWA Install

On mobile: tap the browser menu → **Add to Home Screen**
On desktop Chrome: click the install icon in the address bar

---

## Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes (for proxy) | Your Anthropic key |
| `GOOGLE_SHEET_ID` | Yes (for proxy) | Sheet ID from URL |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | Yes (for proxy) | Full service account JSON as one line |
| `SUPABASE_URL` | For Supabase | Your Supabase project URL |
| `SUPABASE_ANON_KEY` | For Supabase | Your Supabase anon public key |
| `ALLOWED_ORIGIN` | Optional | CORS origin, defaults to `*` |

---

## Troubleshooting

**Sync still failing after proxy setup?**
- Make sure the service account email has been shared on your Google Sheet (Viewer access)
- Check Vercel logs: `vercel logs your-app.vercel.app`

**Supabase contacts not saving?**
- Run the migration SQL again — check for errors in the SQL Editor
- Ensure RLS policies were created (the migration creates open `allow_all` policies)
- Check browser console for Supabase error messages

**FullCalendar not loading?**
- Requires internet access to load from CDN on first visit
- Once cached by the service worker it works offline

**Charts not rendering?**
- Requires session data from Google Sheets to be synced
- The charts auto-render when the Reports tab is opened after a successful sync

**Offline mode not working?**
- Make sure you visited the page at least once while online (for SW caching)
- The offline banner appears automatically when network is lost

---

## File Structure

```
spark-admin-upgraded/
├── index.html              ← Main app (all CSS + JS inline)
├── manifest.json           ← PWA manifest
├── sw.js                   ← Service Worker
├── supabase-migration.sql  ← Run once in Supabase SQL Editor
├── .env.example            ← Copy to .env.local, fill in values
├── api/
│   ├── anthropic.js        ← Anthropic proxy (Vercel function)
│   ├── sheets.js           ← Google Sheets proxy (Vercel function)
│   └── vercel.json         ← Vercel routing config
└── README-UPGRADE.md       ← This file
```
