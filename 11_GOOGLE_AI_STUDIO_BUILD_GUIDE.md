# 11 — Google AI Studio Build Guide

**Product:** HishobPanti · **Version:** 1.0

## 1. Purpose

Use Google AI Studio Build mode as the primary AI-assisted coding workspace. The generated source must remain reviewable and exportable to GitHub. AI Studio does not replace production infrastructure. Cloudflare Workers, D1 and R2 remain the intended production backend/storage.

## 2. Important implementation boundary

Choose a React web application as the shared client. Package it for Android and iOS with Capacitor. Do not select a separate Android-only native workflow if the goal is one shared codebase for both platforms.

AI Studio may generate a server runtime appropriate to its own preview/deployment workflow. Review server code carefully. The production API must run on Cloudflare Workers and use Cloudflare bindings. Do not assume Node.js-specific server code will run on Workers unchanged.

## 3. Repository structure target

```text
hishobpanti/
├── apps/
│   ├── web/                 # React + TypeScript + Vite client
│   └── api/                 # Hono Worker API
├── packages/
│   ├── shared/              # Types and Zod schemas
│   ├── finance/              # Pure money/split/balance/settlement logic
│   └── data-access/          # Repository contracts and adapters
├── migrations/              # Reviewed D1 migrations
├── docs/                    # Product documentation pack
├── capacitor.config.ts
├── package.json
└── README.md
```

This is a target structure. If AI Studio creates a different structure, adapt it without duplicating packages or mixing business logic into page components.

## 4. Master project prompt

Paste this first into AI Studio Build mode:

```text
PROJECT: HishobPanti

Act as a senior product engineer and software architect.
Build a mobile-first group expense management application
for Android, iOS and web.

PRODUCT:
HishobPanti lets people create groups using invite codes,
record shared expenses, add explanatory notes, split bills,
track trip budgets, calculate balances, simplify settlements,
and confirm payments with both sender and receiver participation.
It must support offline expense creation for groups that were
previously synchronized.

CLIENT STACK:
- React
- TypeScript
- Vite
- Responsive web client
- Capacitor packaging for Android and iOS

PRODUCTION BACKEND:
- Cloudflare Workers
- Hono
- Cloudflare D1
- Cloudflare R2
- Better Auth email/password
- Drizzle ORM

LOCAL DATA:
- IndexedDB for web
- SQLite through a Capacitor plugin for native apps
- Shared repository abstraction
- Durable outbox and synchronization coordinator

NON-NEGOTIABLE RULES:
1. Do not replace Cloudflare D1/R2 with Firebase.
2. Do not expose server credentials in frontend code.
3. Store money as integer paise only.
4. Calculate balances from expenses, splits and completed settlements.
5. A suggested transfer is not a payment.
6. Sender marking paid is not completion.
7. Intended receiver must confirm before a settlement clears balances.
8. Offline expenses must be persisted locally before attempting sync.
9. Never discard pending outbox operations before server acknowledgement.
10. Never silently overwrite a financial record modified elsewhere.
11. All group endpoints must enforce server-side membership checks.
12. UI controls must be functional or clearly marked unavailable.

DESIGN:
Use a restrained, dimmed dark fintech style inspired by the
provided reference: deep navy, subtle blue accents, clean typography,
quiet cards and simple spacing. Avoid excessive neon, glowing borders,
3D gloss, visual clutter and AI-looking collage screens.

PROCESS:
First inspect and create the app shell, routes, shared types,
design tokens and reusable components. Do not implement all backend
features in one step. Report files created, dependencies, assumptions,
remaining tasks and tests. Preserve a clean exportable codebase.
```

## 5. Sequenced implementation prompts

### Prompt A — Design system and route skeleton

```text
Implement the HishobPanti design system and complete route skeleton.
Use the agreed restrained dark fintech visual direction. Create shared
tokens, typography, buttons, inputs, cards, status chips, navigation,
loading/empty/error states and responsive layouts. Add routes for
Splash, Sign Up, Login, Verify Email, Forgot Password, Home, Groups,
Create Group, Join Group, Group Detail, Expenses, Add Expense,
Split Editor, Expense Detail, Balances, Smart Settlement,
Settlement Detail, Budget, Insights, Notifications, Profile,
Settings and Offline/Sync Center.
Use realistic placeholder fixtures only in a clearly isolated demo
mode. Do not present them as persisted user data. Make every visible
button either navigate or perform its stated local interaction.
```

### Prompt B — Authentication

