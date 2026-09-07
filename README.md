# Bank Balance Tracker

A small browser-based bank account balance tracker built with plain HTML, Firebase Authentication, Cloud Firestore, and Chart.js.

## Features

- Create an account and sign in with email and password.
- Save one balance entry per date, with an optional note.
- Edit or delete existing entries.
- View entries in a newest-first history table.
- View balance changes over time in a line chart.
- Keep each user's balances isolated under their Firebase user ID.

## Requirements

- A Firebase project.
- Email/password sign-in enabled in **Authentication > Sign-in method**.
- Cloud Firestore created in the Firebase console.
- A local web server. The page uses JavaScript modules and should not be opened directly with `file://`.

## Firebase setup

1. In the Firebase console, create or select a project.
2. Register a web app and copy its Firebase configuration object.
3. Open `index.html` and replace the placeholder values in `firebaseConfig` with the values from Firebase.
4. Enable the Email/Password provider under Firebase Authentication.
5. Create a Firestore database.
6. Add Firestore security rules that only allow an authenticated user to access their own balances. For example:

	 ```text
	 rules_version = '2';
	 service cloud.firestore {
		 match /databases/{database}/documents {
			 match /users/{userId}/balances/{balanceId} {
				 allow read, write: if request.auth != null
													 && request.auth.uid == userId;
			 }
		 }
	 }
	 ```

Do not commit production credentials or permissive Firestore rules to a public repository. Firebase web configuration values identify the project, but authorization is enforced by Authentication and Firestore rules.

## Run locally

From the project directory, start any static web server. For example, with Python installed:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000> in a browser.

## Data model

Balances are stored at:

```text
users/{uid}/balances/{YYYY-MM-DD}
```

Each document contains:

```text
balance: number
note: string
createdAt: timestamp
updatedAt: timestamp
```

The date is used as the document ID, so saving another balance for the same date updates that day's entry.

## Implementation notes

- Firebase SDK modules are loaded from Google CDN version `10.14.0`.
- Chart.js is loaded from jsDelivr.
- The history query orders records by `createdAt` descending; the chart separately sorts document IDs chronologically.
- The current page displays Firebase error messages directly in the UI. For production use, consider mapping technical errors to user-friendly messages and validating balance ranges or currency explicitly.
- The history table currently builds action buttons with `innerHTML`; notes should be treated as untrusted input. A production hardening pass should create cells with `textContent` and attach event listeners instead of interpolating note text into inline handlers.
