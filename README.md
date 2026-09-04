# Site Sequence — standalone setup

This is a standalone copy of the Site Sequence job scheduler, built so it runs entirely
outside Claude, hosted for free on GitHub Pages. It is the same calculator — same rate
tables, same critical-path scheduling, same quote line items, actual-spend logging, time
on site, attendance notes and risk log — nothing about how it works has changed.

Only one file matters for hosting: **`index.html`**. It contains all the HTML, CSS and
JavaScript in one file, with no build step and no server-side code. `README.md` (this
file) and `firestore.rules.txt` are just documentation — GitHub Pages ignores them.

## What changed under the hood

The original ran inside Claude, which provided a hosted database automatically. A GitHub
Pages site has no backend of its own, so this copy talks directly to a small, free
**Firebase** database (Google's managed Firestore) instead. That's the only structural
change — every calculation, screen, and feature is identical. It's what lets your phone
and laptop both see the same live jobs, the same way the Claude version did.

The first time this page connects to a brand-new, empty database, it automatically loads
Theron's current 17 jobs, all their tasks, the crew roster, and all your rate/calendar
settings — captured at export time — so you don't have to retype anything. This only
happens once, and only if the database is empty; it will never overwrite data you've
since added or changed.

## One-time setup (about 10 minutes)

### 1. Create a free Firebase project

1. Go to **[console.firebase.google.com](https://console.firebase.google.com)** and sign
   in with any Google account.
2. Click **Add project**, give it a name (e.g. `propert-site-sequence`), and finish the
   wizard. You can decline Google Analytics — it isn't needed.

### 2. Turn on the database

1. In the left sidebar, under **Build**, click **Firestore Database** → **Create
   database**.
2. Pick any nearby region, and choose **Start in production mode** (we'll set our own
   access rule in step 4).

### 3. Turn on anonymous sign-in

1. In the left sidebar, under **Build**, click **Authentication** → **Get started**.
2. On the **Sign-in method** tab, click **Anonymous**, toggle it **Enable**, and **Save**.
   This lets the page connect automatically — nobody has to type a password.

### 4. Set the access rule

1. Back in **Firestore Database**, open the **Rules** tab.
2. Replace the contents with the rule below (also saved in `firestore.rules.txt` in this
   folder for reference) and click **Publish**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```

   **What this does and doesn't do:** it blocks anyone who hasn't opened the page at all,
   but anyone who *does* open your GitHub Pages link is signed in automatically and can
   read and write everything — the same "anyone with the link" model as the original
   Claude artifact. It is not a login screen and there's no per-person access control. If
   you'd like real logins later (so only specific people can open it), that's a
   straightforward follow-up — just ask.

### 5. Get your config values

1. Click the gear icon next to **Project Overview** → **Project settings**.
2. Scroll to **Your apps**, click the **`</>`** (web) icon, give the app any nickname
   (e.g. `site-sequence-web`), and click **Register app**. Skip the Firebase Hosting
   offer — you don't need it.
3. You'll see a code block containing a `firebaseConfig` object with six values
   (`apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId`).
   Keep this on screen for the next step.

### 6. Paste the config into index.html

1. Open `index.html` in any text editor.
2. Near the top of the `<script>` block, find:

   ```js
   const firebaseConfig = {
     apiKey: "PASTE_YOUR_API_KEY_HERE",
     authDomain: "PASTE_YOUR_PROJECT_ID.firebaseapp.com",
     projectId: "PASTE_YOUR_PROJECT_ID",
     storageBucket: "PASTE_YOUR_PROJECT_ID.appspot.com",
     messagingSenderId: "PASTE_YOUR_SENDER_ID",
     appId: "PASTE_YOUR_APP_ID"
   };
   ```

3. Replace all six placeholder strings with the real values from step 5, and save.

   Until this is done, opening the page just shows a short "setup needed" notice instead
   of the calculator — that's expected, not a bug.

## Publish it on GitHub Pages

1. Create a new GitHub repository (public is simplest and free; a private repo also
   works if your GitHub plan supports Pages on private repos).
2. Add `index.html` (and, if you like, this `README.md`) to the repository root, and
   commit/push.
3. In the repo, go to **Settings → Pages**. Under **Build and deployment**, set
   **Source** to **Deploy from a branch**, pick your default branch and **/ (root)**,
   then **Save**.
4. GitHub will publish the site at `https://<your-username>.github.io/<repo-name>/`
   within a minute or two — reload the Pages settings screen to get the exact link.

That's it — open that link on your phone and your laptop, and both will show the same
live jobs, exactly like the Claude version did.

## Files in this repository

| File | Needed for hosting? | Purpose |
|---|---|---|
| `index.html` | **Yes — this is the whole site** | The calculator: HTML, CSS and JS in one file |
| `README.md` | No (documentation only) | This guide |
| `firestore.rules.txt` | No (documentation only) | The access rule from step 4, to copy into Firebase's Rules tab |

## Cost

Firebase's free "Spark" tier covers this comfortably — one user, a few dozen documents,
far under the free daily read/write limits. No credit card is required to start.
