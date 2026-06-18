# Analytics Hub API Specification (Refined + Multi-Temporal Architecture)

---

# Overview

The Analytics Hub is the **time-range based behavioral intelligence system** for ambassadors.

It explains:

> How performance is evolving, what is driving it, and how it changes over time.

It is strictly separated from:

* Competition Center (cycle ranking system)
* Leaderboard (relative position system)
* Wallet (financial state system)

Analytics is purely:

> behavioral + temporal + attribution-driven insights

---

# Core Architectural Principle (Critical Update)

## The Analytics Page is INHERENTLY MULTI-TEMPORAL

Unlike other modules, Analytics does NOT represent a single time window.

It may simultaneously contain:

* primary analysis window
* comparison window
* trend window
* retention lifecycle window
* GMV attribution window

### Therefore:

> ❌ There is NO single “page time range truth”

---

## UI Implication (Important)

The analytics header MUST NOT display a date range.

Instead:

```text
Analytics Overview
```

Because:

> The page contains multiple independent temporal contexts.

---

# Core Design Principles

## 1. Time-Range First System (Data-Level Only)

All analytics are computed using explicit ranges.

```http
GET /analytics?range=last_7_days
GET /analytics?range=last_30_days
GET /analytics?range=this_month
GET /analytics?from=2026-06-01&to=2026-06-30
```

### Rules

* Either `range` OR (`from` + `to`)
* Never both
* Invalid combinations → `400 Bad Request`

---

## 2. No Ranking Principle

Analytics MUST NOT include:

* rank
* leaderboard position
* competition standing
* reward tiers
* estimated winnings

Analytics is observational, not competitive.

---

## 3. Backend-Owned Computation Rule

Frontend must NEVER compute:

* deltas
* percentages
* comparisons
* trends
* aggregation
* GMV calculations
* retention scoring

All values are precomputed DTOs.

---

## 4. Standard Metric Contract

All scalar metrics follow:

```ts
interface AnalyticsMetric {
  value: number;
  previousValue?: number;
  delta?: number;
  changePercent?: number;
}
```

---

# 5. Key Terminology Clarification

---

## 5.1 Qualified Competition GMV

```text
qualifiedCompetitionGMV =
    GMV generated within competition cycle
    AND
    satisfying attribution rules
```

### Attribution Rules:

* 90-day attribution window
* max 5 orders per customer
* excludes refunds/reversals
* must be properly attributed

---

### Distinction

| Metric                  | Meaning                          |
| ----------------------- | -------------------------------- |
| qualifiedCompetitionGMV | cycle + attribution filtered GMV |
| lifetimeAttributedGMV   | all-time attribution GMV         |

---

## 5.2 Competition Commission

Represents:

> Earnings generated within a competition cycle.

NOT wallet balance.

NOT withdrawable funds.

---

## 5.3 Comparison Model

There are TWO layers:

### A. Implicit Comparison (default)

Each metric includes:

* value
* previousValue
* delta
* changePercent

Compares:

> current range vs previous equivalent range

---

### B. Explicit Comparison (separate endpoint)

Two independent windows compared via `/analytics/comparison`.

---

## 5.4 Multi-Temporal Page Model (NEW)

Every analytics page contains multiple time contexts:

| Context                | Purpose              |
| ---------------------- | -------------------- |
| Primary Range          | main dataset         |
| Comparison Range       | delta analysis       |
| Trend Range            | time series          |
| Retention Window       | lifecycle analysis   |
| GMV Attribution Window | revenue intelligence |

---

# API DESIGN OVERVIEW

## Endpoints

```http
GET /analytics
GET /analytics/trends
GET /analytics/comparison
```

---

# 1. Analytics Overview

## Purpose

Returns performance metrics for a selected time range.

Supports partial metric selection via `include`.

---

## Request

```http
GET /analytics?range=last_30_days&include=all&compare=true
```

---

## Response (NO HEADER DATE USED)

```json
{
  "meta": {
    "primaryRange": {
      "from": "2026-06-01T00:00:00Z",
      "to": "2026-06-30T23:59:59Z"
    },

    "comparisonRange": {
      "enabled": true,
      "from": "2026-05-02T00:00:00Z",
      "to": "2026-05-31T23:59:59Z"
    },

    "generatedAt": "2026-06-30T20:15:11Z"
  },

  "data": {
    "referrals": {
      "value": 48,
      "previousValue": 35,
      "delta": 13,
      "changePercent": 37.1
    },

    "conversions": {
      "value": 22,
      "previousValue": 18,
      "delta": 4,
      "changePercent": 22.2,
      "conversionRate": 45.8
    },

    "qualifiedCompetitionGMV": {
      "value": 1250000,
      "previousValue": 980000,
      "delta": 270000,
      "changePercent": 27.5
    },

    "competitionCommission": {
      "value": 380000,
      "previousValue": 310000,
      "delta": 70000,
      "changePercent": 22.5
    },

    "retention": {
      "value": 220,
      "previousValue": 180,
      "delta": 40,
      "changePercent": 22.2
    }
  }
}
```

