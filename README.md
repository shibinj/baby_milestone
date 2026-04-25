# 👶 Baby Milestone Tracker

A mobile-friendly web app for tracking preterm baby milestones, adjusted for gestational age. Built for research data collection with secure cloud sync via Firebase.

## Features

- **Gestational age adjustment** — milestone dates (6, 9, 12 months) are automatically corrected based on birth gestational age
- **Cloud sync** — data stored in Firebase Firestore, accessible from any device
- **Google Sign-In** — secure authentication, restricted to authorised users only
- **Role-based access** — researchers enter their own records; admins see all records across the team
- **Mobile-first UI** — optimised for phone use in the field
- **Overview dashboard** — total records, oldest and newest entries at a glance
- **Milestone activity panel** — shows upcoming (next 7 days) and recent (past 7 days) milestones on login
- **Export to CSV** — for direct use in Excel / SPSS (admins get an extra "Entered By" column)
- **Export / Import JSON** — for offline backup and data migration

## Access Control

Access is controlled by the `allowedUsers` collection in Firestore. Only users listed there can sign in, and each user has a **role** that determines what they can see.

### Roles

| Role | Can see | Can edit/delete |
|---|---|---|
| `entry` | Their own records only | Their own records |
| `admin` | All records from all users (tagged with "Entered by") | Their own records only |

### Managing users

To add, remove, or promote a user:

1. Go to **Firebase Console → Firestore Database**
2. Open the `allowedUsers` collection
3. **To add**: create a document with the person's Gmail address as the document ID. Add a field `role` (string) with value `"entry"` or `"admin"`
4. **To remove**: delete their document
5. **To promote/demote**: edit the `role` field on their existing document

> A user with no `role` field defaults to `"entry"`.

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

Firebase config values are never stored in the repository. They are injected at deploy time by GitHub Actions using repository secrets, and the local-config script tag is stripped from the deployed HTML.

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
- [ ] `allowedUsers` collection created with at least one `admin` user document

## Security Model

- **Authentication**: Google OAuth via Firebase Auth. Popup-based sign-in with redirect fallback.
- **Authorization**: Enforced server-side by Firestore rules. The client-side allowlist check is advisory; the database rejects writes from non-allowlisted users regardless of client state.
- **Attribution integrity**: Firestore rules require every write to set `ownerEmail` to the authenticated user's own email, so researchers cannot forge "Entered By" attribution seen by admins.
- **XSS protection**: All user-supplied values (baby names, dates) are HTML-escaped before being rendered.
- **Import validation**: Imported JSON backups are type-checked and validated before being written to Firestore (date format, name length, required fields).
- **Secrets**: Firebase web config is injected at build time via GitHub Secrets. The `firebase-config.local.js` file used for local dev is gitignored.

## Firestore Data Structure

```
users/
  {uid}/
    ownerEmail: string              // the researcher's email (for admin attribution)
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
    role: "admin" | "entry"
    name: string                    // optional, for reference
```

## Tech Stack

- Plain HTML / CSS / JavaScript — no build tool required
- Firebase (Firestore + Google Auth) — free tier
- GitHub Actions — automated deployment
- GitHub Pages — free static hosting
