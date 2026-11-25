# 02 — User Workflows

**Product:** HishobPanti · **Version:** 1.0

The server is authoritative for membership, synchronized financial records, and settlement completion.

## 1. Sign up / sign in

```mermaid
flowchart TD
  A[Open HishobPanti] --> B{Has account?}
  B -- No --> C[Sign up: name, email, password]
  C --> D[Send verification email]
  D --> E[Verify email]
  E --> F[Initialize local store]
  B -- Yes --> G[Sign in]
  G --> H[Validate server session]
  F --> I[Load or sync user data]
  H --> I
  I --> J[Home dashboard]
```

Authentication requires internet. Previously authenticated users may view cached data offline subject to session-expiry policy. Never store plaintext passwords.

## 2. Create group and invite members

```mermaid
flowchart TD
  A[Home / Groups] --> B[Create group]
  B --> C[Enter name and type]
  C --> D[Optional description, dates, budget]
  D --> E[Submit to API]
  E --> F[Server creates group and owner membership]
  F --> G[Server creates random invite code]
  G --> H[Show code / copy / share]
  H --> I[Friend opens Join Group]
  I --> J[Enter code]
  J --> K[Server validates code and user]
  K --> L[Create membership]
  L --> M[Sync group data to new member]
```

Group creation and joining require internet. Store only a secure hash of invite code. Codes are credentials, not public group IDs. Apply expiry, revocation, use limits, and rate limiting.

## 3. Add expense online

```mermaid
flowchart TD
  A[Open group] --> B[Tap Add Expense]
  B --> C[Enter amount and title]
  C --> D[Add optional note]
  D --> E[Select payer and participants]
  E --> F[Choose split method]
  F --> G[Preview shares]
  G --> H{Totals valid?}
  H -- No --> I[Show validation error]
  I --> F
  H -- Yes --> J[Save locally and enqueue]
  J --> K[Send idempotent create request]
  K --> L{Server accepted?}
  L -- Yes --> M[Mark synced]
  L -- No --> N[Keep queued; show error]
  M --> O[Refresh balances and budget]
```

Fields: title, amount, payer, participants, split method, category, date/time, optional note, optional receipt (P1). Payer can also be a participant. Split must total the expense exactly.

## 4. Add expense offline

```mermaid
flowchart TD
  A[Open previously synced group] --> B[No network]
  B --> C[Show Offline status]
  C --> D[Enter expense and note]
  D --> E[Validate with cached group/member data]
  E --> F[Save expense + splits in local transaction]
  F --> G[Write outbox operation]
  G --> H[Show Saved locally / Waiting to sync]
  H --> I[Continue using app offline]
  I --> J[Network returns]
  J --> K[Sync queued operation]
  K --> L{Server accepts?}
  L -- Yes --> M[Persist to D1 and acknowledge]
  M --> N[Mark Synced]
  L -- No --> O[Keep local record; show reason]
```

Constraints: only previously synced groups and members are available offline. Local balances are provisional. A local expense is not visible to other members until synchronized. Membership may have changed while offline; server validation decides whether the operation can be accepted.

## 5. Smart settlement

```mermaid
flowchart TD
  A[Open Balances] --> B[Request latest balances]
  B --> C[Server computes net positions]
  C --> D[Separate debtors and creditors]
  D --> E[Run deterministic matching]
  E --> F[Show suggested transfers]
  F --> G[User selects transfer]
  G --> H[Create payment claim]
  H --> I[Sender pays outside app]
  I --> J[Sender marks as paid]
  J --> K[Status: pending_confirmation]
  K --> L[Notify receiver]
  L --> M{Receiver response}
  M -- Confirm --> N[Status: completed]
  N --> O[Recalculate balances]
  M -- Reject --> P[Status: rejected]
  P --> Q[Debt remains outstanding]
```

A suggestion is not a payment. Only the intended receiver may confirm or reject. Sender cannot confirm on behalf of receiver.

## 6. Sender marks paid while offline

```mermaid
flowchart TD
  A[Sender taps Mark as paid offline] --> B[Queue claim locally]
  B --> C[Show Payment claim waiting to sync]
  C --> D[Network returns]
  D --> E[Submit claim with idempotency key]
  E --> F[Server validates sender and group]
  F --> G[Set pending_confirmation]
  G --> H[Notify receiver]
  H --> I[Receiver confirms online]
```

Do not show “Paid” or “Completed” while action is local-only.

## 7. Receiver confirms or rejects

Receiver sees sender, amount, currency, group, claim date, optional payment method/reference and two actions: “I received it” or “I did not receive it”. Confirmation conditionally changes server state from `pending_confirmation` to `completed`; rejection changes it to `rejected`. Duplicate responses are idempotent. Rejected payments remain outstanding.

## 8. Trip budget

```mermaid
flowchart TD
  A[Create or edit group] --> B[Set total budget]
  B --> C[Save]
  C --> D[Members add expenses]
  D --> E[Active expense total changes]
  E --> F[Recalculate spent and remaining]
  F --> G{Budget crossed?}
  G -- No --> H[Show progress]
  G -- Yes --> I[Show warning]
```

Budget usage is active expense totals, not settlement payments.

## 9. Edit/conflict workflow

- Each mutable expense has a server version integer.
- Client sends the version it last read.
- If current, server accepts and increments version.
- If stale, return a conflict and preserve the local edit.
- Show latest server version and local draft for review.
- Do not silently overwrite newer financial records.
- Voiding is a recorded state change, not hard deletion.

## 10. Offline/cloud backup workflow

Local storage preserves pending work on that device. Once synced, D1 becomes authoritative. Scheduled D1 exports go to a private R2 backup bucket. Recovery must be tested in a non-production environment. App deletion/device loss may remove unsynchronized local entries.

## 11. User-facing sync/payment labels

| Situation | Label |
|---|---|
| Local expense saved | Saved locally · Waiting to sync |
| Upload in progress | Syncing |
| Server acknowledged | Synced |
| Sync error | Sync failed · Retry |
| Offline payment claim | Payment claim waiting to sync |
| Claim received by server | Awaiting receiver confirmation |
| Receiver confirms | Payment confirmed |
| Receiver rejects | Payment not received |
| Suggestion only | Suggested transfer · Not paid |