---

## UI DISPLAY RULES

### Header (IMPORTANT CHANGE)

```text
Analytics Overview
```

(No date shown)

---

### Primary Range Indicator

```text
Primary Period
Jun 01 – Jun 30
```

---

### Comparison Indicator (if enabled)

```text
Compared With
May 01 – May 31
```

---

### Metrics UI

```text
48 Referrals
↑ +37.1%
Previous: 35
```

```text
22 Conversions
45.8% Conversion Rate
↑ +22.2%
```

```text
₦1,250,000 GMV
↑ +27.5%
```

```text
₦380,000 Commission
↑ +22.5%
```

```text
220 Retention Points
↑ +22.2%
```

---

# 2. Trend Analytics

## Purpose

Time-series visualization only.

---

## Request

```http
GET /analytics/trends?range=last_30_days&metric=all&granularity=daily
```

---

## Response

```json
{
  "meta": {
    "granularity": "daily"
  },

  "trend": {
    "referrals": [
      { "date": "2026-06-10", "value": 5 }
    ],

    "conversions": [
      { "date": "2026-06-10", "value": 2 }
    ],

    "qualifiedCompetitionGMV": [
      { "date": "2026-06-10", "value": 120000 }
    ],

    "competitionCommission": [
      { "date": "2026-06-10", "value": 32000 }
    ]
  }
}
```

---

## UI

```text
Referral Trend → chart
Conversion Trend → chart
GMV Trend → chart
Commission Trend → chart
```

---

# 3. Retention Analytics

## IMPORTANT FIX: Unified Structure

Retention ALWAYS returns full structure.

### Request

```http
GET /analytics?range=last_30_days&include=retention
```

---

### Response

```json
{
  "retention": {
    "score": 220,

    "breakdown": {
      "secondOrder": {
        "customers": 4,
        "qualifiedOrders": 4
      },
      "thirdOrder": {
        "customers": 3,
        "qualifiedOrders": 3
      },
      "fourthOrder": {
        "customers": 2,
        "qualifiedOrders": 2
      },
      "fifthOrder": {
        "customers": 5,
        "qualifiedOrders": 5
      }
    }
  }
}
```

---

## UI

```text
Retention Progress

2nd Orders → 4 Customers
3rd Orders → 3 Customers
4th Orders → 2 Customers
5th Orders → 5 Customers

Retention Score: 220 Points
```

---

# 4. Qualified GMV Breakdown

## Request

```http
GET /analytics?include=qualifiedGMV
```

---

## Response

```json
{
  "qualifiedCompetitionGMV": 1250000,
  "lifetimeAttributedGMV": 4300000
}
```

---

# 5. Explicit Comparison API

## Purpose

Compares two independent time windows.

---

## Request

```http
GET /analytics/comparison
?from=2026-06-01&to=2026-06-30
&compareFrom=2026-05-01&compareTo=2026-05-31
```

---

## Response

```json
{
  "current": {
    "referrals": 48,
    "conversions": 22,
    "qualifiedCompetitionGMV": 1250000,
    "competitionCommission": 380000
  },

  "previous": {
    "referrals": 35,
    "conversions": 18,
    "qualifiedCompetitionGMV": 980000,
    "competitionCommission": 310000
  },

  "delta": {
    "referrals": 13,
    "conversions": 4,
    "qualifiedCompetitionGMV": 270000,
    "competitionCommission": 70000
  }
}
```

---

# Backend Responsibilities

* Compute all metrics
* Apply attribution rules
* Generate comparison datasets
* Generate trends
* Normalize DTO structure
* Enforce consistency across include modes

---

# Frontend Responsibilities

* Render only backend data
* No computations
* No assumptions about time windows
* Use meta ranges explicitly for labeling

---

# FINAL SYSTEM INSIGHT

Analytics is NOT:

* a single timeline dashboard
* a competition tool
* a wallet system

Analytics IS:

> a multi-temporal behavioral intelligence engine

It simultaneously analyzes:

* primary performance window
* comparative performance window
* temporal trends
* lifecycle retention
* attribution-based GMV

There is NO single time truth per page.

Only structured temporal contexts.
