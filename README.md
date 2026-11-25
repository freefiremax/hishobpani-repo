# HishobPanti — Project Documentation

**Documentation version:** 1.0 · **Prepared:** 25 November 2026 · **Status:** Product foundation and implementation blueprint

> This is the working source of truth for planning and implementation. It records agreed decisions and proposed designs; it does not claim that the app or infrastructure has been built.

## Product summary

HishobPanti is an offline-first group expense tracking, bill-splitting, trip-budgeting, and settlement-management application. It supports trips, meals, outings, events, roommates, and other shared spending.

**Product promise:** Track shared spending. Split it fairly. Settle it clearly.

## Confirmed decisions

| Area | Decision |
|---|---|
| Product name | HishobPanti |
| Primary platforms | Android and iOS |
| Development workspace | Google AI Studio Build mode |
| Frontend direction | React + TypeScript, packaged with Capacitor |
| Backend | Cloudflare Workers + Hono |
| Main database | Cloudflare D1 |
| File storage | Cloudflare R2 |
| Authentication | Email and password |
| Group joining | Generated invite code |
| Offline support | Add expenses and notes offline; synchronize later |
| Splitting | Equal, exact/custom amount, percentage, shares |
| Smart settlement | Net-balance simplification into fewer transfers |
| Payment confirmation | Sender marks paid; receiver must confirm before completion |
| Expense notes | Required title and optional description/note |
| MVP currency | INR; money stored as integer paise |
| Visual direction | Restrained dimmed dark fintech UI inspired by supplied reference; subtle blue accents, no excessive neon/gloss |

## Architecture clarification

Google AI Studio is the development workspace, not the production host or database. Current Build mode documentation describes a React web frontend with a Node.js server runtime, GitHub sync/import, source export, and Cloud Run deployment. It also offers a separate native Android path. HishobPanti selects the web-app route because one web codebase can be packaged for both Android and iOS with Capacitor.

The AI Studio Node.js runtime is not automatically compatible with Cloudflare Workers. Review generated server code and implement/port the production API in a Workers-compatible runtime. Do not assume a generated Node backend can be deployed unchanged to Cloudflare.

Path: AI Studio Build mode → GitHub/export → React/TypeScript client → Capacitor Android/iOS → Cloudflare Workers API + D1 + R2.

References: [AI Studio Build mode](https://ai.google.dev/gemini-api/docs/aistudio-build-mode), [Capacitor](https://capacitorjs.com/docs), [Cloudflare Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/).

## Documentation index

| File | Purpose |
|---|---|
| [01_PRODUCT_REQUIREMENTS.md](01_PRODUCT_REQUIREMENTS.md) | PRD, scope, roles, requirements and acceptance criteria |
| [02_USER_WORKFLOWS.md](02_USER_WORKFLOWS.md) | End-to-end user journeys and state transitions |
| [03_SYSTEM_ARCHITECTURE.md](03_SYSTEM_ARCHITECTURE.md) | Stack, components, infrastructure, data paths and security |
| [04_SYSTEM_MAPPING.md](04_SYSTEM_MAPPING.md) | Requirement-to-screen-to-service-to-data mapping and Mermaid maps |
| [05_DATA_MODEL.md](05_DATA_MODEL.md) | D1 entities, relationships and initial SQL schema |
| [06_API_CONTRACTS.md](06_API_CONTRACTS.md) | API routes, request conventions, permissions and errors |
| [07_FINANCIAL_ENGINE.md](07_FINANCIAL_ENGINE.md) | Paise math, splits, balances and smart settlement |
| [08_OFFLINE_SYNC_AND_BACKUP.md](08_OFFLINE_SYNC_AND_BACKUP.md) | Local database, outbox, sync, conflicts and backups |
| [09_UI_UX_DESIGN_SYSTEM.md](09_UI_UX_DESIGN_SYSTEM.md) | Visual direction, tokens, navigation and screens |
| [10_DEVELOPMENT_ROADMAP_AND_QA.md](10_DEVELOPMENT_ROADMAP_AND_QA.md) | Milestones, tests and release gates |
| [11_GOOGLE_AI_STUDIO_BUILD_GUIDE.md](11_GOOGLE_AI_STUDIO_BUILD_GUIDE.md) | AI Studio workflow and sequenced build prompts |

## Visual reference

The supplied image is included at [assets/ui-reference.png](assets/ui-reference.png). Adapt its dark, blue-led, glass-card aesthetic to expense management; do not copy bank-specific content.

![HishobPanti visual reference](assets/ui-reference.png)

## MVP boundaries

The app records and reconciles payments; it does not move money. “Mark as paid” is a sender claim. The receiver must confirm receipt. Only a receiver-confirmed settlement clears the corresponding balance.

Offline-created expenses remain local and pending until the server acknowledges them. They are not visible to other group members before sync. Receiver confirmation is an online server action.

## Open decisions before production

- Email delivery provider and sender domain.
- Final logo, typography, and exact colors.
- Whether browser/PWA is part of public launch or a companion target.
- Invite-code expiry and maximum-use defaults.
- Maximum group size and upload limits.
- Account deletion and group-record retention policy.
- Privacy policy, terms, and app-store account ownership.
- Backup retention and recovery-point objectives.

## Rules for contributors and AI agents

1. Treat these documents as the source of truth; update them when decisions change.
2. Do not replace Cloudflare D1/R2 with Firebase or another unapproved production store.
3. Never expose secrets in frontend bundles or source control.
4. Never use floating-point arithmetic for money.
5. Never clear a balance for a suggestion or sender-only payment claim.
6. Never discard local outbox operations before server acknowledgment.
7. Do not call a local-only expense cloud-backed.
8. Every user-facing control must work or be clearly marked unavailable.
9. Keep financial calculations deterministic and tested.
10. Record unresolved assumptions rather than inventing facts.


## Package contents

The documentation pack contains the product requirements, workflows, architecture, system mapping, data model, API contracts, financial engine, offline sync and backup design, UI/UX system, roadmap/QA plan, and AI Studio build guide. The reference image is included under `assets/`.
