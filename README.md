# Application Tracker

A single-file dashboard that reads your Gmail (read-only), sorts your job emails into
**Applied · Verification · Assessment · Interview · Reminder · Offer · Rejection**,
and groups them by company so each application is one card at its latest stage.
Everything runs in your browser and stays there — nothing is sent to any server except
read-only requests to Gmail itself.

It's one file (`index.html`), so hosting is just committing it to a repo and turning on
GitHub Pages. The only real setup is a one-time Google OAuth client (~15 min), because a
site you host has to authenticate to Gmail on its own.

---

## Part A — Google OAuth client (one-time, ~15 min)

Google renamed this area to the **Google Auth Platform** in 2025–2026, so menu names differ
from most older guides. Do it in this order:

1. **Create a project** at <https://console.cloud.google.com> — top bar → project picker →
   New project. Call it e.g. `job-tracker`.

2. **Enable the Gmail API** — APIs & Services → **Library** → search *Gmail API* → **Enable**.

3. **Configure the consent screen** — APIs & Services → **OAuth consent screen** →
   **Get started**:
   - App name + your support email.
   - **Audience: External.**
   - Developer contact email → Finish.
   - Leave **Publishing status = Testing** (don't publish).
   - Go to **Audience → Test users → Add users** and add *your own Gmail address*.
     Only test users can sign in while the app is in Testing — that's all you need.

4. **Add the Gmail scope** — **Data Access** → *Add or remove scopes* → add
   `https://www.googleapis.com/auth/gmail.readonly` → Update → Save.
   Gmail is a "sensitive" scope; that's fine in Testing mode and needs no verification.

5. **Create the client** — **Clients** → **Create client**:
   - Application type: **Web application**.
   - Name it anything.
   - **Authorized JavaScript origins → Add URI** → your GitHub Pages origin:
     `https://YOURNAME.github.io`
     ⚠️ **Origin = domain only** — no repo path, no trailing slash. Even for a project site
     served at `.../job-tracker/`, the origin you register is just `https://YOURNAME.github.io`.
   - (Optional, for local testing: also add `http://localhost:8000`.)
   - **Create**, then copy the **Client ID** (ends in `.apps.googleusercontent.com`).

You don't need the client *secret* and you don't need a redirect URI — this app uses the
browser token flow, which is matched on origin.

---

## Part B — Deploy to GitHub Pages

1. Create a **public** repo, e.g. `job-tracker`.
2. Add `index.html` at the repo root and commit.
3. **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `root`**
   → Save.
4. Wait ~1 minute. Your site is at `https://YOURNAME.github.io/job-tracker/`.
5. Confirm that `https://YOURNAME.github.io` is listed under Authorized JavaScript origins
   (Part A, step 5).

---

## Part C — Use it

1. Open the site → **⚙ Settings** → paste your **Client ID** → Save.
   *(Or hardcode it once in `index.html`: set `CONFIG.CLIENT_ID` near the top of the script.)*
2. Click **Connect Gmail**, sign in with the Gmail you added as a test user, and grant
   read-only access.
3. Google shows an **"unverified app"** warning (because it's a personal app in Testing mode).
   Click **Advanced → Go to … (unsafe)** to continue. Expected for an app you own.
4. It scans, classifies, and fills the board. Re-runs are incremental — only new mail.

---

## How syncing works

- **Connect Gmail** (first time) authenticates and does the first scan — back your chosen
  window (30–180 days). After that, a **↻ Sync now** button appears for a manual refresh.
- **Auto-sync runs every 5 minutes while the tab is open.** It also catches up when you
  switch back to the tab after being away. Each sync only pulls mail newer than the last one,
  so it's cheap. Turn it off anytime in **⚙ Settings**.
- Nothing runs while the tab is **closed** — a static site has no background process. Leave
  the tab pinned and it stays current on its own; close it and you'll refresh on next open.
- Because Google access tokens are short-lived, auto-sync may ask you to re-approve Google
  roughly once an hour. If a session expires, the button switches to **Reconnect & sync**.

## Living with it

- **Read-only, always.** It requests only `gmail.readonly`. It cannot send, delete, or
  change anything in your inbox.
- **Your data lives in this browser** (`localStorage`, per origin). Another device or a
  cleared cache starts fresh. **Settings → Clear all data** wipes it.
- **You'll re-sign-in roughly hourly.** Access tokens are short-lived and nothing long-lived
  is stored — that's deliberate, and it's why Testing mode is fine.
- **Fix any misread.** Every card has a status dropdown that overrides the guess.
  Use **+ Add manually** for anything you applied to on a portal with no email.
- **Tune the scan** in Settings: change the time window (30–180 days) or edit the Gmail
  search filter.
- The **Need action** band up top surfaces assessments, interviews, reminders, offers, and
  applications with no reply in 10+ days, so nothing slips.

## Troubleshooting

- **`origin_mismatch` / 400 on sign-in** → the origin in Google Cloud doesn't exactly match
  your site. Check `https` (not `http`), and that it's the bare domain with no path/slash.
- **Sign-in works but no emails** → widen the time window, or check the Gmail filter in
  Settings. Try a broader filter first, then narrow.
- **"Access blocked / not a test user"** → add that exact Gmail address under
  Audience → Test users.
