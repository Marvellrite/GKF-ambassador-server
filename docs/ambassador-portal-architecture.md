# GokaFoods Ambassador Portal Architecture (Refined)

> Refines and supersedes the original Ambassador Portal Architecture.
> This document is the canonical source of truth for cross-module rules.
> Module-level contracts live in their own specs:
> Ambassador Levels API · Analytics Hub API · Competition Leaderboard API ·
> Competition Center API · Ambassador Dashboard API · Referral Management API ·
> Ambassador Settings & Profile API · Wallet & Financial Management API ·
> Ambassador System Integration Architecture (Identity)

---

# Overview

The GokaFoods Ambassador Portal is a unified identity, performance, competition, analytics, and financial management ecosystem built around a recurring monthly growth economy.

The system is built on **five foundational layers** (previously four — Identity was missing):

* **Identity Layer:** A single shared user identity (`roles: ["consumer", "ambassador"]`) that the ambassador app extends rather than owns. See the Integration Architecture spec for cookie-domain vs. token-handoff strategies.
* **Commerce Layer:** Earnings generated from referrals, conversions, and attributed GMV.
* **Competition Layer:** Monthly ranking system that rewards top-performing ambassadors.
* **Insight Layer:** Multi-temporal analytics and optimization tools.
* **Finance Layer:** Ambassador wallet management, bank withdrawals, and internal transfers.

```text
Compete → Earn → Improve → Reward → Reset → Repeat
```

The ambassador system is **not a separate identity** — it is a role-extended experience layer on top of the consumer identity. No duplicate accounts, no identity fragmentation, one login unlocks both apps.

---

# Core Principles

```text
Competition Score → Determines Rank

Rank → Determines Reward Percentage

Competition Commission → Determines Reward Amount
```

These are independent systems working together. None of them should ever be derived on the frontend.

---

# Core Architecture Model

## Data Processing Pipeline

```text
Raw Data
    ↓
Computation Layer
    ↓
Response DTO
    ↓
UI Rendering
```

## Raw Data Layer

Entities: Users, Orders, Referrals, Transactions, Wallet Transactions, Competition Cycles, Attribution Windows, Customer Events, Withdrawal Records.

## Computation Layer

Centralized in the Competition Service. Responsibilities: competition score calculation, rank generation, reward calculation, commission aggregation, GMV aggregation, retention scoring, conversion calculations, attribution validation, wallet balance computation, reward projection generation.

## Response DTO Layer

Responses are grouped into three categories everywhere in the system:

| Category | Purpose | Examples |
|---|---|---|
| Raw Metrics | Transparency | Referrals, orders, wallet balances |
| Computed Metrics | Business logic | Score, rank, conversion rate, retention score |
| Projection / Guidance Metrics | User direction | Estimated reward, rank movement, recommendations, next-level preview |

The frontend is a **renderer, not a calculator**, in every module without exception.

---

# Key System Rules

## 1. Attribution Window Rule

Every referred customer belongs to an ambassador for a limited period, expiring after **90 days OR 5 orders**, whichever occurs first. After expiry:

* GMV attribution stops
* Retention scoring stops
* Analytics visibility ends
* Customer becomes platform-owned

This prevents permanent customer ownership.

> **Clarification (new): Attribution expiry vs. permanent Levels progression**
> Ambassador Levels (a permanent, never-reset system) requires *Lifetime GMV* as a leveling input. Because attribution formally expires, "Lifetime GMV" must be implemented as an **append-only historical ledger**: GMV is credited to the ambassador's lifetime total at the moment it is validly attributed, and it is never clawed back once a customer's window closes. The cycle-scoped `qualifiedCompetitionGMV` (Competition/Analytics) and the permanent `lifetimeAttributedGMV` (Analytics) / `Lifetime GMV` (Levels) are **the same underlying running total** and must be computed by one shared service, not maintained independently by the Levels Service and the Analytics Service.

## 2. Retention System

Bonus scoring, separate from GMV. Only orders 2–5 qualify.

