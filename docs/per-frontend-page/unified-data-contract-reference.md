# GokaFoods Ambassador Portal — Unified Data Contract Reference

This document maps every metric that is shared across more than one module to its **single canonical name**, **owning service**, and **every place it's currently exposed**. If you're adding a new endpoint and it needs one of these values, pull it from the owning service — don't recompute or rename it locally.

---

## How to use this

1. Find the metric you need below.
2. Use the **Canonical Field Name** exactly as written — in every module, not just the one you're working in.
3. Check **Computed By** — that service owns the formula. Nobody else recomputes it.
4. If a module currently uses an old/different name for the same thing, it's flagged in the **Aliases Retired** column — those should be migrated.

---

## 1. Identity Fields

| Canonical Field | Type | Source of Truth | Exposed By | Notes |
|---|---|---|---|---|
| `userId` | string | Consumer System (Identity Service) | All modules (via JWT) | Never duplicated; ambassador system never owns identity |
| `fullName`, `email`, `phoneNumber`, `avatarUrl` | string | Consumer System | Settings (`/ambassador/settings`, read-only) | Mutations go through `/consumer/profile`, never the ambassador app |
| `roles` | string[] | Auth System | JWT payload, all modules | `["consumer", "ambassador"]` — additive, never a separate identity |
| `level` (name only, e.g. "City Partner") | string | Levels Service | Settings identity block, Leaderboard rows, Dashboard (indirectly via Levels) | This is a *display projection* of the Levels Service's `currentLevel.name` — Settings must not compute it independently |

---

## 2. Competition & Ranking Fields

| Canonical Field | Type | Computed By | Exposed By | Notes |
|---|---|---|---|---|
| `cycleId` | string | Competition Service | Competition Center, Dashboard, Wallet (`/wallet/rewards`) | Frontend must always source this from `/competition/history` — never guess/construct it |
| `cycle.startsAt` / `cycle.endsAt` / `cycle.daysRemaining` | ISO date / int | Competition Service | Competition Center, Dashboard, Leaderboard | Monthly, not calendar-aligned |
| `competitionScore` | number | Competition Service | Competition Center, Dashboard, Leaderboard | **Must always equal** `referralScore + conversionScore + gmvScore + retentionScore` — no hidden multipliers |
| `rank.current` / `rank.previous` / `rank.movement` / `rank.direction` | int / int / int / string | Competition Service | Competition Center, Dashboard, Leaderboard (`/me`) | |
| `rewardTier.percentage` | number (%) | Competition Service | Competition Center, Dashboard, Wallet (`/wallet/rewards`, as `rewardPercentage`) | Must match the Rank → Reward Percentage table (1=10%, 2=8%, 3=6%, 4=4%, 5=2%) |
| `totalParticipants` | int | Competition Service | Dashboard, Leaderboard | |

---

## 3. Commission & Reward Fields

| Canonical Field | Type | Computed By | Exposed By | Aliases Retired |
|---|---|---|---|---|
| `competitionCommission` | number (NGN) | Rewards Service | Competition Center, Dashboard, **Wallet** (`/wallet`, `/wallet/rewards`), Analytics | `totalRevenue` (old), `revenueThisCycle` (Wallet), `cycleRevenue` (Wallet rewards history) — all migrated to `competitionCommission` in this refinement pass |
| `estimatedReward` | number (NGN) | Rewards Service | Competition Center, Dashboard | `= competitionCommission × rewardTier.percentage`. Never presented as guaranteed. |
| `rewardEarned` / `rewardAmount` | number (NGN) | Rewards Service | Competition Center (history/detail), Wallet (`/wallet/rewards`) | Settled version of `estimatedReward`, after cycle ends |

**Canonical formula:**
```text
Competition Reward = Competition Commission × Reward Percentage(Current Rank)
```

> Competition Commission is earnings-system only. Competition Score is ranking-system only. They are never merged in any calculation — per the Business Separation Rule.

---

## 4. GMV & Attribution Fields

