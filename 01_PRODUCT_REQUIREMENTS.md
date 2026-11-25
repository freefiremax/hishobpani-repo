# 01 — Product Requirements Document (PRD)

**Product:** HishobPanti · **Version:** 1.0 · **Date:** 25 November 2026 · **Status:** Foundation / MVP specification

## 1. Product vision

HishobPanti helps people manage money they spend together. A user creates a group, invites people with a code, adds shared expenses, divides bills in different ways, manages a trip budget, sees balances, and settles obligations through a two-sided payment confirmation workflow.

The app must remain useful when the device temporarily has no internet. A member can add an expense and an explanatory note offline, then synchronize it when connectivity returns.

## 2. Problem statement

Groups use chat messages, notes, spreadsheets, or memory to track shared spending. This makes it difficult to know who paid, who participated, each person's share, who owes whom, whether a transfer was actually received, how much of a budget remains, and what to do when the internet is unavailable.

HishobPanti consolidates these records into a shared group ledger with transparent calculations and explicit settlement confirmation.

## 3. Goals and outcomes

1. Make recording shared expenses quick.
2. Support equal and non-equal splitting.
3. Show trustworthy balances derived from source records.
4. Reduce unnecessary transfers through debt simplification.
5. Support trip budgets and spending visibility.
6. Preserve local expenses during temporary network outages.
7. Require receiver confirmation before a payment clears a balance.
8. Keep group data private to authorized members.

MVP is successful when a test group can register, create/join a group by invite code, add expenses online/offline, split them, add notes, see correct balances, generate suggestions, confirm payments from both sides, and track a group budget.

## 4. Target users

| Persona | Main use |
|---|---|
| Friends travelling | Fuel, hotels, food, tickets and transport |
| Friends dining | Shared and individual restaurant bills |
| College students | Food, outings, events and purchases |
| Roommates | Rent, groceries and utilities |
| Event organizers | Birthdays, parties and group activities |
| General groups | Any shared spending |

The product is not trip-only. Trip features are one group type among many.

## 5. Scope and priorities

### P0 — MVP must-have

| ID | Requirement |
|---|---|
| AUTH-01 | Email/password signup, login and logout |
| AUTH-02 | Email verification and password reset through a configured provider |
| GROUP-01 | Create group with name, type, description, currency, optional dates and budget |
| GROUP-02 | Generate a secure invite code |
| GROUP-03 | Join group by code while online |
| GROUP-04 | Read previously synchronized group data offline |
| GROUP-05 | Owner, admin and member roles |
| EXP-01 | Add expense with title, amount, payer, date, category, participants and optional note |
| EXP-02 | Equal, exact amount, percentage and shares splitting |
| EXP-03 | Edit/void expense under permission rules while preserving history |
| EXP-04 | Expense history and detail view |
| BAL-01 | Derive balances from expenses, splits and completed settlements |
| SET-01 | Generate optimized settlement suggestions |
| SET-02 | Sender can mark a transfer as paid |
| SET-03 | Intended receiver must confirm receipt |
| SET-04 | Pending/rejected payments do not clear balances |
| BUD-01 | Create and view overall group/trip budget |
| OFF-01 | Add expenses and notes offline for previously synced groups |
| OFF-02 | Persist offline records across app restart |
| OFF-03 | Queue and sync operations when online |
| OFF-04 | Prevent duplicates during retries |
| OFF-05 | Show local, syncing, synced and failed states |
| SEC-01 | Server-side authentication, membership and role checks |
| SEC-02 | Private R2 attachment access with time-limited authorization |

### P1 — Post-MVP

Category budgets and alerts, receipt uploads, notifications/push, PDF/CSV export, advanced filters, spending charts, expense comments, manual encrypted export/restore, and web/PWA launch.

### P2 — Later opportunities

Bill OCR/item splitting, UPI integrations, multi-currency, recurring household expenses, and advanced audit/export functions.

## 6. Out of scope for MVP

- Wallet, custody of funds or initiating bank transfers.
- Claiming to verify UPI/bank payments automatically.
- Financial advice, lending or credit scoring.
- AI deciding who owes what.
- Offline invite-code generation or membership changes.
- Guaranteed cloud backup before a local record syncs.

## 7. Functional requirements

### Authentication

- Email/password registration and login.
- Email verification and password reset.
- Secure server-managed sessions.
- Profile with display name and optional avatar.
- Passwords are never stored in app tables or client-side storage as plaintext.

