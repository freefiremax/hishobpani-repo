# 08 — Offline Sync and Backup Design

**Product:** HishobPanti · **Version:** 1.0 · **Priority:** P0 architecture

## 1. Offline-first promise

A user can add an expense and an optional note to a previously synchronized group without internet. The record is saved on that device immediately, shown as pending synchronization, and uploaded when connectivity returns. It is not visible to other group members until the server accepts it.

Offline capability is not the same as cloud backup. An unsynchronized record exists only on that device until it reaches the server.

## 2. Supported offline behavior

| Capability | Offline support | Notes |
|---|---|---|
| View cached groups | Yes | May be stale |
| View cached members | Yes | Server revalidates on write |
| View cached expenses/notes | Yes | Last synchronized snapshot |
| Add expense/note | Yes | Previously synced group only |
| Calculate provisional balances | Yes | Clearly label as provisional |
| View cached budget | Yes | Provisional usage includes local queued expenses |
| Edit pending local expense | Yes | Before first server acceptance |
| Edit synchronized expense | Queue versioned mutation if supported | Conflict can occur |
| Create group | Draft only | Server required to create invite and owner membership |
| Join by invite code | No | Online server validation required |
| Mark payment sent | Queue command | Not server-pending until acknowledged |
| Confirm/reject payment | No | Receiver action requires server |
| Upload receipt | Queue metadata/file | File upload waits for connection |

## 3. Local persistence architecture

Use a shared repository abstraction with platform-specific adapters:

- Web: IndexedDB, e.g. Dexie.
- Android/iOS: SQLite through a maintained Capacitor plugin.
- Native session token: secure storage plugin, not ordinary local preferences.

Suggested local stores/tables:

- `cached_groups`
- `cached_group_members`
- `cached_expenses`
- `cached_expense_splits`
- `cached_settlements`
- `cached_budgets`
- `sync_outbox`
- `sync_cursors`
- `local_conflicts`
- `cached_notifications`

Local records include `sync_status` (`local`, `pending`, `syncing`, `synced`, `failed`, `conflict`), `local_updated_at`, `server_updated_at`, and `server_version` where applicable.

## 4. Outbox operation format

```json
{
  "operationId": "uuid",
  "entityType": "expense",
  "entityId": "client-generated-expense-uuid",
  "operationType": "create",
  "groupId": "group-uuid",
  "baseVersion": null,
  "createdAtClient": "2026-11-25T18:00:00Z",
  "payload": {
    "title": "Dinner at Ocean View",
    "note": "Pizza, pasta and cold coffees.",
    "totalPaise": 120000,
    "paidByUserId": "current-user-id",
    "splitMethod": "equal",
    "splits": []
  },
  "attemptCount": 0,
  "nextAttemptAt": null,
  "lastError": null
}
```

Never place session tokens, passwords, invite secrets or signed R2 URLs in the outbox payload.

## 5. Add expense offline flow

1. Confirm local database is initialized.
2. Load cached group and member list.
3. Show explicit offline indicator.
4. Validate amount, note length, payer and participant selection using cached data.
5. Calculate all splits in integer paise.
6. In one local transaction, write expense, split rows and outbox operation.
7. Render it immediately with `Saved locally · Waiting to sync`.
8. Recalculate provisional balances and budget usage.
9. When online, submit outbox operation with idempotency key.
10. Server revalidates user session, membership, payer, participants and split totals.
11. If accepted, save canonical D1 records and return server version.
12. Client marks local record synced and removes/archives the acknowledged operation.

## 6. Synchronization coordinator

Triggers:

- App launch/resume.
- Network connectivity returns.
- User taps Retry.
- Periodic foreground check while app is active (avoid aggressive background polling).

Algorithm:

1. Acquire a local sync lock to prevent multiple workers processing the same outbox concurrently.
2. Order operations by dependency and creation order within a group/entity.
3. Send a bounded batch.
4. Server processes each operation idempotently.
5. Match responses by `operationId`.
6. Acknowledge only accepted/already-applied operations.
7. Keep retryable failures with exponential backoff and jitter.
8. Keep conflicts and permanent rejections for user resolution; do not discard.
9. Pull authorized server changes using a cursor.
10. Merge server records into local cache without overwriting unresolved local drafts.
11. Release the sync lock and update last successful sync time.

