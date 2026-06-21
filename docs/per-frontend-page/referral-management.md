# Referral Management API Specification (Refined + Production Grade)

---

# Overview

The Referral Management system is the **relationship + attribution tracking layer** of the ambassador platform.

It answers:

> Who did I bring in, what did they do, and how far did they progress?

It tracks:

* referral generation
* referral lifecycle state
* conversion progression
* ordering behavior
* referral effectiveness

It is NOT:

* a competition system
* a time-series analytics system
* a wallet or earnings system

---

# Core Design Principle

## Referrals are STATE-BASED, not TIME-BASED

Unlike Analytics:

* Referrals are not primarily queried by time range
* They are queried by **relationship state + lifecycle filters**

However:

> Time filtering is OPTIONAL, not foundational

---

# Referral Lifecycle States

Each referral transitions through a lifecycle:

## States

| State    | Meaning                              |
| -------- | ------------------------------------- |
| Joined   | User signed up via referral link     |
| Ordered  | User completed at least one order    |
| Inactive | No meaningful activity after joining |

---

# Key System Concepts

## 1. Referral Identity Model

Each referral contains:

* referred user
* ambassador owner
* referral timestamp
* conversion status
* order history summary

---

## 2. Conversion Definition

A referral is considered:

> Converted when the referred user places their first successful order

---

## 3. Referral Effectiveness

Computed metrics:

* conversion rate
* order activation rate
* inactivity ratio

(All computed backend-side only)

---

# API OVERVIEW

```http
GET /referrals
GET /referrals/list
GET /referrals/stats
GET /referrals/link
GET /referrals/qr
```

---

# 1. Referral Dashboard Overview

## Endpoint

```http
GET /referrals
```

---

## Purpose

Returns high-level referral performance summary.

This is the **top-level referral intelligence card**.

---

## Request

```http
GET /referrals
```

---

## Response

```json
{
  "summary": {
    "totalReferrals": 128,
    "joined": 128,
    "ordered": 54,
    "inactive": 74,

    "conversionRate": 42.1
  },

  "link": {
    "referralCode": "AMB_10291XZ",
    "referralUrl": "https://app.foodplatform.com/r/AMB_10291XZ"
  },

  "qrCode": {
    "url": "https://cdn.foodplatform.com/qrcodes/AMB_10291XZ.png"
  }
}
```

---

## UI PLACEMENT

### Referral Header Card

```text
My Referral Network
128 Total Referrals
42.1% Conversion Rate
```

---

### Lifecycle Breakdown Card

```text
Joined → 128
Ordered → 54
Inactive → 74
```

---

### Referral Link Card

```text
Your Referral Link
https://app.foodplatform.com/r/AMB_10291XZ
```

---

### QR Code Card

Display QR image from `qrCode.url`

---

# 2. Referral List (Core Endpoint)

## Endpoint

```http
GET /referrals/list
```

---

## Purpose

Returns paginated referral entities with filtering.

---

## Query Parameters

| Param  | Type   | Description     |         |          |
| ------ | ------ | ----------------- | ------- | -------- |
| page   | number | pagination       |         |          |
| limit  | number | page size         |         |          |
| status | string | joined            | ordered | inactive |
| search | string | user name/email  |         |          |
| sort   | string | newest            | oldest  |          |

---

## Request

```http
GET /referrals/list?status=ordered&page=1&limit=20
```

---

## Response

```json
{
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 54
  },

  "data": [
    {
      "userId": "usr_001",
      "name": "Amina Yusuf",
      "status": "ordered",
      "joinedAt": "2026-06-10T12:00:00Z",
      "firstOrderAt": "2026-06-12T15:22:00Z",
      "ordersCount": 3
    }
  ]
}
```

---

## UI PLACEMENT

### Filter Tabs

```text
All | Joined | Ordered | Inactive
```

---

### Referral Row

```text
Amina Yusuf
Status: Ordered
Orders: 3
Joined: Jun 10
First Order: Jun 12
```

---

# 3. Referral Effectiveness Statistics (new)

## Endpoint

```http
GET /referrals/stats
```

---

## Purpose

Returns the deeper **effectiveness ratios** referenced under "Referral Effectiveness" above — distinct from `/referrals`, which only returns raw lifecycle counts. This is the endpoint that powers any "how good is my network, not just how big" insight on the referral page.

---

## Optional Query Parameters

| Param | Type   | Description                         |
| ------ | ------ | -------------------------------------- |
| from   | date   | optional filter — start of range      |
| to     | date   | optional filter — end of range        |

> Time filtering here follows the same rule as `/referrals/list`: it's an optional overlay, never the primary dimension.

---

## Request

```http
GET /referrals/stats
```

---

## Response

```json
{
  "effectiveness": {
    "conversionRate": 42.1,
    "repeatOrderRate": 38.9,
    "inactivityRate": 57.8
  },

  "ordersPerConvertedReferral": {
    "average": 2.4,
    "median": 2
  },

  "averageDaysToFirstOrder": 2.1
}
```

---

## UI PLACEMENT

### Effectiveness Card

```text
Conversion Rate
42.1%

Repeat Order Rate
38.9%

Inactivity Rate
57.8%
```

---

### Behavior Insight Card

```text
Avg Orders per Converted Referral
2.4

Avg Days to First Order
2.1 days
```

---

# 4. Referral Link Generation

## Endpoint

```http
GET /referrals/link
```

---

## Purpose

Returns ambassador referral identity.

---

## Response

```json
{
  "referralCode": "AMB_10291XZ",
  "referralUrl": "https://app.foodplatform.com/r/AMB_10291XZ"
}
```

---

## UI

```text
Your Referral Code
AMB_10291XZ

Copy Link
Share QR
```

---

# 5. QR Code Generation

## Endpoint

```http
GET /referrals/qr
```

---

## Response

```json
{
  "qrCodeUrl": "https://cdn.foodplatform.com/qrcodes/AMB_10291XZ.png"
}
```

---

## UI

* Display QR image
* Download button
* Share button

---

# TIME-WINDOW DESIGN CLARIFICATION

## Are referrals time-window aware?

### ✔ YES — BUT ONLY AS FILTER, NOT CORE DIMENSION

Referrals support optional time filtering:

```http
GET /referrals/list?from=2026-06-01&to=2026-06-30
```

---

## BUT:

Referrals are NOT inherently time-based like analytics.

### Difference:

| System      | Time Role             |
| ------------- | ------------------------ |
| Analytics   | Primary dimension     |
| Competition | Cycle-bound dimension |
| Referrals   | Optional filter only  |

---

## WHY THIS MATTERS

If you make referrals time-primary:

* you lose lifecycle clarity
* you break user relationship view
* you fragment referral identity

So instead:

> Referrals = relationship graph
> Time = filter overlay

---

# BACKEND RESPONSIBILITIES

Backend MUST:

* track referral relationships
* compute lifecycle states
* compute conversion status
* compute referral statistics
* generate QR + referral links
* optionally filter by time window
* ensure state consistency

---

# FRONTEND RESPONSIBILITIES

Frontend MUST:

* render referral states only
* NOT compute conversion rates
* NOT derive lifecycle transitions
* NOT aggregate referrals
* treat backend as source of truth

---

# FINAL SYSTEM INSIGHT

Referral Management is:

> a relationship lifecycle engine

NOT:

* a timeline system
* a financial system
* a ranking system

It is the **social graph layer** beneath analytics and competition.

It answers:

* who came in through you
* what they did
* how far they progressed
* how effective your network is