| Canonical Field | Type | Computed By | Exposed By | Notes |
|---|---|---|---|---|
| `qualifiedCompetitionGMV` | number (NGN) | Metrics Service | Competition Center, Analytics, Dashboard (as `gmv.competition`) | Cycle-bound + attribution-filtered (90-day window, max 5 orders/customer, excludes refunds) |
| `gmvScore` | number (points) | Metrics Service | Competition Center | **Must equal** `qualifiedCompetitionGMV ÷ 1000` exactly (1,000 NGN = 1 point). If a weighting multiplier is ever intended, it must be added to this rule explicitly — not left implicit in example payloads. |
| `lifetimeAttributedGMV` | number (NGN) | Metrics Service | Analytics (`/analytics?include=qualifiedGMV`) | Same underlying ledger as `lifetimeGMV` below |
| `lifetimeGMV` | number (NGN) | Metrics Service | Ambassador Levels (`requirements.lifetimeGMV`) | **Same field as `lifetimeAttributedGMV`**, exposed under a different name by Levels. Must be computed once, by one shared append-only ledger, not duplicated logic in two services. |

**Attribution Window Rule:**
```text
A customer's attribution to an ambassador expires after 90 days OR 5 orders,
whichever comes first. After expiry: GMV attribution stops, retention scoring
stops, analytics visibility ends, customer becomes platform-owned.
```

> **Important interaction:** Because attribution expires but Ambassador Levels never resets, `lifetimeGMV` / `lifetimeAttributedGMV` must be append-only — GMV is credited permanently the moment it's validly attributed, and is **never clawed back** once a customer's window closes.

---

## 5. Retention Fields

| Canonical Field | Type | Computed By | Exposed By | Notes |
|---|---|---|---|---|
| `retention.score` / `retention.points` | number | Metrics Service | Analytics (`score`), Dashboard (`points`) | Field name differs by module (`score` vs `points`) — same value, consider unifying in a future pass |
| `retention.breakdown.order2` / `order3` / `order4` / `order5` | object or int | Metrics Service | Analytics (nested `{customers, qualifiedOrders}`), Dashboard (flat int) | Key naming unified to `order2`–`order5` in this refinement pass (Analytics previously used `secondOrder`/`thirdOrder`/etc.) |

**Canonical points table** (this is the single source — every retention score everywhere must reduce to this):

| Order | Points |
| ----: | -----: |
| 2 | 10 |
| 3 | 15 |
| 4 | 20 |
| 5 | 25 |

```text
Retention Score = Σ (customers in tier × points for that tier)
```
No points awarded past the 5th order.

---

## 6. Referral Lifecycle Fields

| Canonical Field | Type | Computed By | Exposed By | Notes |
|---|---|---|---|---|
| `joined` / `ordered` / `inactive` | int | Referral Service | Referral Management (`/referrals`), Dashboard (`referrals` block) | State-based, not time-based. `joined = ordered + inactive` |
| `conversionRate` | number (%) | Referral Service | Referral Management (`/referrals`, `/referrals/stats`), Dashboard (as `competitionRate`/`conversionRate`), Analytics | `= ordered / joined × 100` |
| `conversions` | int | Referral Service | Dashboard, Analytics, **Ambassador Levels** | Renamed from `qualifiedCustomers` in Levels during this refinement pass — same metric everywhere now |
| `referralCode` / `referralUrl` | string | Referral Service | Referral Management (`/referrals`, `/referrals/link`) | |
| `qrCodeUrl` | string | Referral Service | Referral Management (`/referrals`, `/referrals/qr`) | |
| `repeatOrderRate` / `inactivityRate` / `ordersPerConvertedReferral` | number | Referral Service | Referral Management (`/referrals/stats`) | Deeper effectiveness ratios, distinct from the basic lifecycle counts above |

---

## 7. Wallet Ledger Fields

