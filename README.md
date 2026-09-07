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

A live-synced, editable web app for the 3 owners to build out their Entrepreneurial Operating System (EOS) framework together — same real-time, no-login pattern as the checklist above, backed by the same Firebase project in a separate collection.

Covers the full EOS suite:

- **V/TO** — Core Values, Core Focus (purpose/niche), 10-Year Target, Marketing Strategy, 3-Year Picture, and 1-Year Plan, with live counts of on-track/off-track Rocks and open issues.
- **Rocks** — quarterly priorities, one owner and due date each, filterable by quarter, status pill (On Track / Off Track / Done).
- **Scorecard** — weekly measurables table with owner, goal, and an editable cell per week; add/remove measurable rows and week columns.
- **Issues List** — Identify/Discuss/Solve: add issues, assign an owner, mark solved (collapses into a "Solved" section), reopen or delete.
- **L10 Meeting** — the standard 7-segment Level 10 agenda as a live checklist, Customer/Employee headlines, a shared to-do list, and free-form IDS notes. "Reset for next meeting" clears the agenda/headlines/notes but keeps to-dos.

## Owner identity

On first visit, each of the 3 owners picks their name from a "Who's viewing?" prompt — stored only in that browser's `localStorage`, so it's not itself synced. It's used to default new Rocks/to-dos/issues to the current viewer and to show "Viewing as ___" in the header. The 3 owner names themselves are editable from the V/TO tab and *are* synced, so renaming a slot updates everywhere it's referenced. Use **Switch** in the header to change identity on a shared device.

## Firestore setup

Talks to the same Firebase project (`handyman-launch-checklist`) as the checklist, at document path `eos/main`. Add a second scoped rule alongside the existing one:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /checklist/{doc} {
      allow read, write: if true;
    }
    match /eos/{doc} {
      allow read, write: if true;
    }
  }
}
```

Same no-login tradeoff as the checklist app: anyone with the link can view and edit everything.

## Deploy

Same static-site setup as the checklist — no build step. Once GitHub Pages is serving this branch's root, the app is live at `<pages-url>/eos.html`.
