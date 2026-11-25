# 10 — Development Roadmap and QA Plan

**Product:** HishobPanti · **Version:** 1.0

## 1. Delivery strategy

Build a reliable vertical slice before expanding. Each milestone should include functioning UI, validated business logic, persistence, permissions and tests. Do not accept visual-only buttons for P0 features.

## 2. Milestones

### M0 — Repository and design foundation

- Establish GitHub repository and branch/PR convention.
- Create React + TypeScript + Vite project.
- Set up routing, tokens, app shell and reusable components.
- Add responsive layouts and dark restrained design system.
- Add lint, typecheck and test scripts.

Exit gate: app builds; routes load; UI tokens/components are shared; no placeholder controls claim completion.

### M1 — Authentication and profile

- Integrate Better Auth email/password.
- Configure D1 adapter and email delivery.
- Signup, login, logout, verification and password reset.
- Secure session storage in web/native client.
- Profile creation and retrieval.

Exit gate: protected API rejects unauthenticated calls; password/session lifecycle tests pass.

### M2 — Group lifecycle and invite codes

- D1 migrations for groups, memberships and invites.
- Create group, role model, invite generation, redemption, expiry/revoke.
- Group list and group overview.

Exit gate: user can create a group, share code, another user joins, nonmember is blocked.

### M3 — Expense engine

- Expense form and optional note.
- Four split methods and paise arithmetic.
- Expense detail/history, edit, void and audit records.
- Atomic persistence of expense + splits.

Exit gate: financial unit tests pass; group balances recompute correctly from expenses.

### M4 — Offline local database and synchronization

- IndexedDB web adapter and SQLite native adapter.
- Shared repository interfaces.
- Outbox, idempotency, sync coordinator, retry and conflict UI.
- Cached groups/members and provisional balance labels.

Exit gate: airplane-mode expense creation survives restart and syncs once after reconnect.

### M5 — Balances and smart settlement

- Derived group balances.
- Settlement run/suggestion storage.
- Deterministic debtor-creditor matcher.
- Payment claim, receiver confirmation/rejection, settlement history.

Exit gate: circular-debt test passes; sender claim does not affect balances; receiver confirmation applies exactly once.

### M6 — Trip budget and insights

- Group budget and spend progress.
- Category grouping and simple insights.
- Empty/loading/error/offline states.

Exit gate: voided expenses are excluded; settlements are not budget spend.

### M7 — R2 attachments, notifications and backups

- Private R2 upload/download flow.
- Attachment metadata validation.
- In-app notifications; push provider if selected.
- Scheduled export/backup and restore drill.

Exit gate: nonmember cannot access object; restore succeeds in test environment.

### M8 — Hardening and release

- Security review, accessibility, performance and data migration checks.
- Capacitor Android/iOS integration and device testing.
- App icons, splash, privacy policy, terms and store preparation.
- Crash/operational monitoring.

Exit gate: P0 acceptance suite passes on web, Android and iOS-supported test environments.

## 3. Test layers

| Layer | What to test | Tools (suggested) |
|---|---|---|
| Unit | Money parsing, split math, balances, settlement optimizer | Vitest |
| Component | Inputs, forms, status chips, error states | Testing Library |
| API integration | Auth, membership, expense transactions, settlement transitions | Worker test harness / integration tests |
| Database | Migrations, foreign keys, indexes, atomic writes | D1 local/test database |
| Offline | Local persistence, outbox, retries, conflicts | Playwright + device tests |
| End-to-end | Signup → group → expense → settlement → budget | Playwright / native automation |
| Security | IDOR, access control, invite brute force, R2 access | Manual + automated tests |
| Recovery | D1 export restore, local pending record preservation | Runbook exercise |

## 4. P0 financial test suite

| Test | Expected result |
|---|---|
| ₹1,000 / 4 equal | ₹250 each |
| ₹100 / 3 equal | ₹33.34, ₹33.33, ₹33.33 |
| Exact split off by 1 paise | Rejected |
| Percentage total below/above 100% | Rejected |
| Share-based allocation | Sum equals expense total |
| Payer is participant | Correct net result |
| Payer is not participant | Correct net result |
| Expense voided | Excluded from balances and budget |
| Suggested transfer | No balance change |
| Sender marks paid | Still outstanding |
| Receiver confirms | Outstanding obligation reduced once |
| Receiver rejects | Debt remains outstanding |
| Duplicate confirmation | No double application |
| Circular debt X/Y/Z | X→Z ₹100; Y→Z ₹100 |
| Group balance sum | Always zero |

## 5. Offline test suite

- Add expense while offline.
- Restart app while offline; expense remains.
- Reconnect and synchronize; one server record exists.
- Simulate timeout after server commit; retry creates no duplicate.
- Create multiple pending expenses; dependency order is preserved.
- Test server membership revoked while client offline.
- Test concurrent server edit and local edit; conflict is surfaced.
- Test corrupted/outdated cache recovery without deleting outbox.
- Show pending local impact separately from confirmed balance.
- Queue an offline payment claim; UI does not say “completed”.

## 6. Security test suite

- Unauthenticated user cannot access private API.
- User A cannot read Group B records by changing IDs.
- Nonmember cannot upload or download a receipt.
- Sender cannot confirm their own payment claim.
- Wrong receiver cannot confirm/reject.
- Reused/revoked/expired invite code is rejected.
- Invite attempts are rate-limited.
- R2 objects are not publicly enumerable.
- Secret scanning finds no credentials in client bundle or Git.
- Oversized/unapproved MIME upload is rejected.

## 7. Release gates

A milestone cannot be marked complete until:

- Typecheck and lint pass.
- Required unit/integration tests pass.
- No P0 defect remains.
- Manual acceptance steps are recorded.
- DB migrations are reviewed and tested from an empty database.
- Error and offline states have been checked.
- Documentation is updated with changed decisions.

## 8. Operational metrics

Track: request error rate, API latency, D1 errors/latency, outbox age, sync retry counts, sync permanent failures, settlement transition conflicts, backup completion and restore test date. Avoid sending raw notes, receipt content, email addresses or detailed transaction amounts to analytics by default.

## 9. Release-readiness checklist

- [ ] Terms and privacy policy reviewed.
- [ ] Email domain and verification configured.
- [ ] Account deletion and retention policy finalized.
- [ ] Backup schedule/retention and restore tested.
- [ ] Group and receipt authorization reviewed.
- [ ] Offline data-loss warning communicated.
- [ ] Android and iOS permissions reviewed.
- [ ] App store metadata/screenshots prepared.
- [ ] Support/contact and incident process available.
