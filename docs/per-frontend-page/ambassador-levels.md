# Ambassador Levels API Specification (Refined)

## Overview

The Ambassador Levels module is the ambassador's permanent progression system.

Unlike the monthly competition, ambassador levels **never reset**.

Levels reward long-term contribution to the GokaFoods ecosystem and determine the financial and operational benefits available to an ambassador.

These include:

* Commission rates
* Customer discount benefits
* Discounted order allowances
* Platform privileges
* Feature unlocks
* Long-term recognition

The module should answer the following questions:

* What level am I currently?
* Why am I at this level?
* How close am I to the next level?
* What benefits do I currently enjoy?
* What benefits will I unlock next?
* What actions should I take to level up?

---

# Level Hierarchy

| Level                      | Description              |
| --------------------------- | ------------------------ |
| Campus / Community Partner | Entry ambassador level   |
| City Partner               | Intermediate level       |
| State Partner              | Advanced regional level  |
| National Partner           | National ambassador      |
| Global Partner             | Highest ambassador level |

---

# Commission & Customer Benefit Structure

The platform commission pool is fixed at:

```text
15%
```

Each ambassador level receives a percentage share of this pool.

Customer discounts are calculated as:

```text
Customer Discount = Ambassador Commission ÷ 4
```

The backend must use the exact calculated values.

| Level                      | Share of 15% Pool | Ambassador Commission | Customer Discount | Discounted Orders |
| --------------------------- | ----------------- | ---------------------- | ------------------ | ------------------ |
| Campus / Community Partner | 5% of 15%         | 0.75%                  | 0.1875%             | 5                   |
| City Partner               | 7% of 15%         | 1.05%                  | 0.2625%             | 7                   |
| State Partner              | 10% of 15%        | 1.50%                  | 0.375%              | 10                  |
| National Partner           | 14% of 15%        | 2.10%                  | 0.525%              | 14                  |
| Global Partner             | 20% of 15%        | 3.00%                  | 0.75%               | 20                  |

---

# Level Progression Philosophy

Level progression is based on achieving business requirements.

The system does not use XP.

Progress is calculated directly from the requirements needed to unlock the next level.

Example:

State Partner Requirements:

```text
Lifetime Referrals: 200

Conversions: 120

Lifetime GMV: ₦7,000,000
```

Current Ambassador:

```text
Lifetime Referrals: 150

Conversions: 86

Lifetime GMV: ₦4,800,000
```

Requirement Progress:

```text
Referrals

150 / 200

= 75%

Conversions

86 / 120

= 71.67%

Lifetime GMV

₦4.8M / ₦7M

= 68.57%
```

Overall Progress:

```text
Average(
75,
71.67,
68.57
)

= 71.75%
```

Displayed Value:

```text
72%
```

This overall progress percentage is shown in the level summary card.

The detailed requirement progress percentages are shown separately in the requirements section.

This provides:

* Quick understanding at the top
* Full transparency below

---

# Backend Routes

---

## GET /ambassador/levels

### Purpose

Returns the ambassador's current level, progression status, commissions, customer benefits, level requirements, unlocked benefits, and next-level preview.

---

### Response

> **Fixed:** `qualifiedCustomers` renamed to `conversions` — this is the same metric exposed as `conversions` everywhere else in the platform (Dashboard, Analytics, Referrals). Keeping a different key name here just for Levels created an unnecessary translation step for the frontend.

```json
{
  "currentLevel": {
    "id": 2,
    "name": "City Partner",
    "badge": "city",
    "color": "#2563EB"
  },

  "progress": {
    "overallProgressPercentage": 71.75
  },

  "requirements": {
    "lifetimeReferrals": {
      "current": 150,
      "required": 200,
      "progressPercentage": 75
    },

    "conversions": {
      "current": 86,
      "required": 120,
      "progressPercentage": 71.67
    },

    "lifetimeGMV": {
      "current": 4800000,
      "required": 7000000,
      "progressPercentage": 68.57
    }
  },

  "commissions": {
    "platformCommissionPool": "15%",
    "ambassadorCommission": "1.05%",
    "customerDiscount": "0.2625%"
  },

  "customerBenefits": {
    "discountedOrdersAllowed": 7
  },

  "benefits": [
    "Higher referral commission",
    "Customer discount eligibility",
    "Priority campaign access"
  ],

  "nextLevel": {
    "name": "State Partner",

    "commission": "1.50%",

    "customerDiscount": "0.375%",

    "discountedOrdersAllowed": 10,

    "newBenefits": [
      "Higher commission",
      "More discounted customer orders",
      "Access to exclusive campaigns"
    ]
  },

  "remainingRequirements": {
    "lifetimeReferrals": 50,
    "conversions": 34,
    "lifetimeGMV": 2200000
  },

  "recommendations": [
    "Refer 50 more customers.",
    "Increase lifetime GMV by ₦2,200,000.",
    "Generate 34 more Conversions."
  ]
}
```

