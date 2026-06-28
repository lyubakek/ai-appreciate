---
project: "Appretiate"
version: 1
status: draft
created: 2026-06-28
context_type: greenfield
product_type: mobile
target_scale:
  users: medium
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: 8
  hard_deadline: null
  after_hours_only: true
---

# Appretiate — Product Requirements

## Vision & Problem Statement

Regular café-goers who use stamp/punch loyalty cards routinely lose rewards they have
already partly earned: the paper card is at home, lost, or was never picked up at the
counter. Progress lives on a single physical card that can't be backed up, synced, or seen
at a glance, so a half-finished card is invisible until the customer happens to be holding
it at the right café. Customers forfeit free coffees they were owed; cafés lose the
repeat-visit pull the card was meant to create.

The insight is that the value is not digitizing any one café's card — it is the wallet plus
the nudge. A single app holding every participating café's card beats any one café's
standalone program, and the "you're one stamp from a free coffee" notification is the
retention engine paper can never provide. Independent cafés can't build their own app; a
shared, near-zero-cost platform removes the barrier that kept them on paper.

## User & Persona

Primary persona: **the regular coffee drinker** — someone who visits one or more cafés
often enough to care about loyalty rewards, carries their phone (not a wallet of paper
cards), and is motivated by progress and free coffee. The moment they reach for Appretiate
is at the counter, paying for a coffee: they open the app so the café can add a stamp, and
they glance at how close they are to a reward.

### Secondary persona

The café (owner/barista) is a secondary actor in the MVP — present only insofar as they
need to add a stamp to a customer's card. Their experience is deliberately minimal in the
first version (see Non-Goals and Access Control).

## Success Criteria

### Primary
- The core loop works end to end for a real customer at a real café: open a card → present
  a code → café scans or enters it → stamp lands and persists → progress visible → reach the
  reward threshold → redeem a free coffee (self-serve, one-time, system-enforced). If this
  loop holds reliably, the product worked.

### Secondary
- A "reward ready" / "one stamp away" notification measurably brings a customer back for a
  return visit — evidence the notification is functioning as the retention engine. (Nice to
  have; not sufficient on its own.)

### Guardrails
- **No self-granted / fake stamps** — a stamp can only originate from an authenticated café
  action; customers cannot fabricate stamps. Failure here is a regression even if the loop
  otherwise works.
- **Earned progress is never lost** — stamps are durable across app reinstalls and device
  changes; a customer never silently loses progress.
- **No double redemption** — a completed reward can be redeemed exactly once; it cannot be
  replayed or claimed twice.

## User Stories

### US-01: Customer earns a stamp at the counter

- **Given** a signed-in customer with an open card for this café
- **When** they present their code (a QR or a one-time code) and the café scans or enters it
- **Then** a stamp is added to that card and the updated progress is immediately visible to
  the customer

#### Acceptance Criteria
- A stamp originates only from an authenticated café action — there is no path for a
  customer to add a stamp to their own card.
- Both the QR and the manual one-time-code path are accepted; the manual code is
  one-time / rotating so it cannot be screenshotted and replayed.
- The customer's card reflects the new stamp count without a manual refresh.
- A replayed or duplicate scan of the same presented credential within a short window does
  not double-count.
- The stamp that reaches the threshold triggers the reward-ready state rather than an extra
  stamp beyond the threshold.

### US-02: Customer redeems a reward

- **Given** a customer whose card has reached its stamp threshold
- **When** they choose to redeem (self-serve) at the counter
- **Then** the reward is marked redeemed exactly once, a glanceable confirmation is shown to
  the café, and the card advances per the program's rules (reset behavior defined in
  Business Logic)

#### Acceptance Criteria
- A completed reward can be redeemed exactly once; a second attempt is rejected by the
  system (the no-double-redemption guardrail holds regardless of the self-serve flow).
