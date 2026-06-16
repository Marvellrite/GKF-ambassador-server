# Competition Center API Specification (Refined)

## Overview

The Competition Center is the authenticated ambassador’s competition intelligence system.

It provides:

* Live competition performance
* Score breakdown and financial summary
* Historical competition cycles
* Competition comparisons
* Cycle-based analytics per ambassador

It is strictly **cycle-based**, not calendar-based, and all computations are derived from the current competition engine.

---

# Core Design Rules

## 1. Cycle Definition

```text
Current Competition (LIVE)
Cycle: Jun 01 – Jun 30
```

* Competitions are independent cycles
* A new cycle begins immediately after the previous ends
* No yearly or quarterly grouping logic

---

## 2. Financial Naming Standard

| Old Name     | New Name              |
| ------------ | --------------------- |
| totalRevenue | competitionCommission |

Competition Commission represents:

> Ambassador earnings from the competition cycle only

---

## 3. ID Source Rule

Frontend MUST NEVER guess competition IDs.

IDs must come from:

```http
GET /competition/history
```

---

## 4. Business Separation Rule

* Competition Score → Ranking system
* Competition Commission → Earnings system

They must NEVER be merged in UI or calculations.

---

# Endpoints Summary

```http
GET /competition/current

GET /competition/history?page=&limit=

GET /competition/:competitionId

GET /competition/compare?previous=&current=

GET /competition/compare/latest
```

---

# 1. Current Competition

## Endpoint

```http
GET /competition/current
```

---

## Purpose

Returns ambassador’s real-time performance in the active competition cycle.

---

## Optional Behavior (Important)

```http
GET /competition/current?includeBreakdown=true
```

### If `includeBreakdown=false` or omitted:
The response returns a **summary view only**.

### If `includeBreakdown=true`:
The response also includes:

* Competition performance breakdown
* Score composition
* Attribution metrics summary

---

## Response (Default)

```json
{
  "competition": {
    "cycleId": "cmp_2026_06",
    "status": "LIVE",

    "cycle": {
      "startsAt": "2026-06-01T00:00:00Z",
      "endsAt": "2026-06-30T23:59:59Z",
      "daysRemaining": 12
    },

    "rank": {
      "current": 4,
      "previous": 6,
      "movement": 2,
      "direction": "up"
    },

    "competitionScore": 12540,

    "rewardTier": {
      "percentage": 4
    },

    "competitionCommission": 380000,

    "estimatedReward": 15200
  }
}
```

---

## Response (With `includeBreakdown=true`)

Adds:

```json
{
  "performance": {
    "referralScore": 4200,
    "conversionScore": 6100,
    "gmvScore": 2800,
    "retentionScore": 1000,

    "qualifiedCompetitionGMV": 1250000,

    "referrals": 120,
    "conversions": 58,
    "activeAttributedCustomers": 42
  }
}
```

---

## UI Display

```text
Current Competition (LIVE)

Cycle
Jun 01 – Jun 30

Current Rank
#4
↑ 2 positions

Competition Score
12,540 pts

Reward Tier
4%

Competition Commission
₦380,000

Estimated Reward
₦15,200

Days Remaining
12
```

---

# 2. Competition Performance

## Endpoint

```http
GET /competition/current/performance
```

---

## Purpose

Breaks down how the **competition score is formed**.

> This endpoint is strictly cycle-bound and only reflects the active competition.

---

## Design Rule

This endpoint MUST NOT support historical queries.

Reason:
Score composition is only meaningful in the context of a live ranking system.

---

## Response

```json
{
  "performance": {
    "referralScore": 4200,
    "conversionScore": 6100,
    "gmvScore": 2800,
    "retentionScore": 1000,

    "qualifiedCompetitionGMV": 1250000,

    "referrals": 120,
    "conversions": 58,
    "activeAttributedCustomers": 42
  }
}
```

---

## UI Display

```text
Referral Score
4,200

Conversion Score
6,100

GMV Score
2,800

Retention Score
1,000

Qualified Competition GMV
₦1,250,000

Referrals
120

Conversions
58

Active Customers
42
```

---

# 3. Competition History

## Endpoint

```http
GET /competition/history?page=1&limit=10&status=ENDED
```

---

## Purpose

Returns past competition cycles for analysis and comparison.

---

## Request

| Field  | Type              | Description                  |
|--------|------------------|------------------------------|
| page   | number            | pagination page              |
| limit  | number            | items per page               |
| status | string (optional) | LIVE / ENDED filter          |

---

## Response

```json
{
  "history": [
    {
      "cycleId": "cmp_2026_05",
      "cycleName": "May Cycle",
      "status": "ENDED",
      "completedAt": "2026-05-31T23:59:59Z",

      "rank": 5,
      "competitionScore": 14100,

      "competitionCommission": 460000,
      "rewardEarned": 18400
    }
  ],

  "pagination": {
    "page": 1,
    "limit": 10,
    "totalPages": 6,
    "totalItems": 58
  }
}
```

---

## UI Display

```text
May Cycle

Final Rank
#5

Competition Score
14,100

Competition Commission
₦460,000

Reward Earned
₦18,400
```

---

# 4. Competition Detail

## Endpoint

```http
GET /competition/:competitionId
```

---

## Purpose

Returns full breakdown of a specific competition cycle.

---

## Request

| Param | Description |
|------|------------|
| competitionId | Cycle identifier |

---

## Response

```json
{
  "competition": {
    "cycleId": "cmp_2026_05",

    "rank": 5,
    "competitionScore": 14100,

    "breakdown": {
      "referralScore": 4200,
      "conversionScore": 6100,
      "gmvScore": 2800,
      "retentionScore": 1000
    },

    "competitionCommission": 460000,

    "rewardTier": 4,
    "rewardEarned": 18400
  }
}
```

---

## UI Display

```text
May Cycle

Final Rank
#5

Competition Score
14,100

Referral Score
4,200

Conversion Score
6,100

GMV Score
2,800

Retention Score
1,000

Competition Commission
₦460,000

Reward Earned
₦18,400
```

---

# 5. Competition Comparison

## Rule

Frontend MUST use IDs from:

```http
GET /competition/history
```

---

## Endpoint

```http
GET /competition/compare?previous=&current=
```

---

## Alternative (Recommended)

```http
GET /competition/compare/latest
```

Backend auto-resolves:

* current cycle
* previous cycle

---

## Response

```json
{
  "comparison": {
    "previous": {
      "rank": 5,
      "score": 14100,
      "competitionCommission": 460000
    },
    "current": {
      "rank": 2,
      "score": 15600,
      "competitionCommission": 510000
    },
    "delta": {
      "rankChange": 3,
      "scoreChange": 1500,
      "commissionChange": 50000
    }
  }
}
```

---

## UI Display

```text
Rank
#5 → #2

Competition Score
14,100 → 15,600

Competition Commission
₦460,000 → ₦510,000

Difference
+₦50,000
```

---

# Backend Responsibilities

The backend must:

* Compute competition score
* Compute competition commission
* Compute reward projections
* Compute rank movement
* Build score breakdown
* Maintain competition snapshots
* Resolve comparison logic
* Enforce cycle boundaries

The frontend must NOT:

* Compute rankings
* Derive rewards
* Guess competition IDs
* Recalculate scores

---

# Final Architecture Insight

The Competition Center is a **cycle intelligence system**, not a leaderboard system.

It answers:

* Where am I now?
* Why am I here?
* What did I earn?
* How did I perform before?
* How do I compare across cycles?