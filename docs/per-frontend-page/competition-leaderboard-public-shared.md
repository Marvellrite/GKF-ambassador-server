# Competition Leaderboard API Specification (Public + My Position View)

---

# Overview

The Competition Leaderboard is the **public-facing ranking system** for the active competition cycle.

It is designed to:

* Promote transparency
* Encourage competition
* Provide real-time ranking visibility
* Allow ambassadors to locate themselves instantly

---

## Critical Principle

This leaderboard is intentionally **non-financial**.

It must never expose:

* Earnings
* Commission
* Wallet balances
* Rewards
* Revenue data

It is purely a **performance ranking system**.

---

# Endpoints

```http id="lb1"
GET /competition/current/leaderboard?page=1&limit=20

GET /competition/current/leaderboard/me
```

---

# 1. Global Leaderboard (Paginated)

## Purpose

Provides a ranked list of ambassadors for the current competition cycle.

Used for:

* Exploration
* Transparency
* Motivation

---

## Request

```http id="req1"
GET /competition/current/leaderboard?page=1&limit=20
```

### Query Parameters

| Field | Type   | Description                            |
| ----- | ------ | -------------------------------------- |
| page  | number | Page number (default: 1)               |
| limit | number | Items per page (default: 20, max: 100) |

---

## Response

```json id="res1"
{
  "competition": {
    "status": "LIVE",

    "cycle": {
      "startsAt": "2026-06-01T00:00:00Z",
      "endsAt": "2026-06-30T23:59:59Z",
      "daysRemaining": 12
    },

    "totalParticipants": 347
  },

  "leaderboard": [
    {
      "rank": 1,
      "ambassadorId": "amb_001",
      "displayName": "User A",
      "level": "Campus Partner",
      "competitionScore": 16890
    },
    {
      "rank": 2,
      "ambassadorId": "amb_002",
      "displayName": "User B",
      "level": "Community Partner",
      "competitionScore": 16300
    }
  ],

  "pagination": {
    "page": 1,
    "limit": 20,
    "totalPages": 18,
    "totalItems": 347
  },

  "me": {
    "rank": 42,
    "isOnCurrentPage": false
  }
}
```

---

## UI Display

### Competition Header

```text id="ui1"
Current Competition (LIVE)

Cycle
Jun 01 – Jun 30

Days Remaining
12

Total Participants
347
```

---

### Leaderboard List

```text id="ui2"
#1 User A
Level: Campus Partner
16,890 pts

#2 User B
Level: Community Partner
16,300 pts
```

---

### Pagination UI

```text id="ui3"
Previous

1 2 3 4 ... 18

Next
```

---

### My Position Indicator

```text id="ui4"
Your Position
Rank #42
```

If the authenticated user is not on the current page:

```text id="ui5"
Jump to My Position
```

---

# 2. My Position Endpoint (Critical UX Feature)

## Purpose

Allows an ambassador to instantly locate themselves in the leaderboard without scrolling or pagination navigation.

Includes surrounding competitors for context.

---

## Request

```http id="req2"
GET /competition/current/leaderboard/me
```

---

## Response

```json id="res2"
{
  "me": {
    "rank": 42,
    "displayName": "You",
    "level": "Campus Partner",
    "competitionScore": 15600
  },

  "surrounding": [
    {
      "rank": 40,
      "displayName": "User X",
      "level": "Campus Partner",
      "competitionScore": 15880
    },
    {
      "rank": 41,
      "displayName": "User Y",
      "level": "Community Partner",
      "competitionScore": 15720
    },
    {
      "rank": 42,
      "displayName": "You",
      "level": "Campus Partner",
      "competitionScore": 15600,
      "me": true
    },
    {
      "rank": 43,
      "displayName": "User Z",
      "level": "Campus Partner",
      "competitionScore": 15540
    }
  ]
}
```

---

## UI Display

```text id="ui6"
Nearby Rankings

#40 User X
15,880 pts

#41 User Y
15,720 pts

#42 You
15,600 pts

#43 User Z
15,540 pts
```

---

# Data Exposure Rules (Strict)

## Allowed Fields

The leaderboard MAY expose only:

* Rank
* Display Name
* Ambassador Level
* Competition Score

---

## Forbidden Fields

The leaderboard MUST NEVER expose:

* Competition Commission
* Estimated Rewards
* Wallet Balance
* Earnings
* Revenue breakdown
* Historical financial data

---

# Ambassador Level Rule

Ambassador level is included because it directly affects:

* Reward percentage tiers (backend logic)
* Competition weighting logic (optional future enhancement)
* Ranking interpretation context

However:

> It must NOT be used as a visible financial projection tool in the public leaderboard.

---

# Pagination Rules

* Required for global leaderboard
* Default sorting: rank ascending
* No filtering required for MVP
* Designed for scalability (10k+ users ready)

---

# My Position UX Rule (Important)

Frontend should always prioritize:

```text
Show me my rank immediately
```

Backend ensures:

* Instant rank lookup
* Surrounding context (+/- 2 or more)
* No need for pagination navigation to find user

---

# Backend Responsibilities

The backend is responsible for:

* Ranking computation
* Pagination logic
* Cycle-based filtering
* Returning authenticated user position
* Returning surrounding context
* Ensuring no financial leakage
* Enforcing leaderboard data privacy rules

---

# Final Architecture Flow

```text id="arch1"
MongoDB
   ↓
Competition Ranking Engine
   ↓
Leaderboard Service
   ↓
Pagination + Rank Resolver
   ↓
My Position Resolver
   ↓
API Response DTO
   ↓
Frontend Renderer
```

---

# Final Principle

The leaderboard is not a financial dashboard.

It is a **visibility and motivation system**.

It shows:

* Where you are
* Who is ahead
* Who is behind

Nothing more.
