# cash-tracker

Live cash tracker for TS Laundry bank accounts — cash in bank, issued / released cheques, collections and deposits — on Firebase Firestore (project `database-55b53`).

## Install (about 5 minutes)

1. **GitHub** — in `ixshicpa/cash-tracker`, replace `index.html` with the new one (Add file → Upload files → Commit). This README can replace the old one too. No `config.js` is needed: the Firebase connection is inside `index.html`.
2. **Firestore rules** — Firebase console → Firestore → **Rules**. Paste the lines from `firestore-rules-cash.txt` inside the existing `match /databases/{database}/documents { … }` block, next to the payroll rules, then **Publish**. (If your rules already allow everything, you can skip this.)
3. Wait ~1 minute, open https://ixshicpa.github.io/cash-tracker/ and press **Ctrl+F5**.
4. **First run**: the page says *Set up the Cash Tracker*. Create the administrator (username suggested: `administrator@cash.prime`) and a password of 8+ characters.
5. **Bank accounts** tab → add your 3 accounts with the bank balance **at the start of** a chosen date (e.g. Oct 1, 2026). Add more accounts any time.
6. **Users** tab → add users (Administrator / Encoder / Viewer, and which bank accounts they can see). The old record from the previous tracker is listed at the bottom — delete it.

Try it without touching Firebase: open `…/cash-tracker/?demo` (data stays in that browser only).

## How the balances work

| Entry | While open | When dated |
|---|---|---|
| Cheque issued | listed under **Unreleased cheques** | enter the **release date** → deducted from cash in bank |
| Collection | listed under **Undeposited** | enter the **deposit date** → added to cash in bank |
| Bank debit / credit | — | posted on its date (charges, online payments, interest) |
| Transfer | — | debit on one account, credit on the other, same date |

- **Cash in bank** = opening balance + deposited collections + credits − released cheques − debits
- **Projected** = cash in bank − unreleased cheques + undeposited collections
- Release / deposit dates are entered in the two tabs (one by one or tick several), not by editing each entry.
- Every save updates the totals in the same Firestore transaction, so all users see the same numbers instantly. *Bank accounts → Check and fix balances* re-adds everything from scratch if a total ever looks off.
- Mistakes: **Void** keeps the record but removes it from balances (Restore undoes it). **Delete** (admin only) removes it.
- Cheques unreleased for over 180 days are flagged *stale*. Duplicate cheque numbers on the same account trigger a warning.

## Passwords

- Passwords are stored only as a salted PBKDF2 hash in `cash_users` (same method as the payroll app), so it works on the free Spark plan with no Cloud Functions.
- **Admin reset**: Users → *Reset password* → a new temporary password is shown to give to the user. Their old password stops working at once and they are signed out everywhere; at next sign-in they choose their own.
- Users change their own password from the name menu (top right). 5 wrong tries pause sign-in for 5 minutes; 30 minutes idle signs out.

## Data (Firestore, root collections)

`cash_users`, `cash_banks`, `cash_txns`, `cash_audit` (activity log), `cash_settings`. The tracker never reads or writes the payroll collections.

## Security note

Like the payroll app, this tracker does not use Firebase Authentication, so the sign-in screen protects the app, not the database itself: someone with technical skill who has the project's web config could read the cash_* data directly. To harden it later on the free plan, turn on **App Check** (reCAPTCHA) for Firestore so only your GitHub Pages site can reach the database.