- Redemption is customer self-serve for counter speed; it surfaces a confirmation the café
  can glance at. (Residual fraud question — redeeming without a purchase — tracked in Open
  Questions #2.)
- Earned progress and redemption history survive app reinstall / device change.

## Functional Requirements

### Accounts & identity
- FR-001: Customer can create an account and sign in. Priority: must-have
  > Socrates: Counter-argument considered: "a signup wall before any value loses casual
  > users." Resolution: stands — cross-device sync, durable progress, and notifications
  > all require a real account.
- FR-002: Café can authenticate into a café-scoped role shared at the café level (not per-barista accounts). Priority: must-have
  > Socrates: Counter-argument considered: "a separate barista login slows the counter and
  > asks small cafés to manage staff accounts." Resolution: REVISED — authenticate the
  > café device with a café-scoped token rather than individual barista accounts. Keeps the
  > trust boundary (an authenticated café action) without per-staff account overhead.
- FR-003: Customer can recover account access without losing earned stamps. Priority: nice-to-have (deferred)
  > Socrates: Counter-argument considered: "recovery is rare; defer it." Resolution:
  > REVISED — demoted to nice-to-have / deferred from MVP. Tension flagged: this is in
  > direct conflict with the "earned progress never lost" guardrail. See Open Questions #1.

### Cards & wallet (customer)
- FR-004: Customer can find and open a stamp card for a participating café. Priority: must-have
  > Socrates: Counter-argument considered: "in-app café discovery is scope creep; add cards
  > by scanning a café code instead." Resolution: stands — customers need to see and open
  > participating cafés in-app for the wallet to feel like a destination.
- FR-005: Customer can view all their cards and progress at a glance in one place. Priority: must-have
  > Socrates: Counter-argument considered: "early users hold only one card, so the wallet
  > view is premature UI." Resolution: stands — aggregation IS the core thesis; the wallet
  > is the product, not a nice-to-have.
- FR-006: Customer can present a stamp credential — a QR code or a short one-time code — at the counter to receive a stamp. Priority: must-have
  > Socrates: Counter-argument considered: "why customer-shows-QR and not a cheaper static
  > café QR the customer scans?" Resolution: REVISED — support both a QR and a manually
  > readable one-time code, but the credential stays customer-originated (customer presents,
  > café verifies). A static café code could be photographed and reused to self-stamp;
  > customer-presented credentials preserve the no-self-stamp guarantee.

### Stamping & redemption (café)
- FR-007: Café can add one or more stamps (one per qualifying item) by scanning the customer's QR or entering the customer's one-time / rotating code, supplying the item quantity. Priority: must-have
  > Socrates: Counter-argument considered: "manual tap-to-add is faster than aiming a camera
  > at a busy counter." Resolution: REVISED — support both scan and manual entry, but the
  > manual path uses a one-time/rotating customer code (not a static code) so it can't be
  > screenshotted and replayed. Both paths bind the stamp to the specific customer's card.
  > Phase 5 addition: a single interaction grants one stamp per qualifying item (café enters
  > the quantity), not a flat one-per-visit.
- FR-008: Café can reverse or correct a mistaken stamp or redemption, with every reversal audit-logged. Priority: must-have
  > Socrates: Counter-argument considered: "reversal is an abuse vector — a café could claw
  > back legitimately earned stamps." Resolution: REVISED — kept (counter mistakes are
  > common) but every reversal is audit-logged so abuse is visible and disputable.
- FR-009: System awards a reward when a card reaches its stamp threshold. Priority: must-have
  > Socrates: Counter-argument considered: "should the café approve the award rather than
  > auto-grant?" Resolution: stands — threshold counting is deterministic; auto-awarding is
  > correct and removes a needless café step.
- FR-010: Customer can redeem an earned reward self-serve (one-time, system-enforced), producing a redemption confirmation the café can glance at. Priority: must-have
  > Socrates: Counter-argument considered: "café confirmation prevents fraud." Resolution:
  > REVISED — redemption is customer self-serve for counter speed, but the system enforces
  > one-time redemption (preserving the no-double-redemption guardrail) and surfaces a
  > glanceable confirmation. Residual tension: self-serve redemption could be triggered
  > without an actual purchase. See Open Questions #2.

### Café program setup
- FR-011: Café can self-register, and goes live only after operator approval. Priority: must-have
  > Socrates: Counter-argument considered: "operator-only onboarding is a growth bottleneck."
  > Resolution: REVISED — reconciled the Phase 4 operator-only decision with the bottleneck
  > concern: cafés self-register (no bottleneck) but go live only after manual operator
  > approval (fraud control). This supersedes the earlier operator-onboarded-only decision.
- FR-012: Café can configure its program (stamp threshold, reward, and stamp-expiry policy) once onboarded. Priority: must-have
  > Socrates: Counter-argument considered: "operator-set program keeps the café app trivial."
  > Resolution: stands — cafés need to own their own program terms (change the threshold,
  > swap the reward) without operator round-trips.
  > Phase 5 addition: stamp-expiry policy is per-café program config (a café may choose no
  > expiry or a defined inactivity window).

### History & records
- FR-013: Customer can view their per-café history of completed cards (lifetime coffees / rewards earned at each café). Priority: must-have
- FR-014: Café can generate and print a PDF report of completed cards as line items (date and reward value per card) over a selected period, for tax/accounting documentation. Priority: must-have
  > Resolved (shape Phase 7): format is a printable PDF; granularity is per completed-card
  > line items (date + reward value); the report is period-selectable. Still open: which
  > jurisdiction's accounting/tax format the report must satisfy — see Open Questions #3.

### Notifications
- FR-015: Customer is proactively notified — including when not actively using the app — when a reward is ready. Priority: must-have
  > Socrates: Counter-argument considered: "in-app-only is enough; push is heavy infra."
  > Resolution: stands — push is the retention engine named as the core insight; a reward
  > the user only sees on app open defeats the purpose.
