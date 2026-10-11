# MailPriority – Gmail Keyword Dashboard

React (JavaScript, plain CSS) dashboard that shows your Gmail inbox and automatically moves
emails that contain your keywords into an **Important** list.

## Features
- Google sign-in (OAuth 2.0), **read-only** Gmail access – no backend, emails stay in your browser
- Dashboard: stats, important emails, keyword category chart, recent emails
- All Mails / Important / Unread / per-category views, with search
- Full email reader (HTML emails shown safely in a sandboxed frame), "Open in Gmail" link
- Keyword highlighting + "why is this important" box in each email
- Keyword manager: add/remove keywords, create categories, reorder priority, choose which go to Important (saved in the browser)
- Auto-checks for new mail every 60 seconds, "Load more emails" for older ones
- Works on mobile

## Default keywords
| Category | Keywords | In Important? |
|---|---|---|
| 🔥 Urgent | urgent, ASAP, immediately, emergency, critical | Yes |
| ⏰ Deadline | deadline, due, today, tomorrow, by Monday, submit, last date | Yes |
| ✅ Action Required | please respond, action required, confirm, approve, review, send, submit | Yes |
| 📅 Meeting | meeting, schedule, appointment, discussion, call | Yes |
| 📰 Normal / Low | FYI, information, update, newsletter, announcement | No (own list) |

### How detection works (accuracy)
- **Whole words, case-insensitive**: `due` matches "Due today" but not "procedure"; `call` doesn't match "recall".
- **Word forms**: `confirm` → confirmed / confirmation, `approve` → approved / approval, `meeting` → meetings,
  `schedule` → scheduled, `submit` → submitted, `urgent` → urgently, `emergency` → emergencies.
- **Negation ignored**: "not urgent", "non-urgent", "no action required", "no need to confirm".
- **Only the new message is checked**: quoted replies ("On … wrote:", `>` lines), forwarded history,
  signatures and newsletter footers (unsubscribe, privacy policy…) are removed first.
- **Scoring**: a keyword in the subject counts 3x more than in the body. Very common words
  (today, tomorrow, due, call, send, review, confirm, approve, submit, schedule, discussion, update, information)
  are *weak*: alone in the body they don't flag an email, but they do when in the subject, when combined
  with another keyword, or in a polite request ("please send", "kindly confirm").
- **Newsletters / promotions** (Promotions/Social/Forums tab, `List-Unsubscribe`): only the subject is checked,
  so marketing text like "act immediately, ends today" can't make them important. They go to Normal / Low.
- Each email shows *which* keywords matched, *where* (subject/body) and a High/Medium/Low confidence.

Run the accuracy tests any time: `npm run test:filter`. Edit defaults in `src/config/keywords.js`
(or live in the app under **Manage keywords**).

---

## Google Cloud setup (one time, ~10 minutes)

### 1. Create a project
1. Open https://console.cloud.google.com/ and sign in with your Gmail account.
2. Top bar → project dropdown → **New Project** → name it `MailPriority` → **Create**.
3. Make sure the new project is selected in the top bar.

### 2. Enable the Gmail API
1. Left menu → **APIs & Services → Library**.
2. Search **Gmail API** → open it → **Enable**.

### 3. Configure the OAuth consent screen
Left menu → **APIs & Services → OAuth consent screen** (newer console: **Google Auth Platform**).
1. Click **Get started** (or **Configure consent screen**).
2. **App information**: App name `MailPriority`, User support email = your email → Next.
3. **Audience**: choose **External** → Next.
4. **Contact information**: your email → Next → agree → **Create**.
5. **Data access** (or "Scopes") → **Add or remove scopes** → search `gmail.readonly` →
   tick `https://www.googleapis.com/auth/gmail.readonly` → **Update** → **Save**.
6. **Audience** (or "Test users") → **Add users** → add your own Gmail address → **Save**.
   ⚠️ Only emails listed as test users can sign in while the app is in "Testing" mode.

### 4. Create the OAuth Client ID
1. **APIs & Services → Credentials** (or Google Auth Platform → **Clients**).
2. **Create credentials → OAuth client ID**.
3. Application type: **Web application**, name: `MailPriority Web`.
4. **Authorized JavaScript origins → Add URI**: `http://localhost:5175`
   (no trailing slash, no path). Leave "Authorized redirect URIs" empty.
5. **Create** → copy the **Client ID** (looks like `1234-abc.apps.googleusercontent.com`).
   You do **not** need the client secret.

### 5. Add the Client ID to the project
In the project folder, copy `.env.example` to `.env` and paste your ID:
```
VITE_GOOGLE_CLIENT_ID=1234-abc.apps.googleusercontent.com
```

## Run it
```bash
npm install
npm run dev
```
Open http://localhost:5175 → **Sign in with Google**.
You'll see *"Google hasn't verified this app"* → click **Continue** (it's your own app) →
allow **"View your email messages and settings"**.

## Troubleshooting
| Problem | Fix |
|---|---|
| `Error 400: redirect_uri_mismatch` / `origin_mismatch` | Origin in Google Cloud must be exactly `http://localhost:5175`. Changes can take ~5 minutes to apply. |
| `Error 403: access_denied` | Add your Gmail as a **Test user** (step 3.6). |
| "Setup needed" message on login | `.env` missing or wrong name – restart `npm run dev` after editing `.env`. |
| "Gmail API has not been used in project…" | Enable the Gmail API (step 2) and wait a minute. |
| `Port 5175 is already in use` | Close the other app, or change the port in `vite.config.js` **and** the origin in Google Cloud. |
| Session expired | Google tokens last 1 hour – just sign in again. |

## Project structure
```
src/
  App.jsx                 state, loading, auto-refresh
  services/gmail.js       Google sign-in + Gmail API calls + email parsing
  utils/filter.js         keyword matching & priority classification
scripts/test-filter.mjs   accuracy tests for the keyword detection
  config/keywords.js      default keyword categories
  components/             Login, Sidebar, Topbar, Dashboard, MailView, EmailRow, EmailDetail, KeywordManager
  styles/app.css          all styling
```
