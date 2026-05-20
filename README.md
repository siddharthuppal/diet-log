# Diet Log

A personal mobile-first PWA for tracking daily meals, macros, habits, and steps — with Firebase-backed cloud sync so data persists across all devices.

## What it does

**Today tab**
- Tracks 6 configurable meals (breakfast, lunch, dinner + 3 optional workout-day items)
- Tap a meal card to mark it eaten; calories and protein update in real time via animated rings and progress bars
- Separate checklist of 7 daily habits (water glasses, multivitamin, fiber, meal prep, dishes)
- Step counter with slider + manual input; goal: 8,000 steps/day
- Live summary card showing calories, protein, meals logged, and steps against targets
- Targets automatically adjust between rest days (1,633 cal / 133g protein) and workout days (1,828 cal / 164g protein)

**Monthly tab**
- Calendar heatmap: green = all meals logged, amber = partial, red = missed
- Current streak counter
- Monthly totals: perfect / partial / missed days

**Diet tab**
- Full meal plan with ingredients, prep instructions, and macro breakdown for each meal
- Daily totals for rest days and workout days

**Grocery tab**
- Weekly Trader Joe's shopping list organized by category (proteins, grains, vegetables, fruits, pantry)
- Amazon-tagged items that aren't available at TJ's

**Data & sync**
- All progress (meals checked, habits, workout toggle, steps) is saved to Firebase Firestore under the authenticated user's account
- Offline-capable: Firestore's local persistence keeps the app functional without a network; changes sync automatically when connectivity returns
- Connection status pill (green "Connected" / amber "Offline") shown below the date

## Tech stack

- Single-file HTML/CSS/JS — no build step, no dependencies to install
- Firebase Authentication (Google sign-in)
- Firebase Firestore (per-user, per-day document storage)
- Installable PWA via `manifest.json` (add to home screen on iOS/Android)

## Firebase setup

The app requires a Firebase project. Do this once:

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a project
2. Add a **Web App** and copy the config
3. Paste the 6 values into `index.html`, replacing the placeholders in the `firebase.initializeApp({...})` call:

```js
firebase.initializeApp({
  apiKey:            "...",
  authDomain:        "....firebaseapp.com",
  projectId:         "...",
  storageBucket:     "....firebasestorage.app",
  messagingSenderId: "...",
  appId:             "..."
});
```

4. **Build → Firestore Database** → Create database → Production mode
5. **Build → Authentication → Sign-in method** → Enable **Google**
6. **Authentication → Settings → Authorized domains** → add your hosting domain (e.g. `yourusername.github.io`)
7. **Firestore → Rules** → publish the following:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid}/days/{doc} {
      allow read, write: if request.auth.uid == uid;
    }
  }
}
```

## Running locally

No install needed — just serve the file over HTTP (opening `index.html` directly as a `file://` URL will block Firebase):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Or with Node:

```bash
npx serve .
```

Add `localhost` to Firebase's **Authorized domains** list so Google sign-in works locally.

## Deploying to GitHub Pages

1. Push the repo to GitHub
2. **Settings → Pages → Source** → Deploy from branch `main`, folder `/` (root)
3. Add `yourusername.github.io` to Firebase Authorized domains
4. Your app will be live at `https://yourusername.github.io/diet-log/`

## Data model

Each day is stored as a single Firestore document:

```
users/{uid}/days/{YYYY-M-D}
  date:      "2026-5-19"
  checked:   { breakfast: true, lunch: false, ... }
  habits:    { water1: true, multivitamin: false, ... }
  isWorkout: false
  steps:     7450
```
