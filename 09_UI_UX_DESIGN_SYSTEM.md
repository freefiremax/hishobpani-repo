# 09 — UI/UX Design System

**Product:** HishobPanti · **Version:** 1.0 · **Visual direction:** restrained, dimmed fintech UI

## 1. Design intent

Use the user's supplied phone reference as the visual direction: a realistic mobile screen, darkened blue-led interface, simple hierarchy, compact financial widgets and understated gloss. The user specifically rejected an exaggerated “AI-generated” look, excessive neon, over-decorated gradients and collage-style presentations.

The UI should feel premium and calm, not futuristic or flashy. Financial amounts, balances, notes and statuses must remain legible.

## 2. Visual principles

- Deep navy/near-black background; avoid pure black everywhere.
- One primary blue accent for focus and actions.
- Muted violet/teal may be secondary accents, not competing neon glows.
- Cards use subtle tonal separation, low-contrast borders and restrained elevation.
- Avoid glowing outlines, excessive glass blur, shiny 3D effects and ornamental gradients.
- Use consistent spacing, grid and corner radii.
- One screen per design frame; no 4×4 collage output when producing page UI.
- UI sheets should be portrait 9:16, with a single full-screen mobile page and no device hand/photographic mockup unless explicitly requested.
- Use realistic, concise product copy. Avoid unreadable tiny text and fake bank/card content.

## 3. Palette (starting tokens)

| Token | Suggested value | Use |
|---|---|---|
| `bg` | `#080D16` | App background |
| `surface` | `#101A2A` | Main cards |
| `surfaceElevated` | `#152238` | Raised cards/sheets |
| `surfaceMuted` | `#0D1522` | Input background |
| `border` | `#26364D` | Subtle borders |
| `textPrimary` | `#F3F6FC` | Primary text |
| `textSecondary` | `#A7B3C7` | Supporting text |
| `textMuted` | `#75839A` | Tertiary text |
| `primary` | `#3978E8` | Primary action/focus |
| `primarySoft` | `#1E3F78` | Selected surfaces |
| `success` | `#42B883` | Positive/received/synced |
| `warning` | `#E5B45B` | Pending/offline |
| `danger` | `#E36D76` | Rejected/negative |
| `violet` | `#8874C8` | Secondary illustration/accent |

These are initial design tokens, not final brand color approvals. Check contrast for text and interactive states.

## 4. Typography

- Use a clean sans-serif such as Inter, Geist or the platform system font.
- Page title: 24–28 px, medium/semibold.
- Section title: 16–18 px, semibold.
- Body: 14–16 px.
- Supporting labels: 12–13 px; avoid going smaller for essential financial information.
- Amounts: tabular numerals, clear rupee symbol and consistent decimal policy.
- Keep weight and hierarchy restrained; do not make every label bold.

## 5. Layout and components

- Mobile content uses safe areas and scrollable content above a fixed bottom navigation where appropriate.
- Main page horizontal padding: 16–20 px.
- Card radius: 16–20 px; input radius: 12–14 px.
- Minimum tap target: approximately 44 px square.
- Maintain clear separation between sections.
- Avoid too many cards on one screen; prioritize the task.

Core components:

- `AppHeader`: title, back action, notification/profile action.
- `BalanceSummaryCard`: positive, negative and net positions.
- `GroupCard`: name, group type, total spend, member count.
- `BudgetProgress`: spent, limit, remaining and status.
- `ExpenseRow`: icon/category, title, group/date, amount.
- `AmountInput`: locale-aware currency display; stores paise.
- `MemberAvatarStack`: selected participants.
- `SplitMethodSelector`: equal/exact/percentage/shares.
- `SyncStatusChip`: local/syncing/synced/failed/conflict.
- `SettlementStatus`: suggested/pending/completed/rejected.
- `InviteCodeCard`: code, copy and share.
- `EmptyState`: direct explanation and useful action.
- `OfflineBanner`: persistent but non-alarming state indicator.

