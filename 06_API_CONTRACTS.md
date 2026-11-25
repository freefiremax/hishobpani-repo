# 06 — API Contracts

**Product:** HishobPanti · **Version:** 1.0 · **Base path:** `/api/v1`

These routes are proposed contracts. Keep handlers thin and put business rules in services. The authenticated user identity comes from the verified session, never from a trusted `userId` supplied by the client.

## 1. Conventions

- JSON request and response bodies unless a signed upload is used.
- All protected routes require a valid Better Auth session.
- Every group route checks active membership and role server-side.
- IDs are opaque strings (UUIDs recommended).
- Money values are integer paise.
- Timestamps are UTC.
- Mutation requests use `Idempotency-Key` where retries could create duplicate records.
- Versioned updates send `If-Match` or `expectedVersion`.
- Paginated lists use cursor pagination.

Standard success:

```json
{
  "data": {},
  "meta": { "requestId": "req_..." }
}
```

Standard error:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The split total must equal the expense amount.",
    "details": { "field": "splits" }
  },
  "meta": { "requestId": "req_..." }
}
```

## 2. Authentication

Mount Better Auth's supported handler endpoints under the chosen auth path. Do not create competing custom password/session endpoints unless the configured Better Auth integration requires an adapter route.

| Operation | Auth | Notes |
|---|---|---|
| Sign up | Public | Email, password, display name; rate-limited |
| Sign in | Public | Email/password; rate-limited |
| Sign out | Session | Revoke current session |
| Email verification | Token flow | Provider delivery configured |
| Password reset | Token flow | Never reveal whether an email exists unnecessarily |
| Get current user | Session | `GET /api/v1/me` |

## 3. Profiles

| Method | Route | Purpose |
|---|---|---|
| GET | `/me` | Current profile and account summary |
| PATCH | `/me` | Update display name/avatar metadata |
| DELETE | `/me` | Initiate account deletion policy flow (not enabled until policy finalized) |

## 4. Groups and invitations

| Method | Route | Purpose |
|---|---|---|
| GET | `/groups` | List groups for current user |
| POST | `/groups` | Create group and owner membership |
| GET | `/groups/:groupId` | Group details; membership required |
| PATCH | `/groups/:groupId` | Update allowed group fields |
| POST | `/groups/:groupId/archive` | Archive group; owner/admin policy |
| GET | `/groups/:groupId/members` | Member list |
| PATCH | `/groups/:groupId/members/:memberId` | Change role/status where authorized |
| POST | `/groups/:groupId/invites` | Generate invite code |
| GET | `/groups/:groupId/invites` | List active invites; owner/admin |
| POST | `/invites/redeem` | Join using code; online only |
| POST | `/groups/:groupId/invites/:inviteId/revoke` | Revoke invite |

Create group request:

```json
{
  "name": "Goa Trip",
  "type": "trip",
  "description": "Weekend trip",
  "currencyCode": "INR",
  "startDate": "2026-12-12",
  "endDate": "2026-12-15",
  "budgetTotalPaise": 2500000
}
```

Invite redemption request:

```json
{ "code": "USER_ENTERED_CODE" }
```

The code must be checked on the server; return only safe group information after successful redemption. Never expose `code_hash`.

## 5. Expenses and splits

| Method | Route | Purpose |
|---|---|---|
| GET | `/groups/:groupId/expenses` | Paginated history |
| POST | `/groups/:groupId/expenses` | Create expense and splits atomically |
| GET | `/expenses/:expenseId` | Expense detail |
| PATCH | `/expenses/:expenseId` | Versioned edit |
| POST | `/expenses/:expenseId/void` | Void expense, preserve history |
| GET | `/expenses/:expenseId/activity` | Authorized change history |

Create expense request:

```json
{
  "id": "client-generated-uuid",
  "title": "Dinner at Ocean View",
  "note": "Pizza, pasta and two cold coffees.",
  "totalPaise": 120000,
  "paidByUserId": "user-id",
  "categoryId": "food-category-id",
  "splitMethod": "equal",
  "occurredAt": "2026-11-25T19:30:00Z",
  "splits": [
    { "userId": "user-a", "amountPaise": 30000 },
    { "userId": "user-b", "amountPaise": 30000 },
    { "userId": "user-c", "amountPaise": 30000 },
    { "userId": "user-d", "amountPaise": 30000 }
  ]
}
```

The backend independently validates split arithmetic, payer/participant membership, field limits, category and group authorization. Ignore any client-supplied balance value.

## 6. Balances and settlements

| Method | Route | Purpose |
|---|---|---|
| GET | `/groups/:groupId/balances` | Current net balances and calculation timestamp |
| POST | `/groups/:groupId/settlement-runs` | Calculate/persist suggestion run |
| GET | `/groups/:groupId/settlement-runs/:runId` | Read suggestion results |
| POST | `/groups/:groupId/settlements` | Sender records transfer claim from a suggestion/manual debt |
| POST | `/settlements/:settlementId/mark-paid` | Sender marks external transfer as sent |
| POST | `/settlements/:settlementId/confirm` | Intended receiver confirms receipt |
| POST | `/settlements/:settlementId/reject` | Intended receiver rejects claim |
| POST | `/settlements/:settlementId/cancel` | Sender cancels before receiver response if policy allows |
| GET | `/groups/:groupId/settlements` | Settlement history |

Balance response concept:

```json
{
  "data": {
    "currencyCode": "INR",
    "calculatedAt": "2026-11-25T15:00:00Z",
    "balances": [
      { "userId": "x", "netPaise": -10000 },
      { "userId": "y", "netPaise": -10000 },
      { "userId": "z", "netPaise": 20000 }
    ]
  }
}
```

The server computes `netPaise`. Do not accept client-provided net balances as inputs to payment finalization.

Receiver confirmation must be a conditional update: update only where status is `pending_confirmation`, `to_user_id` equals current user and version is current. If no row was updated, return a conflict/already-processed result without double-applying the settlement.

## 7. Budgets and insights

| Method | Route | Purpose |
|---|---|---|
| GET | `/groups/:groupId/budget` | Budget, active spend and remaining |
| PUT | `/groups/:groupId/budget` | Update group budget |
| GET | `/groups/:groupId/insights` | Category/member/time summaries |

Budget spending is based on active expenses. Settlements are not expenses.

## 8. Uploads

| Method | Route | Purpose |
|---|---|---|
| POST | `/uploads/prepare` | Validate attachment and issue scoped upload URL |
| POST | `/expenses/:expenseId/attachments` | Commit metadata after upload |
| GET | `/attachments/:attachmentId/download` | Authorize and issue short-lived read URL |
| DELETE | `/attachments/:attachmentId` | Remove authorized attachment metadata/object per policy |

Validate MIME type, size, group membership, expense access and content policy. Use private R2. Avoid accepting arbitrary object keys from clients.

## 9. Offline sync endpoint

| Method | Route | Purpose |
|---|---|---|
| POST | `/sync/operations` | Submit one or more queued idempotent operations |
| GET | `/sync/changes?cursor=...` | Fetch authorized server changes since cursor |

Each operation carries: `operationId`, `entityType`, `entityId`, `operationType`, `baseVersion` where relevant, `createdAtClient`, and a validated payload. Server returns one outcome per operation: `accepted`, `already_applied`, `conflict`, `rejected` or `retryable_error`.

Do not return one whole-batch failure if operations are independently processable. Preserve operation ordering for dependent writes (expense before attachment metadata, for example).

## 10. Error codes

| HTTP | Code | Meaning |
|---:|---|---|
| 400 | VALIDATION_ERROR | Invalid data or arithmetic |
| 401 | UNAUTHENTICATED | Missing/invalid session |
| 403 | FORBIDDEN | Authenticated but not authorized |
| 404 | NOT_FOUND | Resource not found/visible |
| 409 | CONFLICT | Stale version or invalid state transition |
| 410 | INVITE_EXPIRED | Invite expired/revoked/exhausted |
| 413 | FILE_TOO_LARGE | Upload exceeds configured limit |
| 415 | UNSUPPORTED_MEDIA_TYPE | Disallowed upload type |
| 422 | FINANCIAL_INVARIANT_FAILED | Split/balance invariant failed |
| 429 | RATE_LIMITED | Too many attempts |
| 500 | INTERNAL_ERROR | Unexpected server error |
| 503 | RETRYABLE_UNAVAILABLE | Temporary infrastructure failure |

## 11. Authorization matrix

| Resource/action | Requirement |
|---|---|
| Read group | Active group membership |
| Create expense | Active membership; payer and participants are members |
| Edit/void expense | Expense creator/owner/admin policy |
| Create invite | Owner/admin policy |
| Redeem invite | Valid authenticated user and valid code |
| Create payment claim | Sender is group member and claim is supported by outstanding obligation/policy |
| Confirm/reject | Current user is exactly the intended receiver |
| Upload/download receipt | Authorized group member and expense access |
| Read notifications | Notification belongs to current user |

## 12. API implementation checklist

- [ ] Generate OpenAPI or typed route contracts from source schemas.
- [ ] Validate every input at runtime.
- [ ] Centralize session and group membership checks.
- [ ] Use transactional/batched persistence for related financial records.
- [ ] Use idempotency keys for retryable writes.
- [ ] Make settlement transitions conditional and exactly-once in effect.
- [ ] Avoid leaking resource existence to unauthorized users.
- [ ] Add rate limits to login, invite redemption and upload preparation.
