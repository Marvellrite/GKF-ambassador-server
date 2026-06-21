# Wallet & Financial Management API Specification (Refined)

## Overview

The Wallet & Financial Management subsystem is responsible for managing an ambassador's earnings, withdrawals, internal transfers, and complete financial history.

It answers four primary questions:

- How much money do I currently have?
- Where did my money come from?
- Where has my money gone?
- What actions can I perform with my funds?

---

# Design Principles

The wallet system should remain **ledger-driven**.

Every financial movement must exist as a transaction.

No wallet balance should ever be edited directly.

Wallet Balance = Sum(All Credits) − Sum(All Debits)

---

# Financial States

An ambassador's money exists in different states.

> **Fixed:** "Revenue This Cycle" renamed to **Competition Commission** — this is the same metric the Competition Center spec exposes under that name, and the two should be sourced from the same computation rather than maintained as separate fields with different names.

| State | Meaning |
|---------|----------|
| Available Balance | Money immediately available for withdrawal or transfer |
| Pending Earnings | Earnings awaiting settlement |
| Withdrawable Balance | Amount eligible for bank withdrawal |
| Competition Commission | Earnings generated during the current competition cycle |
| Lifetime Earnings | Total earnings since account creation |
| Total Withdrawn | Total money successfully withdrawn |
| Total Transferred | Total amount moved into Consumer Wallet |

---

# API Endpoints

---

# 1. Wallet Summary

## Endpoint

GET /wallet

## Purpose

Returns a complete financial overview of the ambassador.

This endpoint powers the main wallet page.

---

## Expected Response

> **Fixed:** `revenueThisCycle` renamed to `competitionCommission`.

```json
{
  "success": true,
  "data": {
    "wallet": {
      "currency": "NGN",

      "availableBalance": 184500,

      "pendingEarnings": 27500,

      "withdrawableBalance": 184500,

      "competitionCommission": 358000,

      "lifetimeEarnings": 1584000,

      "totalWithdrawn": 920000,

      "totalTransferred": 180000
    },

    "quickActions": {
      "canWithdraw": true,
      "canTransferToConsumerWallet": true
    },

    "lastUpdated": "2026-06-18T09:20:44Z"
  }
}
```

---

## UI Placement

### Wallet Header Card

Displays

- Available Balance (large)
- Pending Earnings
- Withdrawable Balance

---

### Revenue Card

Displays

- Competition Commission
- Lifetime Earnings

---

### Activity Card

Displays

- Total Withdrawn
- Total Transferred

---

### Quick Action Buttons

- Withdraw to Bank
- Transfer to Consumer Wallet

---

# 2. Wallet Transaction History

## Endpoint

GET /wallet/transactions

---

## Purpose

Returns every wallet transaction regardless of type.

---

## Optional Query Parameters

| Parameter | Description |
|------------|-------------|
| page | Pagination |
| limit | Items per page |
| type | credit, debit |
| category | commission, reward, withdrawal, transfer |
| from | Start date |
| to | End date |
| sort | asc or desc |

Example

```
GET /wallet/transactions?page=1&limit=20&type=credit
```

---

## Expected Response

```json
{
  "success": true,
  "data": {
    "summary": {
      "credits": 560000,
      "debits": 120000,
      "net": 440000
    },

    "transactions": [
      {
        "id": "txn_001",
        "type": "credit",
        "category": "Referral Commission",
        "amount": 8500,
        "status": "completed",
        "description": "Commission from Order #GK2319",
        "createdAt": "2026-06-15T11:10:21Z"
      },
      {
        "id": "txn_002",
        "type": "debit",
        "category": "Withdrawal",
        "amount": 50000,
        "status": "completed",
        "description": "Withdrawal to GTBank",
        "createdAt": "2026-06-14T15:30:18Z"
      }
    ],

    "pagination": {
      "page": 1,
      "limit": 20,
      "totalItems": 124,
      "totalPages": 7
    }
  }
}
```

---

## UI Placement

### Financial Summary

Shows

- Total Credits
- Total Debits
- Net Movement

---

### Transaction Table

Columns

- Type
- Category
- Amount
- Status
- Description
- Date

---

### Filters

- Transaction Type
- Category
- Date Range
- Search
- Pagination

---

# 3. Withdrawal History

## Endpoint

GET /wallet/withdrawals

---

## Purpose

Returns every withdrawal request made by the ambassador.

---

## Optional Query Parameters

| Parameter | Description |
|------------|-------------|
| status | pending, processing, completed, failed |
| page | Pagination |
| limit | Items per page |
| from | Start date |
| to | End date |

