# Stay Fit — Deployment Guide

The branded PWA for Stay Fit personal training. Manage members, sessions, payments, and body measurements. Members see their own progress.

**Stack:** GitHub Pages (hosting) + Firebase (database & auth) — both free.

---

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | The entire app |
| `manifest.json` | PWA install metadata |
| `sw.js` | Service worker for offline support |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `favicon-32.png` | App icons |
| `logo-white.png`, `logo-dark.png` | Brand logo variants used in UI |
| `README.md` | This guide |

---

## Step 1: Create a Firebase project (free)

1. Go to **https://console.firebase.google.com**
2. Click **"Add project"** → name it (e.g. "stayfit-app") → continue → disable Google Analytics → **Create project**
3. Click the **"</>"** (Web) icon → register a web app
4. Copy the config block shown:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "stayfit-app.firebaseapp.com",
     projectId: "stayfit-app",
     storageBucket: "stayfit-app.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123:web:abc..."
   };
   ```

5. Enable Firestore:
   - **Build → Firestore Database → Create database**
   - Production mode → region near you (asia-south1 for India) → Enable
   - **Rules tab** → paste:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{collection}/{docId} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
   - Click **Publish**

6. Enable Anonymous Auth:
   - **Build → Authentication → Get started**
   - **Sign-in method** → **Anonymous** → Enable → Save

---

## Step 2: Configure the app

Open `index.html` in a text editor. Near the top, find:

```js
const firebaseConfig = {
  apiKey: "REPLACE_WITH_YOUR_API_KEY",
  ...
};

// ⚠️ TRAINER PIN — change this to whatever you want before deploying.
window.TRAINER_PIN = "1234";
```

Two things to update:
- **Replace the six Firebase values** with the ones from Step 1
- **Change the TRAINER_PIN** to anything you want (4–6 digits recommended). This is the PIN the trainer types to access the trainer view. Members never enter it.

Save the file.

> **Security note:** Both the Firebase API key and the PIN are visible in the source code if someone really looks. The Firebase keys are designed to be public (security is enforced by the Firestore rules). The PIN is a soft barrier — fine for keeping members out of the trainer view, but not a real auth system. For stronger security, we'd need a proper backend.

---

## Step 3: Deploy via GitHub Pages

### 3.1 — Choose your URL strategy

**Option A: Pretty URL — `https://YOUR_USERNAME.github.io`** (recommended)
- Create a repo named **exactly** `YOUR_USERNAME.github.io` (e.g. if your username is `priya`, name the repo `priya.github.io`)
- This special name gives you a URL with **no project segment**
- ⚠ Only one such repo allowed per GitHub account, and the repo name must match your username

**Option B: Standard URL — `https://YOUR_USERNAME.github.io/stayfit-app/`**
- Repo named anything (e.g. `stayfit-app`)
- Works fine but has the repo name in the URL

**Option C: Custom domain — `https://app.stayfit.in`**
- Buy a domain (Namecheap, GoDaddy: ~₹800/year)
- Either Option A or B works as base — then add the custom domain in Settings → Pages → Custom domain
- Add a CNAME DNS record pointing to `YOUR_USERNAME.github.io`
- Enable Enforce HTTPS

### 3.2 — Create the repo

1. Go to **https://github.com/new**
2. Name it per the option above
3. Set **Public** (private requires GitHub Pro)
4. Don't add README/license
5. **Create repository**

### 3.3 — Push the files

Terminal in unzipped folder:

```bash
git init
git add .
git commit -m "Initial Stay Fit app"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/REPO_NAME.git
git push -u origin main
```

Or use GitHub's web upload: open the empty repo → "uploading an existing file" → drag the contents (not the folder) → commit.

### 3.4 — Enable GitHub Pages

1. Repo **Settings** → left sidebar **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / Folder: `/ (root)` → Save
4. Wait 1–2 minutes. URL appears at the top.

---

## Step 4: Install on the phone

### Android (Chrome)
1. Open the URL in Chrome
2. **Three-dot menu → "Install app"** (or "Add to Home Screen")
3. Stay Fit icon on home screen — opens full-screen

### iPhone (Safari only)
1. Open the URL in Safari
2. Share button → **"Add to Home Screen"** → Add
3. Icon on home screen — opens full-screen

---

## Step 5: Trainer workflow

1. Open the app → "I'm the Trainer" → enter your PIN
2. Add members (with phone numbers — these are how members log in)
3. For each member, pick **billing mode**:
   - **Flat Package** — fixed price for a period (₹5000/month, etc.)
   - **Per Session** — charged per completed session (₹500 × sessions done)
4. Schedule sessions, mark them as completed when done
5. Record payments as they come in
6. Track each member's body measurements (Members → tap a member → "Log entry" under Body Measurements)
7. BMI auto-calculates from weight + height

---

## Step 6: Share with members

Send each member:
- The URL
- "Tap **I'm a Member**, enter your phone number"

Members install the same way (Step 4) and see only their own data — sessions, payment status, and body measurement progress.

---

## How billing works

**Flat Package mode**: Member is charged the full `Package Amount` upfront. Due = Package Amount − Payments received. Used for monthly/quarterly/yearly packages.

**Per Session mode**: Member is charged for each completed session at the configured rate. Due = (Completed Sessions × Rate) − Payments received. The "due" amount grows automatically each time a session is marked complete. Best for members who train at varying frequencies (3, 4, 5 sessions/week).

Switch a member between modes anytime by editing them.

---

## Updating the app later

Edit `index.html` → push:

```bash
git add .
git commit -m "Describe change"
git push
```

GitHub Pages redeploys in 1–2 minutes. Users get the new version on next app open.

---

## Free tier limits

- **GitHub Pages**: unlimited bandwidth for public repos
- **Firebase Spark plan**: 50k reads/day, 20k writes/day, 1 GiB storage

A trainer with 50 members generates roughly 1,000 reads/day. Effectively free forever.

---

## Troubleshooting

**"CONNECTING" forever** → Firebase config wrong OR Anonymous Auth not enabled. Check browser console.

**"Incorrect PIN"** → Update `window.TRAINER_PIN` in `index.html` to match what you're entering, push the change.

**Member login "not found"** → Trainer must add that exact phone number first. Last 10 digits match, so format doesn't matter.

**"Permission denied"** → Firestore rules not published.

**404 after enabling Pages** → Wait 2 min for first deploy. Confirm Settings → Pages shows `main` / `/ (root)`.

**Want to reset all data** → Firebase console → Firestore → delete each collection (`members`, `sessions`, `payments`, `measurements`).

---

## What's included

- ✅ Two-role app: Trainer (PIN-protected) and Member (phone-based)
- ✅ Members can't access trainer view — PIN gate blocks them
- ✅ Two billing modes per member: Flat Package or Per Session
- ✅ Session-based dues auto-calculate as sessions are completed
- ✅ Body measurements log: weight, height, auto-calculated BMI
- ✅ Members see their progress (read-only); trainer enters measurements
- ✅ Real-time sync across all devices via Firebase
- ✅ Installable PWA on iPhone, Android, desktop
- ✅ Free hosting + free database

## What's not included (yet)

- **Automated WhatsApp reminders** — current button opens WhatsApp with pre-filled message; auto-sends need WhatsApp Business API (paid)
- **Multiple trainers per deployment** — currently one deployment = one trainer
- **Photos for measurements** — only numeric data right now