| Order | Points |
| ----: | -----: |
|     2 |     10 |
|     3 |     15 |
|     4 |     20 |
|     5 |     25 |

No points after the 5th order. **This table is canonical** — any module displaying a retention score must equal `Σ(customers in tier × points for that tier)` exactly. A mismatch is a bug, not a stylistic difference (see Appendix A).

## 3. Competition Score Formula

```text
Competition Score =
Referral Score +
Conversion Score +
GMV Score +
Retention Score
```

No hidden multipliers. **The breakdown components returned by `/competition/current/performance` must always sum exactly to the `competitionScore` returned by `/competition/current`.** Any endpoint where they don't is a backend defect (see Appendix A).

## 4. GMV Rule

Only attributed orders contribute to GMV.

```text
NGN 1,000 GMV = 1 Point
```

GMV tracking ends after 90 days or 5 orders, whichever comes first. `gmvScore` in any response must equal `qualifiedCompetitionGMV / 1000` exactly, unless a documented weighting multiplier is introduced here — if a multiplier is intended, it must be added to this rule explicitly rather than left implicit in example payloads (see Appendix A).

---

# Monthly Competition System

## Competition Cycle

* Duration: 1 calendar month
* Rankings reset every month; historical records remain available
* Competitions are independent cycles, not calendar/quarterly grouped

```text
Month Start → Competition Active → Leaderboard Locked → Rewards Distributed → Monthly Reset
```

---

# Monthly Reward System

## Reward Philosophy

No fixed prize pools. Top performers receive a percentage of the **Competition Commission** they generated during the cycle.

## Reward Formula

```text
Competition Reward = Competition Commission × Rank Percentage
```

## Reward Distribution

| Rank | Percentage |
| ---: | ---------: |
|    1 |        10% |
|    2 |         8% |
|    3 |         6% |
|    4 |         4% |
|    5 |         2% |

Only the Top 5 receive rewards.

## Competition Commission Definition

> Formerly called "Cycle Revenue" / "totalRevenue" / "revenueThisCycle" in earlier drafts. **`competitionCommission` is now the single canonical field name** for this metric across every module — Dashboard, Competition Center, and **Wallet**. Wallet's `revenueThisCycle` and the reward-history `cycleRevenue` field should be renamed to `competitionCommission` to match (see Appendix A).

```text
Competition Commission =
All eligible earnings generated during the current competition cycle
```

Eligible: referral commissions, conversion commissions, GMV commissions, approved ambassador incentives.
Excluded: previous cycle earnings, previous competition rewards, historical wallet balances, manual adjustments.

## Reward Projection System

Dynamic, never a guarantee.

```text
Estimated Reward = Competition Commission × Reward Percentage(Current Rank)
```

---

# Financial System Boundaries

```text
Ambassador Wallet System (This Project)
                 |
       ---------------------
       |                   |
       ↓                   ↓
Personal Bank      GKF Consumer Wallet
(External)         (Separate Project)
```

## Settlement Lifecycle (formerly "Model B")

```text
Earned → Pending → Withdrawable
```

* Pending = settlement delay (e.g. 7 days), enforced for payout safety
* Withdrawable = confirmed cleared funds
* Financial state is independent of competition cycles

## Wallet Ownership Rules

**Ambassador Wallet** — funds belong to the ambassador, sourced from referral/conversion/GMV commissions, competition rewards, and approved incentives. Ambassadors can view balances, withdraw to personal banks, and transfer to a GKF Consumer Wallet. Ambassadors cannot transfer to other ambassadors, modify balances directly, or reverse completed withdrawals.

**GKF Consumer Wallet** — a separate system; this project only supports one-way internal transfers into it. No reverse flow exists back to the Ambassador Wallet or to a personal bank.

```text
Ambassador Wallet → Consumer Wallet → Food Orders        (one-way only)
```

---

# Time-Window Philosophy (consolidated)

Every module has its own time-awareness rule. This table is the single place that reconciles them — previously this was scattered across nine separate documents with no shared summary.

