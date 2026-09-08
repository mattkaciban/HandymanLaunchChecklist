# Handy's Location Launch Checklist — Live

A live-synced, editable web version of the 90-day New Location Launch Checklist. Anyone who opens the page URL sees and edits the same shared checklist in real time, backed by Firebase Firestore.

## Deploy

This is a single static file (`index.html`) with no build step.

1. In this repo's **Settings → Pages**, set the source to deploy from this branch's root folder.
2. Save. GitHub will give you a URL (usually `https://<username>.github.io/<repo>/`) within a minute or two.
3. Share that URL with anyone who should see/edit the checklist — no login required.

Any other static host (Netlify, Vercel, Cloudflare Pages) works the same way — just point it at `index.html`.

## Firestore setup

The page talks to a Firestore database (project `handyman-launch-checklist`) at the document path `checklist/main`. It reads and writes:

- `checks` — map of task IDs to booleans
- `fields` — Market / Operator / Launch Date / Coordinator
- `kpi` — the Week 4/8/12 "Actual" values

**Security rules** — in the Firebase console under Firestore → Rules, scope access to only this collection (rather than leaving the whole database in test mode):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /checklist/{doc} {
      allow read, write: if true;
    }
  }
}
```

There's no login/auth on this page — anyone with the page URL can view and edit. That's the intended tradeoff for a simple two-person shared checklist; it also means anyone who discovers the Firebase project's client config could read/write that one collection directly. If that's ever a concern, the next step up would be adding Firebase Auth (e.g. email link sign-in) and rules that check `request.auth != null`.

## How syncing works

- Every checkbox, field, and KPI actual writes straight to Firestore as soon as it changes (checkboxes immediately, text fields/KPI values after a short debounce while typing).
- All open tabs/devices listen for changes and update live — no refresh needed.
- The sidebar status dot shows connection state: amber pulsing = connecting, green = live, red = offline/error.
- **Reset checklist** clears the shared state for everyone currently viewing the link, not just your own browser — it asks for confirmation first.

---

# EOS Framework — Live (`eos.html`)

A live-synced web app for the 3 owners to build out their Entrepreneurial Operating System (EOS) framework together, backed by the same Firebase project as the checklist in a separate `eos/main` document. Unlike the checklist, this app **requires Google sign-in** and only allowlisted accounts can open it.

Covers the full EOS suite:

- **V/TO** — Core Values, Core Focus (purpose/niche), 10-Year Target, Marketing Strategy, 3-Year Picture, and 1-Year Plan. A quarter-scoped "At a Glance" card counts on-track / off-track / done Rocks for the current quarter plus open issues. **Print V/TO** produces a clean one-pager (text fields are mirrored into printable text so nothing is clipped).
- **Rocks** — quarterly priorities grouped by owner, each with an editable quarter, owner, due date, and color-coded status (On Track / Off Track / Done). Filter by quarter; adding or re-dating a Rock moves the filter to follow it so it never disappears.
- **Scorecard** — weekly measurables with an owner, a goal, and a direction (`≥` higher is better / `≤` lower is better). Each weekly cell colors itself green or red against the goal; non-numeric entries stay unscored. Add or remove measurable rows and week columns; deleting a row also clears its stored weekly values.
- **Issues List** — Identify, Discuss, Solve. Issues are ranked with ▲/▼ and the top 3 are highlighted, since that's what you actually work in the meeting. Solve records a date; solved issues collapse into their own section and can be reopened.
- **L10 Meeting** — the standard 7-segment Level 10 agenda with a **shared timer** per segment (all three of you see the same clock; segments turn red when over their allotted minutes) and a running meeting total against the 90-minute target. Plus Customer/Employee headlines, a shared to-do list, IDS notes, and a **1–10 meeting rating** per owner with a live average. "Reset for next meeting" clears the agenda, timers, headlines, ratings and notes but keeps to-dos.

The header shows connection state, who last edited and when, and who you're currently acting as.

## Sign-in setup (do this once)

The app uses Google sign-in, and the allowlist lives in the Firestore rules — **not** in the page source, so owner email addresses are never published in this public repo.

1. **Enable Google sign-in**: Firebase console → project `handyman-launch-checklist` → **Authentication** → **Sign-in method** → enable **Google** → Save.
2. **Authorize the GitHub Pages domain**: still under **Authentication** → **Settings** → **Authorized domains** → **Add domain** → `mattkaciban.github.io`. Sign-in fails silently from unauthorized domains, so don't skip this.
3. **Set the rules** (Firestore Database → Rules), replacing the three placeholder addresses with the owners' actual Google account emails:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /checklist/{doc} {
      allow read, write: if true;
    }
    match /eos/{doc} {
      allow read, write: if request.auth != null
        && request.auth.token.email_verified == true
        && request.auth.token.email in [
             'owner-one@example.com',
             'owner-two@example.com',
             'owner-three@example.com'
           ];
    }
  }
}
```

4. Click **Publish**.

To add or remove someone later, edit that email list and publish again — no code change or redeploy needed. Anyone signed in with a non-allowlisted account gets a clear "not on the allowlist" message naming the address they used.

Note the checklist app (`index.html`) is deliberately left open — only the `eos` collection requires auth.

## Owner identity

Separate from *authentication*, the app tracks which of the 3 owner slots you are, so new Rocks, to-dos and issues default to you. On first sign-in you pick your name once; the choice is stored against your Google account in Firestore, so it follows you to every device. **Switch** in the header changes it (useful on a shared laptop). The 3 owner names are editable from the V/TO tab and are synced, so renaming a slot updates everywhere it's referenced.

## How syncing works

Every edit writes to Firestore and all open tabs update live. Two details worth knowing:

- **Typing is never interrupted.** Incoming updates don't rebuild a field while you're typing in it — they're applied when you click away or pause. In-flight keystrokes are re-applied over incoming data, so a slow connection can't roll back what you just typed.
- **Write failures are visible.** The status dot turns red with "Couldn't save" rather than failing silently.

## Deploy

Same static-site setup as the checklist — no build step. Once GitHub Pages is serving this branch's root, the app is live at `<pages-url>/eos.html`.
