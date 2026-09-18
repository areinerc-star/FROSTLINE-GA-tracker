# FROSTLINE — Line Up List — deploy guide

This folder is the whole site: one `index.html`, no build step. Deploy it as-is
to Vercel as a static site. Live, shared data comes from Firebase.

---

## 1. Set up Firebase (5–10 min)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and open your existing project (or create one — free "Spark" plan is enough for this).
2. **Add a web app**: Project settings (gear icon) → General → "Your apps" → Add app → Web (`</>`). Give it any nickname. Firebase shows you a `firebaseConfig` object — copy it.
3. Open `index.html` in this folder, find this block near the top of the `<script type="module">` section, and paste your real values in place of every `REPLACE_ME`:

   ```js
   const firebaseConfig = {
     apiKey: "REPLACE_ME",
     authDomain: "REPLACE_ME.firebaseapp.com",
     projectId: "REPLACE_ME",
     storageBucket: "REPLACE_ME.appspot.com",
     messagingSenderId: "REPLACE_ME",
     appId: "REPLACE_ME"
   };
   ```

4. **Turn on Firestore**: left sidebar → Build → Firestore Database → Create database → start in **production mode** → pick a region close to you (e.g. `asia-southeast1` for the Philippines).
5. **Turn on Storage**: left sidebar → Build → Storage → Get started → same region → production mode.
6. **Set security rules** (production mode blocks everything by default — this tool needs open read/write for your team to use it without a login step):

   Firestore rules (Firestore → Rules tab):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /orders/{orderId} {
         allow read, write: if true;
       }
     }
   }
   ```

   Storage rules (Storage → Rules tab):
   ```
   rules_version = '2';
   service firebase.storage {
     match /b/{bucket}/o {
       match /orders/{allPaths=**} {
         allow read, write: if true;
       }
     }
   }
   ```

   **Important**: `if true` means anyone with the link can read and write the
   board — fine for an internal tool you're not sharing publicly, but it's
   not real access control. See "Suggestions" below for locking this down
   once you're past testing.

---

## 2. Deploy to Vercel

Pick whichever you're more comfortable with:

**Option A — Vercel CLI (fastest)**
```bash
npm i -g vercel
cd frostline-vercel-deploy
vercel deploy --prod
```
Follow the prompts (link to your Vercel account, accept defaults — it's a static site, no framework, no build command).

**Option B — Vercel dashboard, no CLI**
1. [vercel.com/new](https://vercel.com/new) → "Add New" → Project.
2. If this folder isn't in a Git repo yet, use the "Deploy" drag-and-drop area and drop the whole `frostline-vercel-deploy` folder in.
3. Framework preset: **Other**. Build command: leave blank. Output directory: leave blank (root).
4. Deploy.

**Option C — Git-connected (best for ongoing edits)**
1. Push this folder to a new GitHub repo.
2. [vercel.com/new](https://vercel.com/new) → import that repo.
3. Same settings as Option B. Every push to `main` auto-redeploys.

Once it's live, share the Vercel URL with your team — everyone who opens it sees the same board in real time.

---

## Suggestions

- **Lock down access before this goes beyond your team.** The `if true` rules above are open to anyone with the URL. Two reasonable next steps, in order of effort:
  - Cheapest: put the Vercel deployment behind Vercel's built-in **password protection** (Project → Settings → Deployment Protection — Pro plan feature).
  - More correct long-term: add **Firebase Authentication** (Google sign-in restricted to your company email domain is a common pattern) and change the rules above to `allow read, write: if request.auth != null;`. I can build this in if you want it.
- **Custom domain**: Vercel → Project → Settings → Domains, point something like `tracker.frostline.yourdomain.com` at it instead of the default `.vercel.app` URL.
- **Backups**: Firestore data isn't automatically backed up on the free plan. If this becomes the real source of truth, consider a scheduled export (Firebase console → Firestore → Import/Export) or upgrading to Blaze plan for automated backups.
- **Storage costs**: image/PDF uploads count against Firebase Storage's free tier (5GB). Fine for a while; worth checking usage after a few months of real logos and reference files.
- **Confirm the real status list.** I built the six statuses (New Client, Proposing Design, For Mock Up, Request for Revise, Approved, In Production) from what was visible in your production tracker's screenshot — the dropdown there was cut off by a scrollbar, so if the real list is different, send it over and I'll update `index.html` to match exactly.