> **Architectural note (carried from the master doc):** `lifetimeGMV` here and `lifetimeAttributedGMV` in the Analytics Hub are the same underlying number. Because GMV attribution formally expires after 90 days or 5 orders per customer, this field must be implemented as an append-only ledger — GMV is credited permanently at the moment it's validly attributed and is never clawed back once a customer's window closes. Both fields should be computed by one shared service, not maintained independently by the Levels Service and the Analytics Service.

---

## GET /ambassador/levels/catalog

### Purpose

Returns all available ambassador levels and associated benefits.

Useful for onboarding, level comparison, and upgrade visibility.

---

### Response

```json
{
  "levels": [
    {
      "id": 1,
      "name": "Campus / Community Partner",
      "commission": "0.75%",
      "customerDiscount": "0.1875%",
      "discountedOrdersAllowed": 5
    },
    {
      "id": 2,
      "name": "City Partner",
      "commission": "1.05%",
      "customerDiscount": "0.2625%",
      "discountedOrdersAllowed": 7
    },
    {
      "id": 3,
      "name": "State Partner",
      "commission": "1.50%",
      "customerDiscount": "0.375%",
      "discountedOrdersAllowed": 10
    },
    {
      "id": 4,
      "name": "National Partner",
      "commission": "2.10%",
      "customerDiscount": "0.525%",
      "discountedOrdersAllowed": 14
    },
    {
      "id": 5,
      "name": "Global Partner",
      "commission": "3.00%",
      "customerDiscount": "0.75%",
      "discountedOrdersAllowed": 20
    }
  ]
}
```

---

## GET /ambassador/levels/history

### Purpose

Returns the ambassador's progression history.

---

### Response

```json
{
  "history": [
    {
      "level": "Campus / Community Partner",
      "achievedAt": "2025-11-18"
    },
    {
      "level": "City Partner",
      "achievedAt": "2026-03-06"
    }
  ]
}
```

---

# UI Structure

The Ambassador Levels page should contain six major sections.

---

# 1. Current Level Card

Displayed at the very top.

Purpose:

Provide an instant understanding of current progression.

Contents:

* Current badge
* Current level
* Progress bar
* Overall progress percentage
* Next level target

Example:

```text
City Partner

███████░░░

72%

Progress to State Partner
```

The displayed percentage is the average of all next-level requirement completion percentages.

---

# 2. Progress Requirements

Displays every requirement needed for the next level.

Purpose:

Provide full transparency into progression.

Example:

```text
Referrals

150 / 200

███████░░░

75%

Conversions

86 / 120

███████░░░

72%

Lifetime GMV

₦4.8M / ₦7M

██████░░░░

69%
```

Each requirement should have:

* Current value
* Required value
* Progress percentage
* Progress bar

---

# 3. Current Benefits

Displays all currently unlocked benefits.

Examples:

* Ambassador commission
* Customer discount percentage
* Discounted customer orders
* Priority campaigns
* Ambassador privileges
* Platform recognition

---

# 4. Next Level Preview

Shows what will be unlocked after upgrading.

Example:

```text
State Partner

Unlocks

Commission increases to 1.50%

Customer discount increases to 0.375%

10 discounted customer orders

Exclusive campaigns
```

---

# 5. Level Comparison Table

Displays every available ambassador level.

| Level              | Ambassador Commission | Customer Discount | Discounted Orders |
| ------------------ | ---------------------- | ------------------ | ------------------ |
| Campus / Community | 0.75%                  | 0.1875%             | 5                   |
| City               | 1.05%                  | 0.2625%             | 7                   |
| State              | 1.50%                  | 0.375%              | 10                  |
| National           | 2.10%                  | 0.525%              | 14                  |
| Global             | 3.00%                  | 0.75%               | 20                  |

This allows ambassadors to visualize the complete progression ladder.

---

# 6. Recommendations

Backend-generated guidance.

Example:

```text
Recommended Next Steps

- Refer 50 more customers.

- Increase lifetime GMV by ₦2.2M.

- Convert 34 more Conversions.
```

The frontend should never generate these recommendations.

---

# Backend Responsibilities

The Levels Service is responsible for:

* Determining ambassador levels
* Validating upgrade eligibility
* Computing requirement progress
* Computing overall progression percentage
* Commission calculations
* Customer discount calculations
* Discounted order allowance
* Benefit lookup
* Recommendation generation
* Progress comparison against next level requirements

All business rules remain centralized in the backend.

---

# Time Window Awareness

The Ambassador Levels module is not time-window aware.

Level progression is cumulative and permanent.

Lifetime metrics determine progression.

Historical achievements are exposed through the level history endpoint.

---

# API Design Philosophy

Responses should contain three categories of information.

## Raw Metrics

* Lifetime referrals
* Lifetime GMV
* Conversions

## Computed Metrics

* Current level
* Requirement progress percentages
* Overall progression percentage
* Upgrade eligibility
* Commission rates
* Customer discount percentage
* Discounted order allowance

## Guidance Metrics

* Remaining requirements
* Recommended actions
* Next level preview
* Unlockable benefits

This ensures the Ambassador Levels module remains transparent, motivational, explainable, and fully aligned with the platform's long-term growth philosophy.