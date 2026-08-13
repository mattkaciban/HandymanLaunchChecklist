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