## 7. Retry and idempotency

Use client-generated stable operation IDs plus a server idempotency record scoped to authenticated user and route/operation. If a request times out after server commit, retry returns the existing resource/outcome rather than creating another expense. Reusing the same key with a different request hash should be rejected.

Use exponential backoff with jitter for retryable network/5xx conditions. Do not automatically retry validation failures, permission failures, stale invite codes or conflicts without user action.

## 8. Conflict handling

Every synchronized mutable record has a server version. Client sends its base version. If the server version has changed:

- Server does not silently overwrite.
- Local draft remains preserved.
- UI shows server version and local proposed version.
- User can review and choose a safe action, e.g. discard local draft, reapply changes to latest version or create a correction expense.
- Financially material edits should generate audit history.

Offline membership revocation is another conflict case: a user may create an expense offline after their membership was removed. The server rejects sync. Keep the local copy and explain that the group no longer accepts it; provide export/copy details rather than discarding it.

## 9. Provisional balances

When offline, calculate balance estimates using cached active expenses/splits and cached completed settlements, including local pending expenses as a separate provisional layer. UI must distinguish:

- **Confirmed server balance:** derived from synchronized records.
- **Pending local impact:** unsynchronized local expenses/claims.
- **Estimated balance:** confirmed balance plus pending local impact.

Do not send or settle a provisional balance as though it were authoritative. Refresh server balances before creating a payment suggestion/claim whenever online.

## 10. Offline payment claim

If sender taps “Mark as paid” offline:

- Queue a claim operation locally.
- Show `Payment claim waiting to sync`.
- Do not show `Awaiting receiver confirmation` until the server accepts the claim.
- When online, submit claim idempotently.
- Server validates sender, receiver, group and current settlement rules.
- Server changes state to `pending_confirmation` and notifies receiver.
- Receiver confirms/rejects online.

Receiver confirmation is not an offline action because it must be authoritative and protected against replay/duplicate confirmation.

## 11. Cloud backup and restore

### Local data

- Local database protects against brief network loss and app restarts.
- App uninstall, device loss, storage cleanup or device failure can destroy records that have not synchronized.
- Show a reminder when unsynced records exist and offer export where practical.

### Cloud data

- D1 is authoritative after successful synchronization.
- Scheduled exports/backups should be written to a private R2 backup bucket/prefix.
- Separate production data and backup object permissions.
- Encrypt exports before long-term storage if required by the selected backup process and key-management design.
- Set retention, access logging and restoration procedures.
- Test restore in a non-production database.
- Record backup completion/failure and alert an operator on failure.

### Manual export

A future encrypted export can include group, expense, split, settlement and budget data in a documented format. Do not export passwords, auth tokens or invitation code hashes. A restore must validate ownership and avoid merging duplicate IDs into an existing group accidentally.

## 12. Sync UI states

| State | Label | User action |
|---|---|---|
| No pending ops | Synced | None |
| Local write committed | Saved locally · Waiting to sync | Continue offline |
| Upload in progress | Syncing | Avoid duplicate submit |
| Server accepted | Synced | Normal use |
| Network/server transient | Sync failed · Retry | Retry later |
| Version/membership conflict | Needs attention | Review and resolve |
| Payment claim queued | Payment claim waiting to sync | Wait for connection |
| Claim accepted | Awaiting receiver confirmation | Receiver responds |

## 13. Offline acceptance tests

- Create expense in airplane mode; close/reopen app; record remains.
- Add note offline; after sync, note matches exactly.
- Interrupt upload after server commit; retry does not duplicate.
- Multiple queued expenses sync in dependency order.
- A rejected permission change preserves local record.
- A conflict shows both server and local versions.
- Provisional balance is not labelled confirmed.
- Payment claim queued offline is not presented as paid/completed.
- Receiver confirmation changes status once.
- Cloud backup can be restored in test environment.
