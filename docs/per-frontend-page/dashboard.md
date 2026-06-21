# Ambassador Dashboard API Specification (Refined)

## Endpoint

```http id="d1a9qp"
GET /ambassador/dashboard
```

---

# Overview

The Ambassador Dashboard is a **real-time aggregated intelligence endpoint** that powers the main ambassador home screen.

It consolidates:

* Competition performance (cycle-based)
* Attribution performance (behavior-based)
* Financial status (settlement-based)
* Rank progression
* Referral funnel health
* Activity stream

It answers:

> How am I performing right now, and what should I focus on next?

---

# Core System Architecture

## 1. Metric Scope Model

All dashboard metrics belong to exactly one of three scopes:

### A. Competition Scope

Cycle-bound metrics used for ranking:

* Competition Score
* Competition Commission
* Qualified GMV (cycle-based)
* Rank
* Reward tier

```text id="csp1"
Valid only within current competition cycle
```

---

### B. Attribution Scope

Customer-based behavioral metrics:

Rules:

* 90-day attribution window (configurable)
* Max 5 orders per customer contribution
* Excludes refunded/reversed orders

```text id="att1"
Attribution = customer relationship validity window
```

---

### C. Financial Scope (Settlement Lifecycle)

Real money lifecycle:

```
Earned → Pending → Withdrawable
```

Rules:

* Pending = settlement delay (e.g. 7 days)
* Withdrawable = confirmed cleared funds

```text id="fin1"
Financial state is independent of competition cycles
```

---

# 2. Attribution Rule Engine

A customer contributes ONLY if:

* Within attribution window
* Has not exceeded order cap (e.g. 5 orders)
* Order is valid (not refunded or reversed)

---

# Dashboard Data Structure

```text id="ui0"
Dashboard
├── Competition Snapshot
├── Performance Metrics
├── Revenue & Earnings
├── Estimated Competition Reward
├── Rank Progress
├── Referral Overview
├── Recent Activity Feed
└── Smart Insights (Future Feature)
```

---

# 1. Competition Snapshot

## Endpoint Section

Included in:

```http id="ep1"
GET /ambassador/dashboard
```

---

## Response

```json id="r1"
{
  "competition": {
    "cycleId": "cmp_2026_06",

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
      "percentage": 4,
      "eligible": true
    },

    "competitionCommission": 380000,

    "estimatedReward": 15200,

    "totalParticipants": 347
  }
}
```

---

## UI Display

```text id="ui1"
Current Competition (LIVE)

Cycle
Jun 01 – Jun 30

Rank
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

Participants
347
```

---

# 2. Performance Metrics

## Response

> **Fixed:** `retention.points` corrected from `220` to `250` to match the canonical retention points table (4×10 + 3×15 + 2×20 + 5×25 = 250). The `orderBreakdown` counts themselves were already correct and are unchanged.

```json id="r2"
{
  "performance": {
    "referrals": {
      "competition": 48,
      "lifetime": 312
    },

    "conversions": {
      "competition": 22,
      "lifetime": 180,
      "competitionRate": 45.8
    },

    "gmv": {
      "competition": 1250000
    },

    "retention": {
      "points": 250,
      "orderBreakdown": {
        "order2": 4,
        "order3": 3,
        "order4": 2,
        "order5": 5
      }
    },

    "attributedCustomers": {
      "active": 17,
      "nearOrderLimit": 3,
      "nearTimeLimit": 1
    }
  }
}
```

---

## UI Display

### Referrals

```text id="ui2"
This Competition
48

Lifetime
312
```

---

### Conversions

```text id="ui3"
This Competition
22

Conversion Rate
45.8%

Lifetime
180
```

---

### GMV (Qualified)

```text id="ui4"
Competition GMV
₦1,250,000
```

---

### Retention

```text id="ui5"
Retention Points
250
```

```text id="ui6"
Attributed Customer Orders

2nd Order → 4 customers
3rd Order → 3 customers
4th Order → 2 customers
5th Order → 5 customers
```

---

### Attributed Customers

```text id="ui7"
Active Customers
17

Near Order Limit
3

Near Time Limit
1
```

---

# 3. Revenue & Earnings