---

## Expected Response

```json
{
  "success": true,
  "data": {
    "summary": {
      "totalWithdrawn": 920000,
      "pendingWithdrawals": 1,
      "completedWithdrawals": 18
    },

    "withdrawals": [
      {
        "id": "wd_101",
        "amount": 50000,
        "status": "completed",
        "bankName": "GTBank",
        "accountName": "John Doe",
        "accountNumber": "0123456789",
        "reference": "GKF-WD-238729",
        "requestedAt": "2026-06-14T10:20:00Z",
        "completedAt": "2026-06-14T12:10:00Z"
      }
    ],

    "pagination": {
      "page": 1,
      "limit": 20,
      "totalItems": 19,
      "totalPages": 1
    }
  }
}
```

---

## UI Placement

### Withdrawal Statistics

Displays

- Total Withdrawn
- Pending Withdrawals
- Completed Withdrawals

---

### Withdrawal History Table

Columns

- Amount
- Status
- Bank
- Reference
- Requested Date
- Completed Date

---

### Filters

- Status
- Date Range
- Pagination

---

# 4. Consumer Wallet Transfer History

## Endpoint

GET /wallet/transfers

---

## Purpose

Returns transfers from Ambassador Wallet to the GKF Consumer Wallet.

---

## Optional Query Parameters

| Parameter | Description |
|------------|-------------|
| page | Pagination |
| limit | Items per page |
| status | completed, pending, failed |
| from | Start date |
| to | End date |

---

## Expected Response

```json
{
  "success": true,
  "data": {
    "summary": {
      "totalTransferred": 180000,
      "transferCount": 8
    },

    "transfers": [
      {
        "id": "tr_120",
        "amount": 10000,
        "status": "completed",
        "consumerWalletId": "cw_001",
        "reference": "GKF-TR-100234",
        "createdAt": "2026-06-10T13:15:12Z"
      }
    ],

    "pagination": {
      "page": 1,
      "limit": 20,
      "totalItems": 8,
      "totalPages": 1
    }
  }
}
```

---

## UI Placement

### Transfer Summary

Displays

- Total Transferred
- Number of Transfers

---

### Transfer History Table

Columns

- Amount
- Status
- Reference
- Date

---

### Filters

- Status
- Date Range
- Pagination

---

# 5. Competition Reward History

## Endpoint

GET /wallet/rewards

---

## Purpose

Returns every competition reward credited into the wallet.

---

## Optional Query Parameters

| Parameter | Description |
|------------|-------------|
| competitionId | Specific cycle |
| page | Pagination |
| limit | Items per page |

---

## Expected Response

> **Fixed:** `cycleRevenue` renamed to `competitionCommission`.

```json
{
  "success": true,
  "data": {
    "rewards": [
      {
        "competitionId": "cmp_2026_06",
        "competitionMonth": "June 2026",
        "rank": 2,
        "rewardPercentage": 8,
        "competitionCommission": 510000,
        "rewardAmount": 40800,
        "creditedAt": "2026-07-01T09:00:00Z"
      }
    ],

    "pagination": {
      "page": 1,
      "limit": 12,
      "totalItems": 6,
      "totalPages": 1
    }
  }
}
```

---

## UI Placement

### Reward History Table

Columns

- Competition Cycle
- Rank
- Competition Commission
- Reward %
- Reward Amount
- Credited Date

---

# 6. Withdraw to Personal Bank

## Endpoint

POST /wallet/withdraw

---

## Purpose

Creates a withdrawal request.

---

## Request

```json
{
  "amount": 50000,
  "bankAccountId": "bank_123"
}
```

---

## Success Response

```json
{
  "success": true,
  "message": "Withdrawal request submitted successfully.",
  "data": {
    "withdrawalId": "wd_104",
    "amount": 50000,
    "status": "processing",
    "remainingBalance": 134500,
    "estimatedCompletion": "2026-06-19T17:00:00Z"
  }
}
```

---

## Validation Errors

Possible errors

```json
{
  "success": false,
  "message": "Insufficient withdrawable balance."
}
```

```json
{
  "success": false,
  "message": "Bank account not found."
}
```

```json
{
  "success": false,
  "message": "Minimum withdrawal amount is NGN 2,000."
}
```

---

## UI Placement

Withdrawal Modal

Fields

- Amount
- Saved Bank Account Selector

Displays

- Withdrawable Balance
- Estimated Remaining Balance

Submit Button

---

# 7. Transfer to GKF Consumer Wallet

## Endpoint

