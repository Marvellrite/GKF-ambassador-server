# GokaFoods Ambassador Portal Architecture

# Overview

The GokaFoods Ambassador Portal is a unified performance, competition, analytics, and financial management ecosystem built around a recurring monthly growth economy.

The system is built on four foundational layers:

* **Commerce Layer:** Earnings generated from referrals, conversions, and attributed GMV.
* **Competition Layer:** Monthly ranking system that rewards top-performing ambassadors.
* **Insight Layer:** Analytics and optimization tools that help ambassadors improve performance.
* **Finance Layer:** Ambassador wallet management, bank withdrawals, and internal transfers.

The platform is designed as a repeatable engagement cycle.

```text
Compete → Earn → Improve → Reward → Reset → Repeat
```

---

# Core Principles

The system intentionally separates three concepts:

```text
Competition Score → Determines Rank

Rank → Determines Reward Percentage

Cycle Revenue → Determines Reward Amount
```

These are independent systems working together.

---

# Core Architecture Model

## Data Processing Pipeline

All ambassador data follows a strict processing pipeline.

```text
Raw Data
    ↓
Computation Layer
    ↓
Response DTO
    ↓
UI Rendering
```

---

# Raw Data Layer

The Raw Data Layer is the system source of truth.

Entities include:

* Users
* Orders
* Referrals
* Transactions
* Wallet Transactions
* Competition Cycles
* Attribution Windows
* Customer Events
* Withdrawal Records

---

# Computation Layer

The Computation Layer is the business engine.

Responsibilities include:

* Competition score calculation
* Rank generation
* Reward calculation
* Revenue aggregation
* GMV aggregation
* Retention scoring
* Conversion calculations
* Attribution validation
* Wallet balance computation
* Reward projection generation

This logic is centralized within the Competition Service.

---

# Response DTO Layer

Responses are grouped into three categories.

## Raw Metrics

Used for transparency.

Examples:

* Referrals
* Revenue
* Orders
* Wallet balances

---

## Computed Metrics

Used for business logic.

Examples:

* Competition score
* Rank
* Conversion rate
* Retention score
* Reward eligibility

---

## Projection Metrics

Used for user guidance.

Examples:

* Estimated reward
* Rank movement
* Progress to next rank
* Growth opportunities

---

# Key System Rules

# 1. Attribution Window Rule

Every referred customer belongs to an ambassador for a limited period.

The attribution window expires after:

* 90 days
* OR 5 orders

Whichever occurs first.

After expiry:

* GMV attribution stops
* Retention scoring stops
* Analytics visibility ends
* Customer becomes platform-owned

This prevents permanent customer ownership.

---

# 2. Retention System

Retention is a separate bonus scoring system.

Retention does not modify GMV.

Only orders 2–5 qualify.

| Order | Points |
| ----: | -----: |
|     2 |     10 |
|     3 |     15 |
|     4 |     20 |
|     5 |     25 |

After the fifth order:

```text
No additional retention points are awarded.
```

---

# 3. Competition Score Formula

```text
Competition Score =
Referral Score +
Conversion Score +
GMV Score +
Retention Score
```

No hidden multipliers exist.

Everything is transparent.

---

# 4. GMV Rule

Only attributed orders contribute to GMV.

Example:

```text
NGN 1,000 GMV = 1 Point
```

GMV tracking ends after:

* 90 days
* OR 5 orders

Whichever comes first.

---

# Monthly Competition System

## Competition Cycle

Competition operates on a monthly basis.

Rules:

* Duration: 1 calendar month
* Rankings reset every month
* Historical records remain available

Competition lifecycle:

```text
Month Start
      ↓
Competition Active
      ↓
Leaderboard Locked
      ↓
Rewards Distributed
      ↓
Monthly Reset
```

---

# Monthly Reward System

# Reward Philosophy

Rewards are dynamic.

There are no fixed prize pools.

Top performers receive a percentage increase on the revenue they generated during the competition cycle.

---

# Reward Formula

```text
Competition Reward =
Cycle Revenue × Rank Percentage
```

---

# Reward Distribution

| Rank | Percentage |
| ---: | ---------: |
|    1 |        10% |
|    2 |         8% |
|    3 |         6% |
|    4 |         4% |
|    5 |         2% |

Only the Top 5 ambassadors receive rewards.

---

# Revenue Definition

```text
Cycle Revenue =
All eligible earnings generated during the current competition month
```

Eligible revenue includes:

* Referral commissions
* Conversion commissions
* GMV commissions
* Approved ambassador incentives

Excluded:

* Previous cycle earnings
* Previous competition rewards
* Historical wallet balances
* Manual adjustments

---

# Reward Projection System

Reward projection is dynamic.

It should never be presented as a guaranteed reward.

Formula:

```text
Estimated Reward =
Cycle Revenue × Reward Percentage(Current Rank)
```

Rewards may increase or decrease throughout the month.

---

# Financial System Boundaries

Three systems exist.

```text
Ambassador Wallet System (This Project)
                 |
       ---------------------
       |                   |
       ↓                   ↓
Personal Bank      GKF Consumer Wallet
(External)         (Separate Project)
```

---

# Wallet Ownership Rules

# Ambassador Wallet

The Ambassador Wallet is an earnings wallet.

Funds belong to the ambassador.

Sources include:

* Referral commissions
* Conversion commissions
* GMV commissions
* Competition rewards
* Approved incentives

Ambassadors can:

* View balances
* Withdraw to personal bank accounts
* Transfer funds to a GKF Consumer Wallet

Ambassadors cannot:

* Transfer to other ambassadors
* Modify balances directly
* Reverse completed withdrawals

---

# GKF Consumer Wallet

The Consumer Wallet is a separate system.

This project only supports internal transfers into it.

The Consumer Wallet:

* Cannot withdraw to personal banks
* Cannot transfer back to Ambassador Wallets

Its purpose is platform consumption.

Flow:

```text
Ambassador Wallet
        ↓
Consumer Wallet
        ↓
Food Orders
```

No reverse flow exists.

```text
Consumer Wallet
       X
Personal Bank
```

---

# Portal Structure

# 1. Ambassador Dashboard

Purpose:

Provide a single performance snapshot.

Endpoint:

```http
GET /ambassador/dashboard
```

Displays:

## Competition

* Current rank
* Competition score
* Rank movement
* Time remaining

## Performance Metrics

* Monthly referrals
* Monthly conversions
* Conversion rate
* Monthly GMV
* Retention bonus

## Revenue

* Revenue generated this cycle
* Wallet balance
* Pending earnings

## Reward Projection

* Estimated reward
* Current reward percentage
* Reward stability

## Growth

* Progress to next rank
* Growth opportunities
* Customer health indicators

---

# 2. Competition Center

Purpose:

Competition management and history.

Displays:

## Live Competition

* Leaderboard
* Current rank
* Competition score
* Revenue generated
* Reward percentages
* Estimated rewards
* Countdown

## Competition History

* Historical rankings
* Historical rewards
* Revenue performance

## Competition Details

* Referral score
* Conversion score
* GMV score
* Retention score
* Revenue generated
* Final rank
* Reward earned

## Competition Comparison

| Metric  | Previous Month | Current Month |
| ------- | -------------: | ------------: |
| Rank    |              5 |             2 |
| Score   |         14,100 |        15,600 |
| Revenue |       NGN 460k |      NGN 510k |

Endpoints:

```http
GET /competition/current

GET /competition/history

GET /competition/:id

GET /competition/compare
```

---

# 3. Leaderboard System

## Public Leaderboard

Displays:

* Top ambassadors
* Rank
* Competition score

Does not display:

* Earnings
* Revenue breakdown
* Wallet information

---

## Internal Leaderboard

Displays:

* Full rankings
* User highlight
* Pagination
* Monthly filters

Endpoints:

```http
GET /leaderboard/public

GET /leaderboard/internal?competitionId=
```

---

# 4. Analytics Hub

Purpose:

Performance optimization.

Displays:

* Referral trends
* Conversion trends
* GMV trends
* Revenue trends
* Retention trends
* Competition comparisons

Filters:

* Last 7 days
* Last 30 days
* Monthly
* Custom range

Endpoint:

```http
GET /analytics?from=&to=
```

---

# 5. Referral Management

Features:

* Referral link generation
* QR codes
* Referral tracking
* Conversion tracking

Referral statuses:

* Joined
* Ordered
* Inactive

Endpoints:

```http
GET /referrals

GET /referrals/list
```

---

# 6. Wallet & Financial Management

Purpose:

Manage ambassador finances.

Features:

## Wallet Summary

* Available balance
* Pending earnings
* Withdrawable balance
* Revenue this cycle

## Transfers

* Withdraw to personal bank account
* Transfer to GKF Consumer Wallet

## History

* Transaction history
* Withdrawal history
* Competition reward history

Endpoints:

```http
GET /wallet

GET /wallet/transactions

GET /wallet/withdrawals

GET /wallet/transfers
```

---

## Withdraw To Bank

```http
POST /wallet/withdraw
```

Request:

```json
{
  "amount": 50000,
  "bankAccountId": "bank_123"
}
```

---

## Transfer To Consumer Wallet

```http
POST /wallet/transfer-to-consumer-wallet
```

Request:

```json
{
  "amount": 10000
}
```

---

# 7. Ambassador Levels

Long-term progression system independent of competitions.

Displays:

* Current level
* Upgrade requirements
* Commission rates
* Benefits

Endpoint:

```http
GET /ambassador/levels
```

---

# 8. Settings & Profile

Features:

* Profile management
* Payment details
* Notifications
* Security settings

Endpoint:

```http
GET /ambassador/settings
```

---

# Backend Architecture

## Competition Service

Handles:

* Competition scores
* Rankings
* Reward calculations
* Monthly cycles

---

## Metrics Service

Handles:

* Aggregations
* Attribution validation
* GMV calculations

---

## Rewards Service

Handles:

* Competition rewards
* Wallet crediting
* Reward settlements

---

## Referral Service

Handles:

* Referral tracking
* Attribution management

---

## Wallet Service

Handles:

* Earnings ledger
* Bank withdrawals
* Consumer wallet transfers
* Transaction history

---

# System Philosophy

## Transparency

Every score, reward, and calculation is explainable.

## Fairness

No permanent customer ownership exists.

## Sustainability

Rewards scale with actual value generated.

## Separation of Concerns

Ambassador finances and consumer finances remain separate systems.

## Competition

Every month is a fresh opportunity to compete.

## Continuous Engagement

Monthly resets keep ambassadors active and motivated.