| Module | Time Role | Notes |
|---|---|---|
| Analytics Hub | **Primary, multi-temporal dimension** | No single "page time truth" — primary, comparison, trend, retention, and GMV-attribution windows can all differ simultaneously. Header must never show a single date range. |
| Competition Center | **Cycle-bound** | Monthly, not calendar-based. `/current/performance` is strictly live-only, no historical query support. |
| Leaderboard | **Cycle-bound, current only** | Always reflects the active competition; no historical leaderboard endpoint exists yet (see Appendix B). |
| Referral Management | **Optional filter only** | State-based core (Joined/Ordered/Inactive); time is an overlay, never the primary dimension. |
| Wallet | **Current-state + optional history filters** | Summary fields (`availableBalance`, `pendingEarnings`, `withdrawableBalance`, `lifetimeEarnings`) always reflect now. Only `competitionCommission` is cycle-scoped. |
| Ambassador Levels | **Not time-window aware** | Permanent, cumulative, never resets. History endpoint is timestamped, but the level itself isn't queried by range. |
| Settings | **Non-time-windowed** | Exceptions: last password change, last login, session timestamps. |

---

# Portal Structure

## 0. Identity & Cross-System Integration

Shared identity via `roles: ["consumer", "ambassador"]` on one JWT/session. Two supported sync strategies:

* **Option A (recommended):** Shared cookie domain (`.gokafoods.com`), httpOnly cookies, no token passing.
* **Option B (fallback):** Secure token-handoff redirect (`/auth/sync?token=`) for fully independent deployments.

Role verification always happens server-side; the frontend never trusts a locally cached role.

## 1. Ambassador Dashboard

```http
GET /ambassador/dashboard
```

Single aggregated snapshot: Competition Snapshot, Performance Metrics, Revenue & Earnings, Estimated Competition Reward, Rank Progress, Referral Overview, Recent Activity Feed, Smart Insights (not yet active in production).

## 2. Competition Center

```http
GET /competition/current?includeBreakdown=true
GET /competition/current/performance
GET /competition/history?page=&limit=&status=
GET /competition/:competitionId
GET /competition/compare?previous=&current=
GET /competition/compare/latest
```

Cycle-bound, not calendar-bound. Competition Score (ranking) and Competition Commission (earnings) must never be merged in UI or computation. Frontend must never guess competition IDs — always source them from `/competition/history`.

## 3. Leaderboard System

```http
GET /competition/current/leaderboard?page=&limit=
GET /competition/current/leaderboard/me
```

Strictly non-financial — never exposes commission, rewards, wallet balance, or revenue. Allowed fields only: rank, display name, level, competition score. The `/me` endpoint returns surrounding context (±2 ranks) so the frontend never needs pagination to find the current user.

## 4. Analytics Hub

```http
GET /analytics?range=&include=&compare=
GET /analytics/trends?range=&metric=&granularity=
GET /analytics/comparison?from=&to=&compareFrom=&compareTo=
```

Inherently multi-temporal (see Time-Window Philosophy). No ranking data of any kind. All deltas, percentages, and trends are precomputed backend DTOs — the frontend computes nothing.

## 5. Referral Management

```http
GET /referrals
GET /referrals/list?status=&page=&limit=&search=&sort=
GET /referrals/stats
GET /referrals/link
GET /referrals/qr
```

State-based (Joined / Ordered / Inactive), not time-based. Time filtering on `/referrals/list` is an optional overlay only.

## 6. Wallet & Financial Management

```http
GET /wallet
GET /wallet/transactions?page=&limit=&type=&category=&from=&to=&sort=
GET /wallet/withdrawals?status=&page=&limit=&from=&to=
GET /wallet/transfers?page=&limit=&status=&from=&to=
GET /wallet/rewards?competitionId=&page=&limit=
POST /wallet/withdraw
POST /wallet/transfer-to-consumer-wallet
```

Ledger-driven: `Wallet Balance = Σ(Credits) − Σ(Debits)`. No balance is ever edited directly. `quickActions` (`canWithdraw`, `canTransferToConsumerWallet`) are always backend-supplied, never hardcoded on the frontend.

## 7. Ambassador Levels