- FR-016: Customer gets a "one stamp away" nudge. Priority: must-have
  > Socrates: Counter-argument considered: "push nudges feel like spam and drive uninstalls."
  > Resolution: stands — a well-timed nudge is the core retention lever; the risk is in
  > tuning frequency, not in having the capability.

## Non-Functional Requirements

- A stamp or redemption registers with user-perceived completion in under ~1 second at the
  counter, so the interaction does not hold up the queue.
- A stamp issued while the café's connectivity is poor is never silently lost: the
  interaction either completes or recovers cleanly so the stamp is ultimately recorded
  exactly once (no loss, no duplication).
- A café can observe only the data needed to stamp the customer in front of it; that
  customer's activity at other cafés and their broader profile are not exposed to any café.
- Reward-ready and "one stamp away" notifications reach the customer within minutes of the
  event that triggers them.
- Completed-card history and records are retained for the life of the account
  (indefinitely), supporting lifetime per-café stats and tax/accounting documentation.

## Business Logic

Appretiate accumulates café-issued stamps against a per-café threshold, converts a
completed set into a one-time redeemable reward, then archives the completed card to a
per-café history and starts the customer on a fresh card.

The rule consumes stamps issued by a café at the counter — one stamp per qualifying item
purchased — measured against a per-café program that defines the stamp threshold, the
reward, and a stamp-expiry policy (a café may choose no expiry or a defined inactivity
window). Its output is a redeemable reward the moment the threshold is reached, plus the
customer notifications that fire as they approach and reach it.

The customer encounters the rule as visible progress that grows with each visit; at the
threshold the reward unlocks and can be redeemed exactly once. On redemption the completed
card does not vanish — it moves into a per-café history. There the customer sees their
cumulative consumption at that café ("how many coffees here"), and the café can produce a
printable record of completed cards for tax and accounting documentation. A fresh card then
begins for the next cycle.

## Access Control

Multi-user with three roles and a clear trust boundary around the value-granting action.

- **Customer** — signs up / signs in with an authenticated account (specific sign-in method
  deferred to stack selection). Can open/hold café stamp cards, present a code to receive a
  stamp, view progress across all their cards, and redeem a reward self-serve once eligible
  (the system enforces one-time redemption). Cannot grant stamps to themselves.
- **Café** — authenticates into a café-scoped role shared at the café level (not per-barista
  accounts). Can scan a customer's code or enter their one-time code to add a stamp to that
  customer's card for this café, and can reverse/correct a mistaken stamp (audit-logged).
  Cannot view or alter unrelated customers' data beyond the stamping interaction.
- **Operator (administrative)** — approves café self-registrations before they go live and
  manages platform-level onboarding. Minimal surface; exists to gate fraud, not to run
  day-to-day café operations.

The value-granting action — **adding a stamp** — is gated behind the authenticated café
role. Customers present a code; the café side scans or enters it. This direction (customer
presents, café verifies) is what prevents self-granted stamps. Redemption is self-serve but
one-time and system-enforced (see FR-010 and Open Questions #2). No per-barista accounts,
no multi-barista accountability, and no café-staff management in the MVP.

## Non-Goals

- **No per-barista accounts / staff management** — the café authenticates as a café-level
  role (FR-002), not individual employees. Per-staff accountability is out of MVP scope.
- **No payment / POS integration** — the app never processes payments or integrates with the
  till; the café only records stamps. This keeps Appretiate off the critical transaction
  path and out of payment-compliance scope.
- **Account recovery is deferred** — not in the MVP (FR-003 demoted to nice-to-have). The
  tension with the durability guardrail is tracked in Open Questions #1.
- **No public café self-go-live** — cafés may self-register but cannot operate until an
  operator approves them (FR-011). Open, unmoderated café onboarding is explicitly out of
  scope for fraud-control reasons.

Deliberately left open (not ruled out as non-goals): a café discovery surface beyond opening
a card, and any cross-café stored value / gift-card concept.

## Open Questions

1. **How is "earned progress never lost" guaranteed if account recovery (FR-003) is
   deferred?** — A deferred recovery path conflicts with the durability guardrail. Owner:
   user. Needs resolution before the guardrail can be claimed as met (e.g. server-side
   account binding, or accept that a lost login = lost progress in v1).
2. **How is self-serve redemption (FR-010) prevented from being triggered without an actual
   purchase?** — Self-serve speed trades away the café-confirm fraud check. Owner: user.
   Options to explore downstream: café glances at the confirmation screen, time/location
   binding, or a lightweight café acknowledgement.
3. **Which jurisdiction's accounting/tax format must the café PDF export (FR-014) satisfy?**
   — Partially resolved: format = printable PDF, granularity = per completed-card line items
   (date + reward value), period-selectable. Still open: the specific jurisdiction's required
   fields / certification and any retention rules. Owner: user. Deferrable until a target
   market is chosen.