## Response

```json id="r3"
{
  "earnings": {
    "competitionCommission": 380000,

    "totalEarnings": 1250000,

    "pendingEarnings": 65000,

    "withdrawableBalance": 1185000,

    "lastPayout": {
      "amount": 120000,
      "date": "2026-05-31T00:00:00Z"
    }
  }
}
```

---

## UI Display

### Competition Commission

```text id="ui8"
₦380,000
```

---

### Lifetime Earnings

```text id="ui9"
₦1,250,000
```

---

### Pending (Settlement Lifecycle)

```text id="ui10"
₦65,000
```

---

### Withdrawable

```text id="ui11"
₦1,185,000
```

---

### Last Payout

```text id="ui12"
₦120,000
2026-05-31
```

---

# 4. Estimated Competition Reward

## Response

```json id="r4"
{
  "rewardProjection": {
    "currentRank": 4,
    "rewardPercentage": 4,
    "eligible": true,
    "estimatedReward": 15200,

    "calculation": {
      "competitionCommission": 380000,
      "percentage": 4
    }
  }
}
```

---

## UI Display

```text id="ui13"
Estimated Competition Reward
₦15,200

Rank #4

Reward Tier
4%
```

---

# 5. Rank Progress

## Response

```json id="r5"
{
  "rankProgress": {
    "currentRank": 4,
    "nextRank": 3,
    "pointsGap": 630,
    "estimatedRevenueGap": 70000
  }
}
```

---

## UI Display

```text id="ui14"
Current Rank
#4

Next Rank
#3

Points Needed
630

Estimated Revenue Needed
₦70,000
```

---

# 6. Referral Overview

## Response

```json id="r6"
{
  "referrals": {
    "joined": 48,
    "ordered": 22,
    "inactive": 26,
    "activeAttributedCustomers": 17
  }
}
```

---

## UI Display

```text id="ui15"
Joined
48

Converted
22

Inactive
26

Active Customers
17
```

---

# 7. Recent Activity Feed

## Response

```json id="r7"
{
  "recentActivities": [
    {
      "type": "order",
      "message": "Customer placed 2nd order",
      "scoreAdded": 10,
      "createdAt": "2026-06-14T09:15:00Z"
    }
  ]
}
```

---

## UI Display

```text id="ui16"
+10 Points
Customer placed 2nd order
2 mins ago

+50 Points
New referral converted
Yesterday
```

---

# 8. Smart Insights (Future Feature)

## Status

This feature is NOT active in the current system.

It must NOT be rendered in production UI.

---

## Planned Response

```json id="r8"
{
  "insights": [
    {
      "type": "opportunity",
      "title": "Move Into Top 3",
      "description": "You need 630 more points."
    }
  ]
}
```

---

## Future UI Concept Only

```text id="ui17"
Opportunity
Move Into Top 3
You need 630 more points
```

---

# Complete Response Shape

```json id="r9"
{
  "competition": {},
  "performance": {},
  "earnings": {},
  "rewardProjection": {},
  "rankProgress": {},
  "referrals": {},
  "recentActivities": [],
  "insights": []
}
```

---

# System-Level Business Rules

## 1. Competition Scope

* Cycle-based only
* No cross-cycle mixing
* Only qualified data included

---

## 2. Attribution Scope

* 90-day window (configurable)
* Max 5-order contribution per customer
* Excludes invalid/refunded orders

---

## 3. Financial Scope (Settlement Lifecycle)

```
Earned → Pending → Withdrawable
```

Pending exists to enforce settlement safety before payout finalization.

---

## 4. Metric Separation Rules

### Competition Score

Used only for ranking

### Competition Commission

Used only for reward calculation

### Competition GMV

Only qualified attribution + cycle-bound orders

---

# Final Architecture Model

```text id="arch1"
MongoDB
   ↓
Aggregation Service
   ↓
Attribution + Competition Engine
   ↓
Financial Settlement Layer
   ↓
Dashboard DTO Builder
   ↓
API Response
   ↓
Frontend Renderer
```

---

# Final Principle

The frontend is a **renderer**, not a calculator.

All intelligence lives in the backend.

Frontend only displays truth.