## 6. Navigation

Recommended mobile bottom navigation:

1. Home
2. Groups
3. Central Add action
4. Insights
5. Profile

Group detail uses contextual tabs or a segmented navigation: Overview, Expenses, Balances, Budget. Settlement can be opened from Balances. Do not overload the bottom bar with all domain areas.

## 7. Page inventory and content hierarchy

| Page | Main purpose | Primary action |
|---|---|---|
| Splash | Brand and session loading | Continue |
| Sign up | Create email/password account | Create account |
| Login | Authenticate | Log in |
| Verify email | Confirm address | Resend/continue |
| Forgot password | Reset credentials | Send reset link |
| Home | Quick overview | Add expense / create group |
| Groups list | Find groups | Create/join |
| Create group | Start group | Create group |
| Invite code | Share access | Copy/share code |
| Join group | Redeem invite | Join |
| Group overview | See spend and key stats | Add expense |
| Expenses | Browse records | Add expense |
| Add expense | Capture spend | Review split |
| Split editor | Allocate amount | Save expense |
| Expense detail | Explain a record | Edit/void |
| Members | See/manage participants | Invite |
| Balances | Understand net positions | Simplify |
| Smart settlement | See proposed transfers | Create payment claim |
| Settlement detail | Track payment | Mark paid/confirm/reject |
| Budget | Monitor plan vs spend | Edit budget |
| Insights | Explore patterns | Filter period |
| Notifications | Review updates | Open relevant record |
| Profile | Manage identity | Edit profile |
| Settings | Preferences/security | Save setting |
| Offline/sync center | Inspect pending operations | Retry/resolve |

## 8. Dashboard content rules

Home should prioritize:

1. Greeting and profile/notification controls.
2. A compact balance summary (you owe / you are owed / net position; avoid ambiguous “total balance”).
3. Quick actions: Create Group, Add Expense, Join Group.
4. Active group list.
5. Relevant budget progress.
6. Recent expense/activity list.
7. Bottom navigation.

Do not display a combined personal net balance without explaining whether it is “you are owed minus you owe” across all groups. Each group balance remains independently inspectable.

## 9. Page design sheet requirements

When generating design references, generate **one page per image** at 9:16. Maintain the same base frame, status bar, typography, palette and component language. Do not combine multiple phone screens into a single poster. The UI must be readable and practical, not a glossy AI concept.

Suggested image generation prompt preamble:

> Single 9:16 mobile application UI screen for HishobPanti, restrained dark fintech design, deep navy background, subtle blue accents, calm realistic typography, clean spacing, understated cards, minimal gradients, no neon glow, no 3D, no collage, no hand holding phone, no extra phone mockups. Show one complete usable page with accurate, readable interface hierarchy.

## 10. Accessibility and interaction states

- Use adequate contrast for primary text, inputs and status chips.
- Do not convey payment state by color alone; include text labels/icons.
- Provide loading, empty, error, offline, syncing, conflict and success states.
- Confirm destructive actions such as void expense, revoke invite and archive group.
- Make disabled actions explain why (e.g. “Connect to confirm payment”).
- Provide accessible labels for icons and form controls.

## 11. Responsive behavior

- Mobile: single-column, bottom navigation, full-screen forms/sheets.
- Tablet: constrained content width and adaptive two-column dashboard where useful.
- Web desktop: left navigation/sidebar, centered content and wider data tables; preserve the same visual tokens.
- Avoid stretching mobile cards across extremely wide screens.

## 12. Design acceptance criteria

- All core screens share a consistent palette and spacing system.
- Text and balances are readable at normal phone scale.
- Home dashboard communicates owed/owing clearly.
- Offline and payment statuses are explicit.
- Each generated design image contains one page only in 9:16.
- The design is subdued and attractive, not over-glossy or neon-heavy.