```http
GET /ambassador/levels
GET /ambassador/levels/catalog
GET /ambassador/levels/history
```

Permanent progression, never resets, not driven by XP — driven directly by lifetime requirement completion. Overall progress is the average of each requirement's individual completion percentage.

## 8. Settings & Profile

```http
GET /ambassador/settings
GET /ambassador/payment-details
POST /ambassador/payment-details
PATCH /ambassador/payment-details
GET /ambassador/notifications/settings
PATCH /ambassador/notifications/settings
GET /ambassador/security
GET /ambassador/security/sessions
DELETE /ambassador/security/sessions/:id
```

`/ambassador/settings` is an **aggregation layer**, not an identity owner. Identity fields are a read-only projection from the Consumer System; identity mutations always go through `/consumer/profile`. Security actions are delegated to the Auth Service; payment actions are delegated to the Wallet Service.

---

# Backend Architecture

* **Identity / Auth Service** — shared JWT, role verification, session management, password/2FA actions.
* **Competition Service** — scores, rankings, reward calculations, monthly cycles.
* **Metrics Service** — aggregations, attribution validation, GMV calculations (single source of truth for lifetime + cycle-scoped GMV, shared with Levels).
* **Rewards Service** — competition rewards, wallet crediting, settlement.
* **Referral Service** — referral tracking, attribution management, lifecycle state.
* **Wallet Service** — earnings ledger, withdrawals, consumer-wallet transfers, transaction history.
* **Settings Aggregation Service** — composes identity (read-only), payment config, notifications, security, and preferences from the services above.

---

# System Philosophy

* **Transparency** — every score, reward, and calculation is explainable.
* **Fairness** — no permanent customer ownership.
* **Sustainability** — rewards scale with actual value generated.
* **Separation of Concerns** — ambassador finances and consumer finances remain separate systems; identity is never duplicated.
* **Consistency (new)** — any two endpoints describing the same underlying metric (score breakdown vs. total, GMV vs. GMV score, rank vs. reward tier, Wallet's commission field vs. Competition Center's) must reconcile exactly. A documentation example that violates a stated formula is treated as a defect to fix, not a precedent to follow.
* **Continuous Engagement** — monthly resets keep ambassadors active; Levels remain permanent underneath the monthly competition layer.

---

# Appendix A — Cross-Document Issues Found (Pending Fixes)

1. **Retention score doesn't match the points table.** Analytics Hub and Dashboard both show a retention example of 4/3/2/5 customers across orders 2–5 with a stated score of **220**. Using the canonical table above, the correct total is **250** (4×10 + 3×15 + 2×20 + 5×25). Fix the example or confirm whether the points table itself needs to change.
2. **GMV score doesn't match the GMV rule.** Competition Center shows `qualifiedCompetitionGMV: 1,250,000` alongside `gmvScore: 2800`. Per the 1,000 NGN = 1 point rule, this should be **1250**, not 2800.
3. **Competition Score doesn't equal its own breakdown.** Competition Center's default `/competition/current` response shows `competitionScore: 12540`, but `/competition/current/performance`'s breakdown for the same cycle (4200+6100+2800+1000) sums to **14100**.
4. **Reward tier doesn't match rank.** Competition History/Detail for `cmp_2026_05` shows `rank: 5` with `rewardTier: 4` and `rewardEarned: 18400`. Per the Reward Distribution table, rank 5 = 2%, so this should be `rewardTier: 2`, `rewardEarned: 9200`.
5. **Naming drift on Competition Commission.** Wallet uses `revenueThisCycle` (summary) and `cycleRevenue` (rewards history) for what Competition Center calls `competitionCommission`. Standardize on `competitionCommission` everywhere (applied in this refined doc).

# Appendix B — Open Gaps

* `GET /referrals/stats` is listed in the Referral Management API's endpoint summary but never documented with a purpose, request, or response.
* `GET /competition/current/performance` is documented in the Competition Center spec but missing from that spec's own top-level "Endpoints Summary" list.
* No historical (non-current) leaderboard endpoint is defined anywhere, even though Competition History exists for individual ambassadors.