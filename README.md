# 👶 Baby Milestone Tracker

A mobile-friendly web app for tracking preterm baby milestones, adjusted for gestational age. Built for research data collection with secure cloud sync via Firebase.

## Features

- **Gestational age adjustment** — milestone dates (6, 9, 12 months) are automatically corrected based on birth gestational age
- **Cloud sync** — data stored in Firebase Firestore, accessible from any device
- **Google Sign-In** — secure authentication, restricted to authorised users only
- **Mobile-first UI** — optimised for phone use in the field
- **Milestone activity panel** — shows upcoming (next 7 days) and recent (past 7 days) milestones on login
- **Export to CSV** — for direct use in Excel / SPSS
- **Export / Import JSON** — for offline backup and data migration

## Access Control

Only users added to the `allowedUsers` collection in Firestore can sign in. To add or remove a user:

1. Go to **Firebase Console → Firestore Database**
2. Open the `allowedUsers` collection
3. Add a document with the person's Gmail address as the document ID
4. To remove access, delete their document

## Local Development

**Requirements:** Python 3 (pre-installed on most systems)

1. Clone the repo
2. Copy your Firebase config into `firebase-config.local.js` (see template below)
3. Serve locally:

```bash
python3 -m http.server 5500
```

4. Open `http://localhost:5500` in your browser

### `firebase-config.local.js` template

This file is gitignored and must never be committed.

```js
const FIREBASE_LOCAL_CONFIG = {
    apiKey:            "YOUR_API_KEY",
    authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
    projectId:         "YOUR_PROJECT_ID",
    storageBucket:     "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId:             "YOUR_APP_ID"
};
```

## Deployment

The app deploys automatically to GitHub Pages when a commit is pushed to `main`.

Firebase config values are never stored in the repository. They are injected at deploy time by GitHub Actions using repository secrets.

### Required GitHub Secrets

Set these in **GitHub → Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `FIREBASE_API_KEY` | Firebase API key |
| `FIREBASE_AUTH_DOMAIN` | e.g. `your-project.firebaseapp.com` |
| `FIREBASE_PROJECT_ID` | Firebase project ID |
| `FIREBASE_STORAGE_BUCKET` | e.g. `your-project.appspot.com` |
| `FIREBASE_MESSAGING_SENDER_ID` | Messaging sender ID |
| `FIREBASE_APP_ID` | Firebase app ID |

### Firebase Setup Checklist

- [ ] Firebase project created
- [ ] Google Sign-In enabled (Authentication → Sign-in method → Google)
- [ ] Firestore database created in production mode
- [ ] Firestore security rules published (see `firestore.rules`)
- [ ] GitHub Pages domain added to Firebase authorised domains
- [ ] `allowedUsers` collection created with at least one user document

## Firestore Data Structure

```
users/
  {uid}/
    babies: [
      {
        name: string,
        birthDateStr: "DD/MM/YYYY",
        gestationalWeeks: number,
        gestationalDays: number,
        milestones: [
          { months: 6,  date: ISO string },
          { months: 9,  date: ISO string },
          { months: 12, date: ISO string }
        ]
      }
    ]

allowedUsers/
  {email}/
    name: string
```

## Tech Stack

- Plain HTML / CSS / JavaScript — no build tool required
- Firebase (Firestore + Google Auth) — free tier
- GitHub Actions — automated deployment
- GitHub Pages — free static hosting
