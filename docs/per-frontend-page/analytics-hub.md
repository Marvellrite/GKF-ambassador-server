# Analytics Hub API Specification (Refined)

---

# Overview

The Analytics Hub is the **time-range based performance intelligence system** for ambassadors.

It answers:

> What is driving my performance, and how is it changing over time?

Unlike:

* Competition Center (cycle + ranking)
* Leaderboard (position + comparison)

Analytics is strictly:

> trend + behavior + change detection

---

# Core Design Principles

## 1. Time-Range Based System

All analytics are computed within a bounded time window:

```http id="an1"
GET /analytics?from={date}&to={date}
```

### Optional Presets

```http id="an2"
GET /analytics/preset?range=7d|30d|monthly
```

or:

* last_7_days
* last_30_days
* this_month
* custom_range

---

## 2. No Ranking System

Analytics MUST NOT include:

* Rank
* Leaderboard position
* Reward tier
* Competition standing

It is strictly non-competitive.

```text id="an3"
Analytics = performance movement
NOT competition position
```

---

## 3. Trend-Based Rule (Core Requirement)

Every metric MUST include:

* current value
* previous value
* percentage change
* direction (implicit or explicit)

---

# API Endpoints

```http id="ep1"
GET /analytics?from=&to=

GET /analytics/preset?range=7d|30d|monthly

GET /analytics/comparison?from=&to=&compareFrom=&compareTo=
```

---

# 1. Full Analytics Overview

## Purpose

Provides a **complete performance snapshot** within a time range.

---

## Request

```http id="req1"
GET /analytics?from=2026-06-01&to=2026-06-16
```

---

## Response

```json id="res1"
{
  "range": {
    "from": "2026-06-01T00:00:00Z",
    "to": "2026-06-16T23:59:59Z"
  },

  "referrals": {
    "value": 48,
    "previousValue": 35,
    "changePercent": 37.1
  },

  "conversions": {
    "value": 22,
    "previousValue": 18,
    "changePercent": 22.2,
    "conversionRate": 45.8
  },

  "gmv": {
    "value": 1250000,
    "previousValue": 980000,
    "changePercent": 27.5
  },

  "revenue": {
    "value": 380000,
    "previousValue": 310000,
    "changePercent": 22.5
  },

  "retention": {
    "points": 220,
    "previousPoints": 180,
    "changePercent": 22.2
  }
}
```

---

## UI Display

### Header

```text id="ui1"
Analytics Overview

Jun 01 – Jun 16
```

---

### Referrals

```text id="ui2"
48 Referrals
↑ +37.1%
Previous: 35
```

---

### Conversions

```text id="ui3"
22 Conversions
↑ +22.2%
Conversion Rate: 45.8%
Previous: 18
```

---

### GMV

```text id="ui4"
₦1,250,000
↑ +27.5%
Previous: ₦980,000
```

---

### Revenue

```text id="ui5"
₦380,000
↑ +22.5%
Previous: ₦310,000
```

---

### Retention

```text id="ui6"
220 Points
↑ +22.2%
Previous: 180
```

---

# 2. Trend Analytics (Time Series)

## Purpose

Shows **behavioral movement over time**, not summaries.

---

## Response

```json id="res2"
{
  "trend": {
    "referrals": [
      { "date": "2026-06-10", "value": 5 },
      { "date": "2026-06-11", "value": 7 },
      { "date": "2026-06-12", "value": 6 }
    ],

    "conversions": [
      { "date": "2026-06-10", "value": 2 },
      { "date": "2026-06-11", "value": 3 }
    ],

    "gmv": [
      { "date": "2026-06-10", "value": 120000 },
      { "date": "2026-06-11", "value": 180000 }
    ]
  }
}
```

---

## UI Display

```text id="ui7"
Referral Trend
5 → 7 → 6 → 8 → 9

Conversion Trend
2 → 3 → 2 → 4

GMV Trend
₦120k → ₦180k → ₦160k → ₦210k
```

---

# 3. Retention Analytics

## Purpose

Tracks **customer lifecycle progression behavior**.

---

## Response

```json id="res3"
{
  "retention": {
    "order2": 4,
    "order3": 3,
    "order4": 2,
    "order5": 5,
    "retentionScoreContribution": 220
  }
}
```

---

## UI Display

```text id="ui8"
Retention Breakdown

2nd Orders → 4 customers
3rd Orders → 3 customers
4th Orders → 2 customers
5th Orders → 5 customers

Retention Score: 220 pts
```

---

# 4. Revenue Analytics

## Purpose

Tracks earnings movement over time (financial intelligence only).

---

## Response

```json id="res4"
{
  "revenue": {
    "current": 380000,
    "previous": 310000,
    "changePercent": 22.5,

    "pending": 65000,
    "withdrawable": 1185000
  }
}
```

---

## UI Display

```text id="ui9"
Revenue Performance

₦380,000
↑ +22.5%
Previous: ₦310,000

Pending
₦65,000

Withdrawable
₦1,185,000
```

---

# 5. GMV Analytics (Attribution-Based)

## Purpose

Separates:

* competition-qualified GMV
* full attributed GMV

---

## Response

```json id="res5"
{
  "gmv": {
    "competitionGMV": 1250000,
    "attributedGMV": 4300000
  }
}
```

---

## UI Display

```text id="ui10"
Competition GMV
₦1,250,000

Attributed GMV
₦4,300,000
```

---

# 6. Analytics Comparison Mode

## Purpose

Compare two time ranges.

---

## Request

```http id="req2"
GET /analytics/comparison?from=&to=&compareFrom=&compareTo=
```

---

## Response

```json id="res6"
{
  "current": {
    "referrals": 48,
    "conversions": 22,
    "gmv": 1250000,
    "revenue": 380000
  },

  "previous": {
    "referrals": 35,
    "conversions": 18,
    "gmv": 980000,
    "revenue": 310000
  },

  "delta": {
    "referrals": 13,
    "conversions": 4,
    "gmv": 270000,
    "revenue": 70000
  }
}
```

---

## UI Display

```text id="ui11"
Comparison

Referrals
35 → 48 (+13)

Conversions
18 → 22 (+4)

GMV
₦980,000 → ₦1,250,000 (+₦270,000)

Revenue
₦310,000 → ₦380,000 (+₦70,000)
```

---

# Analytics Hub UI Layout

```text id="ui12"
Analytics Hub

Date Filter
[7D] [30D] [Monthly] [Custom]

--------------------------------------

Performance Overview
Referrals
Conversions
GMV
Revenue
Retention

--------------------------------------

Trend Charts
Referral Trend
Conversion Trend
GMV Trend

--------------------------------------

Retention Analysis
Order Lifecycle Behavior

--------------------------------------

Revenue Insights
Pending vs Withdrawable

--------------------------------------

GMV Attribution
Competition GMV
Attributed GMV
```

---

# Backend Responsibilities

The backend MUST:

* Aggregate time-range metrics
* Compute previous period comparisons
* Normalize attribution rules
* Ensure consistent revenue definitions
* Generate time-series datasets
* Enforce filter correctness across all endpoints

---

# Frontend Responsibilities

The frontend MUST:

* Render only backend-provided values
* Perform no aggregation or calculation
* Use trends strictly as display data
* Use delta values as authoritative

---

# Key System Insight

Analytics is NOT:

* ranking
* competition
* leaderboard

Analytics IS:

> a behavioral microscope over time

It explains:

* why performance changed
* how performance evolved
* what is accelerating or declining

Not:

* where you stand in competition
