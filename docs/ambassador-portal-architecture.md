# GokaFoods Ambassador Portal Architecture

## Overview

The GokaFoods Ambassador Portal is a unified performance and rewards system built on three core principles:

- **Commerce Layer:** real-world earnings from referrals, conversions, and GMV.
- **Competition Layer:** seasonal gamified ranking system based on 2-week cycles.
- **Insight Layer:** analytics and optimization tools.

The system is designed as a cycle-based competitive economy where ambassadors:

```text
Compete -> Earn -> Improve -> Reset -> Compete again
```

## Core Architecture Model

### Data Processing Pipeline

All ambassador data follows a strict backend flow:

```text
Raw Data -> Computation Layer -> Response DTO
```

### Raw Data

Raw data is the source of truth and is stored permanently in the database.

Key entities include:

- Users
- Orders
- Referrals
- Transactions
- Order timestamps
- Customer attribution windows

### Computation Layer

The computation layer is the business logic engine. It is responsible for:

- Competition score calculation
- Retention scoring
- GMV aggregation within valid attribution windows
- Conversion rate calculation
- Ranking generation
- Reward eligibility

This logic should be centralized in the Competition Service layer.

### Response DTO

The backend returns structured, frontend-ready data:

- **Raw metrics:** used for transparency.
- **Computed metrics:** used for consistent business logic.
- **Display metrics:** used for UI rendering.

## Key System Rules

### 1. Attribution Window Rule

Each referred customer is linked to an ambassador for a limited attribution window:

- First 90 days, or
- First 5 orders,
- Whichever comes first.

After the attribution window expires:

- GMV is no longer tracked for that ambassador.
- Retention is no longer tracked for that ambassador.
- Analytics visibility for that customer ends for that ambassador.
- The customer becomes a system-wide independent user.

This ensures fairness and prevents long-term passive advantage.

### 2. Retention System

Retention does not modify GMV. It is a separate reward layer based on a bonus bucket model.

#### Retention Scoring

| Order Number | Points |
| --- | ---: |
| 2nd order | 10 |
| 3rd order | 15 |
| 4th order | 20 |
| 5th order | 25 |

After the 5th order, no further retention rewards are awarded.

#### Final Score Composition

```text
Total Competition Score =
Referral Score +
Conversion Score +
GMV Score +
Retention Bonus Score
```

There are no multipliers, no hidden inflation, and no opaque scoring rules.

### 3. GMV Scoring Rule

GMV is calculated strictly within the attribution window.

Example scoring:

```text
NGN 1,000 GMV = 1 point
```

Only the first 5 orders or the first 90 days count. After expiry, the GMV is excluded completely.

### 4. Competition System

#### Cycle Definition

- Duration: 2 weeks
- Rankings reset after cycle completion
- Each new cycle starts with fresh rankings

#### Rewards Distribution

| Rank | Reward |
| ---: | ---: |
| 1st | NGN 150,000 |
| 2nd | NGN 100,000 |
| 3rd | NGN 70,000 |
| 4th | NGN 50,000 |
| 5th | NGN 30,000 |

#### Boost Tiers

| Rank Range | Boost |
| --- | --- |
| 6-11 | 10% commission boost for the next cycle only |
| 12-50 | 5% commission boost for the next cycle only |

### 5. Earnings Multiplier Rule

- Boosts apply only to the next cycle.
- Boosts are not permanent.
- Boosts do not stack infinitely.
- Each cycle recalculates from scratch.

## Portal Structure

### 1. Ambassador Dashboard

The dashboard provides a single performance snapshot.

Displays:

- Current competition rank
- Live competition score
- Cycle-based referrals
- Conversions
- GMV within valid attribution windows
- Retention bonus
- Earnings summary
- Active commission boost
- Time left in the current cycle
- Progress to next rank

Backend endpoint:

```http
GET /ambassador/dashboard
```

### 2. Competition Center

The Competition Center is the core gamification engine.

#### Live Competition

- Real-time leaderboard
- User rank highlight
- Prize tiers
- Boost tiers
- Cycle countdown

#### Competition History

- Past cycle rankings
- Score snapshots
- Reward history

#### Competition Detail

Shows a full breakdown of a selected cycle:

- Referral score
- Conversion score
- GMV score
- Retention bonus
- Final rank

#### Comparison View

Compares performance across cycles.

| Metric | Cycle N-2 | Cycle N-1 | Current | Change |
| --- | ---: | ---: | ---: | ---: |
| Rank | 5 | 3 | 2 | +3 |
| Score | 12,400 | 14,100 | 15,600 | +26% |
| GMV | NGN 380k | NGN 460k | NGN 510k | +34% |

Backend endpoints:

```http
GET /competition/current
GET /competition/history
GET /competition/:id
GET /competition/compare
```

### 3. Leaderboard System

#### Public Leaderboard

The public leaderboard is used for marketing visibility.

Includes:

- Top ambassadors only
- Rank
- Total competition score
- Basic stats

Does not include:

- Full scoring breakdown
- Earnings visibility

#### Internal Leaderboard

The internal portal leaderboard includes:

- Complete ranking list
- Pagination
- Current user highlight
- Cycle filters

Backend endpoints:

```http
GET /leaderboard/public
GET /leaderboard/internal?competitionId=
```

### 4. Analytics Hub

The Analytics Hub is the performance optimization layer.

Metrics:

- Referral trends
- Conversion efficiency
- GMV trends
- Retention behavior
- Earnings trends
- Cycle performance comparison

Time filters:

- Last 7 days
- Last 30 days
- Monthly views
- Custom range

Backend endpoint:

```http
GET /analytics?from=&to=
```

### 5. Referral Management

Features:

- Referral link generator
- QR code
- Referred users list
- Conversion tracking per user
- Referral status tracking

Referral statuses:

- Joined
- Ordered
- Inactive

Backend endpoints:

```http
GET /referrals
GET /referrals/list
```

### 6. Wallet and Earnings

The Wallet and Earnings section provides financial tracking.

Includes:

- Total earnings
- Pending earnings
- Withdrawable balance
- Transaction history
- Withdrawal history
- Commission breakdown

Backend endpoints:

```http
GET /wallet
GET /wallet/transactions
GET /wallet/withdrawals
```

### 7. Ambassador Levels System

The Ambassador Levels system handles long-term progression outside competitions.

Includes:

- Current level
- Upgrade requirements
- Commission rate changes
- Discount eligibility
- Order limit benefits

Backend endpoint:

```http
GET /ambassador/levels
```

### 8. Settings and Profile

Features:

- Profile management
- Referral code
- Payment details
- Notifications
- Security settings

Backend endpoint:

```http
GET /ambassador/settings
```

## Backend Architecture

### Core Services

#### 1. Competition Service

The Competition Service is the central business logic service.

Handles:

- Score calculation
- Ranking generation
- Cycle management
- Reward distribution

#### 2. Metrics Service

Handles:

- Raw data aggregation
- GMV tracking
- Attribution logic

#### 3. Rewards Service

Handles:

- Cash payouts
- Commission boosts
- Cycle reward distribution

#### 4. Referral Service

Handles:

- Referral tracking
- Attribution mapping

#### 5. Wallet Service

Handles:

- Earnings ledger
- Withdrawals
- Payouts

## System Philosophy

The system is designed as a fair seasonal economy.

### Transparency

All scoring logic is explainable and visible.

### Competition

Every 2 weeks is a fresh competitive reset.

### Fairness

Attribution windows prevent long-term advantage accumulation.

### Repeat Engagement

The boost system encourages continuous motivation across cycles.
