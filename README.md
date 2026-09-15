# Bank Balance Tracker

A browser-based personal bank account balance tracker branded as FinFlow. It uses plain HTML, Tailwind CSS, Firebase Authentication, Cloud Firestore, and Chart.js.

## Features

- Sign in or create an account with email and password.
- Sign in with Google OAuth.
- Try an interactive demo without logging in; demo changes are kept in memory only.
- Save one balance entry per date, with an optional note.
- Create multiple independently tracked accounts with custom names.
- Delete unused accounts and their balance history.
- Automatically migrate existing single-account balances into a `Main Account` on first sign-in after the multi-account update.
- Import and export account history as CSV files.
- Edit or delete existing entries.
- View dashboard metrics for the latest, highest, and lowest balances, percentage change, and record count.
- View entries in a searchable history table with a confirmation dialog for deletion.
- View balance changes over time in a responsive line chart.
- Switch between light and dark themes.
- Keep each authenticated user's balances isolated under their Firebase user ID.

## Requirements

- A Firebase project.
- Email/password and Google sign-in enabled in **Authentication > Sign-in method**.
- Cloud Firestore created in the Firebase console.
- A local web server. The page uses JavaScript modules and should not be opened directly with `file://`.

## Firebase setup

1. In the Firebase console, create or select a project.
2. Register a web app and copy its Firebase configuration object into `index.html` if using a different Firebase project.
3. Enable the Email/Password and Google providers under Firebase Authentication.
4. Add the local development URL, such as `http://localhost:8000`, to Firebase Authentication's authorized domains if required.
5. Create a Firestore database.
6. Add the rules from [`firestore.rules`](firestore.rules), or paste them into the Firestore Rules tab. The rules temporarily support both the legacy balance path and the new account paths so existing users can migrate safely.

	 Deploy with the Firebase CLI if it is configured for this project:

	 ```bash
	 firebase deploy --only firestore:rules
	 ```
	Then open <http://localhost:8000> in a browser.

The current page is configured for the `bank-tracker-8bf36` Firebase project. Do not use permissive Firestore rules in production. Firebase web configuration values identify the project, but authorization is enforced by Authentication and Firestore rules.

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

For users who have migrated, balances are stored at:

```text
users/{uid}/accounts/{accountId}/balances/{YYYY-MM-DD}
```

Account documents are stored at `users/{uid}/accounts/{accountId}` and contain a user-facing `name`. Account names are unique per user and can be custom values such as `patrimotial1234`. Transfers between accounts are not supported; each account has an independent history.

Each document contains:

```text
balance: number
note: string
createdAt: timestamp
updatedAt: timestamp
```

The date is used as the document ID, so saving another balance for the same date updates that day's entry.

The demo mode uses sample entries in browser memory and does not read from or write to Firestore. On the first authenticated load, the app creates a `Main Account`, copies legacy balances into it in batches, and leaves the original collection in place as a temporary backup.

## Implementation notes

- Tailwind CSS, Inter, and Chart.js are loaded from CDNs; Firebase SDK modules are loaded from Google CDN version `10.14.0`.
- The history query orders records by `createdAt` descending; metrics and the chart sort entries by their date document ID.
- The chart uses USD formatting and the form accepts decimal balance values.
- CSV imports use the exported `Date, Balance USD, Note` format; matching dates are updated in the selected account.
- Notes are HTML-escaped before being displayed, but edit/delete controls still use inline event handlers. A future hardening pass could attach listeners programmatically and avoid interpolating values into handler attributes.
- Firebase error messages are displayed directly in the authentication and data error UI. For production use, consider mapping technical errors to user-friendly messages and validating balance ranges explicitly.