| Canonical Field | Type | Computed By | Exposed By | Notes |
|---|---|---|---|---|
| `availableBalance` | number (NGN) | Wallet Service | Wallet (`/wallet`) | `= Σ(Credits) − Σ(Debits)`, never edited directly |
| `pendingEarnings` | number (NGN) | Wallet Service | Wallet, Dashboard (`earnings.pendingEarnings`) | Settlement-delay bucket — part of Earned → Pending → Withdrawable lifecycle |
| `withdrawableBalance` | number (NGN) | Wallet Service | Wallet, Dashboard | Confirmed, cleared funds only |
| `lifetimeEarnings` | number (NGN) | Wallet Service | Wallet, Dashboard (`earnings.totalEarnings`) | Field name differs slightly (`lifetimeEarnings` vs `totalEarnings`) — same metric, worth unifying |
| `totalWithdrawn` / `totalTransferred` | number (NGN) | Wallet Service | Wallet | |
| `quickActions.canWithdraw` / `canTransferToConsumerWallet` | bool | Wallet Service | Wallet | Always backend-supplied — frontend never hardcodes eligibility |

**Settlement Lifecycle (formerly "Model B"):**
```text
Earned → Pending → Withdrawable
```

---

## 8. Ambassador Levels Fields

| Canonical Field | Type | Computed By | Exposed By | Notes |
|---|---|---|---|---|
| `currentLevel.id` / `.name` / `.badge` / `.color` | mixed | Levels Service | Levels (`/ambassador/levels`), Settings (identity projection), Leaderboard (`level`) | Five-tier hierarchy: Campus/Community → City → State → National → Global |
| `progress.overallProgressPercentage` | number (%) | Levels Service | Levels | `= average(progressPercentage across all next-level requirements)` |
| `requirements.lifetimeReferrals` / `.conversions` / `.lifetimeGMV` | object | Levels Service | Levels | `conversions` shares its name with the Referral/Dashboard/Analytics field (see §6); `lifetimeGMV` shares its ledger with Analytics' `lifetimeAttributedGMV` (see §4) |
| `ambassadorCommission` / `customerDiscount` / `discountedOrdersAllowed` | string/string/int | Levels Service | Levels (`/ambassador/levels`, `/ambassador/levels/catalog`) | `customerDiscount = ambassadorCommission ÷ 4`, always |

---

## 9. Settings / Configuration Fields

| Canonical Field | Type | Owned By | Notes |
|---|---|---|---|
| `payment.*` | object | Wallet Service | Settings only aggregates/configures, never owns |
| `notifications.*` | object | Ambassador System (Settings) | Fully ambassador-owned, no cross-module dependency |
| `security.*`, `sessions.*` | object | Auth Service | Settings is a client only — all mutating actions (password change, 2FA, session termination) are delegated |
| `preferences.currency` / `.timezone` | string | Ambassador System (Settings) | UI/UX personalization only, no business logic dependency |

---

## Appendix A — Renames Applied in This Refinement Pass

| Old Name | New Canonical Name | Where |
|---|---|---|
| `totalRevenue` | `competitionCommission` | Competition Center (original rename) |
| `revenueThisCycle` | `competitionCommission` | Wallet (`/wallet`) |
| `cycleRevenue` | `competitionCommission` | Wallet (`/wallet/rewards`) |
| `qualifiedCustomers` | `conversions` | Ambassador Levels |
| `secondOrder` / `thirdOrder` / `fourthOrder` / `fifthOrder` | `order2` / `order3` / `order4` / `order5` | Analytics Hub (matched to Dashboard's existing convention) |

## Appendix B — Open Items Still Needing a Product Decision

These are documentation/example bugs that were corrected for internal consistency, but the *underlying formulas themselves* are worth a quick sign-off since they directly affect ambassador payouts:

1. **GMV-to-score ratio:** confirmed at 1,000 NGN = 1 point. If a weighting multiplier was actually intended (the original example implied something closer to ~2.24:1), that needs to be decided and written into §4 above before it ships.
2. **Retention points table:** confirmed at 10/15/20/25 for orders 2–5. Worth a final sign-off since it directly determines `competitionScore` and therefore reward payouts.
3. **`lifetimeEarnings` vs `totalEarnings` naming** (Wallet vs Dashboard) — flagged but not renamed in this pass since it's lower-risk than the others; worth a quick cleanup pass.
4. **No historical leaderboard endpoint exists** — `/competition/current/leaderboard` only covers the live cycle. If "past month's leaderboard" is ever a requirement, it needs its own endpoint design.