### Groups and invitations

- Types: Trip, Dining, Food, College, Household, Event, Other.
- One base currency per group in MVP; default INR.
- Owner/admin can invite using a generated code.
- Codes are random, rate-limited, expirable/revocable, and stored as hashes.
- Joining requires online server validation.
- Offline access only applies to previously synchronized groups/data.

### Expenses

An expense contains ID, group, title, optional explanatory note, amount in paise, payer, category, date/time, split method, participant shares, creator, timestamps, active/voided status, version and optional attachment references.

The payer can also be a participant. Participants need not include every member. Each expense split must equal the total exactly.

### Split methods

1. **Equal:** divide total and allocate leftover paise deterministically.
2. **Exact/custom:** user enters member amounts; total must match expense total.
3. **Percentage:** percentages stored as basis points; sum must equal 10,000 (100%).
4. **Shares:** proportional allocation using documented integer weights.

Show a split preview before save.

### Balances

Balances are derived, not manually editable. Positive means a member should receive; negative means the member owes. Total member net balance for a group must equal zero.

### Smart settlement

Use a deterministic greedy debtor-creditor matching algorithm. It creates a recommendation, not a payment. Store algorithm version and balance snapshot. Do not claim a globally minimum number of transfers for every graph.

### Payment confirmation

1. Sender marks a transfer as paid.
2. Server status becomes `pending_confirmation` (or remains locally queued until sync).
3. Receiver is notified.
4. Receiver confirms or rejects.
5. Only receiver confirmation changes status to `completed`.
6. Only completed settlements affect balances.
7. Rejected claims remain visible and debt stays outstanding.

### Budgets

Overall group budget is P0. Trip dates are optional. Budget usage sums active expenses only; repayments are not new spending. Category budgets can be P1.

### Offline

Previously synced groups/members/expenses/budgets are readable offline. New expenses/notes are locally persisted and queued. Local balances are provisional while pending changes exist. Failed sync never silently deletes an entry. Payment claims may queue; receiver confirmation must occur online. Creating/joining groups requires server validation.

## 8. Roles and permissions

| Action | Owner | Admin | Member |
|---|---:|---:|---:|
| View group | Yes | Yes | Yes |
| Add own expense | Yes | Yes | Yes |
| Edit/void own expense | Yes | Yes | Yes |
| Edit other expenses | Configurable | Configurable | No by default |
| Invite people | Yes | Yes | No by default |
| Edit group/budget | Yes | Configurable | No |
| Manage roles | Yes | Limited | No |

The server enforces permissions. Hiding a UI button is not security.

## 9. Non-functional requirements

**Security:** HTTPS, server-side session checks, group authorization on every protected request, private R2, short-lived signed URLs, rate limits, audit history, secure secrets, local native encryption where supported, and data minimization. Do not claim browser IndexedDB is automatically encrypted by the app.

**Reliability:** idempotent writes, batched/atomic related writes, explicit sync status, deterministic calculations, and tested backup restoration.

**Usability:** quick entry, readable amounts, accessible contrast, large touch targets, obvious offline status, and clear distinction between suggested and completed payment.

**Performance:** cached group views open without network, local save does not wait for server, paginate long history, index common queries, and monitor D1 latency/overload.

## 10. Privacy-aware analytics

Possible events: `signup_completed`, `group_created`, `invite_joined`, `expense_created`, `expense_synced`, `settlement_suggested`, `payment_claimed`, `payment_confirmed`, `budget_created`, `sync_failed`.

Never send raw expense notes, receipt contents, email addresses, or detailed financial values to analytics by default.

## 11. Acceptance criteria

- Another authenticated user can join a group using its invite code.
- Expense note survives app restart and sync.
- Offline expense survives restart and failed sync.
- Repeated sync does not duplicate an expense.
- All split methods are exact to paise.
- Group net balances sum to zero.
- Suggested transfer does not change balances.
- Sender marking paid does not clear balance.
- Receiver confirmation clears the obligation exactly once.
- Receiver rejection leaves the debt outstanding.
- Budget counts expenses, not settlements.
- Non-members cannot access group records or its R2 objects.

## 12. Open decisions

Email provider, exact invite defaults, final logo/color/type system, web/PWA launch scope, group-size and upload limits, account deletion/retention policy, and backup retention objectives remain to be finalized.