```text
Implement email/password authentication using Better Auth.
Keep secrets and session signing server-side. Add signup, login,
logout, email verification, forgot password and reset flows.
Add protected routes and session-aware loading states. Do not create
a second password table or store plaintext passwords. Add tests for
unauthenticated API access and session expiry. Keep the UI connected
to the actual auth integration; do not simulate successful login.
```

### Prompt C — Groups and invite codes

```text
Implement group creation, member list, roles, invite-code creation,
invite-code redemption, expiry and revocation. Use Cloudflare D1 via
Workers API. The server generates a high-entropy invite token and
stores only its hash. Validate redemption on the server and rate-limit
attempts. The creator becomes owner. Add membership authorization tests.
Offline creation may save a local draft only; do not invent a valid
invite code offline.
```

### Prompt D — Expense form, notes and split editor

```text
Implement expense creation with title, optional note (maximum 500
characters), amount, payer, date/time, category, participants and
split method. Support equal, exact amount, percentage and shares.
Use integer paise and deterministic remainder allocation. Persist an
expense and all split rows atomically. Payer and participants must be
members of the group. Add preview and validation states. Add edit and
void flows with version checking and activity history.
```

### Prompt E — Financial engine

```text
Create a pure, independently tested TypeScript financial package.
Implement rupee string to paise parsing, four split methods, balance
calculation, deterministic debtor-creditor matching, and settlement
state transition validation. Never use floating-point money math.
Only active expenses and completed settlements affect balances.
Suggestions and pending/rejected claims do not. Assert group balances
sum to zero. Include the circular debt example from the documentation.
```

### Prompt F — Offline database and sync

```text
Implement a shared local repository interface with IndexedDB on web
and SQLite on Android/iOS. Store cached groups, members, expenses,
splits, settlements, budgets, sync cursors and an outbox. Users can add
expenses and notes offline for previously synchronized groups. Save
expense, splits and outbox operation in one local transaction.
Add automatic foreground sync on reconnect/resume, idempotency keys,
retry backoff, operation dependencies, server acknowledgements and
conflict preservation. Clearly label provisional balances and local
records. Never silently discard outbox records.
```

### Prompt G — Settlement and two-sided confirmation

```text
Implement settlement suggestions separately from actual payment
records. Sender may mark a transfer as paid; server changes it to
pending_confirmation and notifies the intended receiver. Only that
receiver can confirm or reject. Receiver confirmation conditionally
changes pending_confirmation to completed exactly once. Rejection
leaves debt outstanding. Sender cannot self-confirm. An offline sender
claim remains queued locally until server acknowledgement. Add tests
for every state transition and duplicate requests.
```

### Prompt H — Budgets, insights and cloud files

```text
Implement group budget, spend/remaining calculations, simple category
insights and private R2 receipt upload/download. Budget spending counts
active expenses only, never settlement transfers. Upload URLs must be
short-lived and issued only after group authorization. Validate size
and MIME type on the server. Add clear offline, upload and retry states.
```

### Prompt I — QA and production hardening

```text
Review the complete application against docs/01 through docs/10.
Identify incomplete flows, fake data, nonfunctional buttons, broken
routes, frontend-only authorization, floating-point financial math,
unsafe secrets, missing outbox safety and settlement state errors.
Add unit, integration and end-to-end tests. Do not claim a feature is
complete until it has real persistence and the required tests.
Produce a gap report and fix P0 issues before visual polish.
```

## 6. AI Studio working rules

- Work one module at a time.
- Ask AI Studio to inspect existing code before modifying it.
- Keep source code synced/exported to GitHub frequently.
- Review dependency additions and generated server code.
- Run typecheck, lint and tests after each milestone.
- Never paste production secrets into prompts or frontend code.
- Keep mock/demo data behind a deliberate development flag.
- Ask for a file/change summary after each task.
- Update `/docs` whenever a product or architecture decision changes.

## 7. GitHub milestone discipline

Suggested branches:

- `main`: stable, reviewed code.
- `feature/design-foundation`
- `feature/auth`
- `feature/groups-invites`
- `feature/expenses-splits`
- `feature/offline-sync`
- `feature/financial-engine`
- `feature/settlements`
- `feature/budgets`
- `feature/cloud-storage`

Use small commits with descriptive messages. Do not allow generated code to overwrite the documentation source of truth.

## 8. Definition of done

A feature is complete only when:

1. UI works on target screen sizes.
2. API/local behavior is implemented, not simulated.
3. Data persists in the correct store.
4. Validation and authorization are enforced.
5. Loading/empty/error/offline/conflict states exist.
6. Unit and integration tests pass.
7. P0 acceptance criteria are updated.
8. Documentation and README are updated.