POST /wallet/transfer-to-consumer-wallet

---

## Purpose

Transfers funds from the Ambassador Wallet into the GKF Consumer Wallet.

This action is irreversible.

---

## Request

```json
{
  "amount": 10000
}
```

---

## Success Response

```json
{
  "success": true,
  "message": "Transfer completed successfully.",
  "data": {
    "transferId": "tr_212",
    "amount": 10000,
    "remainingBalance": 174500,
    "consumerWalletBalance": 48500,
    "completedAt": "2026-06-18T14:35:00Z"
  }
}
```

---

## Validation Errors

```json
{
  "success": false,
  "message": "Insufficient wallet balance."
}
```

```json
{
  "success": false,
  "message": "Consumer wallet not found."
}
```

---

## UI Placement

Transfer Modal

Displays

- Available Balance
- Consumer Wallet Balance

Fields

- Transfer Amount

Confirmation Notice

> Transfers to the GKF Consumer Wallet cannot be reversed.

Submit Button

---

# Recommended Wallet Page Layout

```
Wallet Dashboard
│
├── Wallet Summary Cards
│     ├── Available Balance
│     ├── Pending Earnings
│     ├── Withdrawable Balance
│     ├── Competition Commission
│
├── Quick Actions
│     ├── Withdraw to Bank
│     └── Transfer to Consumer Wallet
│
├── Financial Analytics
│     ├── Lifetime Earnings
│     ├── Total Withdrawn
│     └── Total Transferred
│
├── Transaction History
│
├── Withdrawal History
│
├── Consumer Wallet Transfers
│
└── Competition Reward History
```

## Time Window Awareness

Unlike the Analytics Hub or Competition Center, the Wallet subsystem should **not** be globally time-window aware because it serves as the ambassador's financial ledger. Instead, historical resources should support **optional date-range filters** (`from`, `to`) for reporting and searching, while summary values such as Available Balance, Pending Earnings, Withdrawable Balance, and Lifetime Earnings should always reflect the current state. The only inherently cycle-aware metric on the wallet dashboard is **Competition Commission**, which is automatically scoped to the active competition month and is the same field exposed by the Competition Center.

## Quick Actions

### Purpose

The `quickActions` object communicates which wallet actions are currently permitted for the authenticated ambassador.

Although every ambassador is generally expected to have access to withdrawals and Consumer Wallet transfers, these permissions are intentionally returned by the backend rather than being hardcoded into the frontend.

This ensures that business rules remain centralized within the backend and allows the frontend to simply render the UI according to the current system state.

### Example Response

```json
"quickActions": {
  "canWithdraw": true,
  "canTransferToConsumerWallet": true
}
```

### Why It Exists

Returning these permissions from the backend provides several architectural benefits.

#### 1. Centralized Business Rules

The backend remains the single source of truth.

Instead of every frontend application implementing its own logic to determine whether an action should be enabled, the backend communicates the current permissions directly.

This prevents duplicated business logic across multiple clients.

---

#### 2. Future-Proofing

Although every ambassador may currently be allowed to perform both actions, future business requirements may introduce temporary restrictions such as:

* Wallet frozen
* Account suspended
* Pending KYC verification
* Withdrawal maintenance window
* Fraud review
* Regulatory restrictions
* Competition reward lock period
* Minimum wallet age before first withdrawal

When any of these occur, the response can simply become:

```json
"quickActions": {
  "canWithdraw": false,
  "canTransferToConsumerWallet": true
}
```

without requiring any frontend code changes.

---

#### 3. Consistent User Experience

Instead of allowing the ambassador to open a withdrawal modal only to receive an error after submission, the frontend can immediately disable or hide unavailable actions.

For example:

* Disable the "Withdraw" button
* Display a tooltip explaining why the action is unavailable
* Prevent unnecessary API calls

---

#### 4. Multi-Platform Compatibility

If GokaFoods later supports:

* Web
* Android
* iOS
* Admin Dashboard

all clients receive identical permission information from the backend.

This guarantees consistent behavior across every platform.

---

### Frontend Usage

The frontend should render action buttons based on these permissions.

Example:

```ts
if (wallet.quickActions.canWithdraw) {
    // Enable Withdraw button
} else {
    // Disable button or show explanation
}

if (wallet.quickActions.canTransferToConsumerWallet) {
    // Enable Transfer button
}
```

### Current System Behaviour

At present, both values are expected to be `true` for almost every ambassador.

They are included not because permissions currently differ, but because they establish a scalable API contract that can accommodate future business rules without requiring changes to the frontend interface or response structure.