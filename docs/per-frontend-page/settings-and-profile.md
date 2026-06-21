# Ambassador Settings & Profile API Specification (Refined)

## Overview

The Ambassador Settings module is the **configuration and control center for ambassador-specific account features**.

It does not own user identity.

Instead, it **extends the base consumer identity** from the main GokaFoods system and overlays ambassador-specific settings, preferences, and operational controls.

---

## 🧠 System Boundary Rule

### Identity Source of Truth

All identity data originates from the Consumer System:

* name
* email
* phone
* avatar

The ambassador system must never mutate or redefine identity.

---

### Ambassador System Responsibility

The ambassador system only manages:

* ambassador configuration
* financial settings (withdrawal preferences)
* security controls (2FA, sessions)
* notification preferences
* ambassador-specific preferences

---

# Design Philosophy

The Settings module is divided into independent configuration domains:

1. Identity View (Read-only from Consumer System)
2. Payment & Withdrawal Configuration
3. Notification Preferences
4. Security Controls
5. Session & Device Activity
6. Ambassador Preferences

This ensures:

* clean separation of systems
* no duplicated identity logic
* scalable multi-repo architecture

---

# Backend Architecture Rule

The endpoint below is an **aggregation layer**, not a data owner:

```http
GET /ambassador/settings
```

It composes data from:

* Consumer System (Identity Service)
* Ambassador System (Configuration Service)
* Wallet System (Financial Service)
* Auth System (Security Service)

---

# GET /ambassador/settings

## Purpose

Returns a fully composed ambassador settings view.

---

## Response

```json
{
  "identity": {
    "fullName": "John Doe",
    "email": "john@example.com",
    "phoneNumber": "+2348012345678",
    "avatarUrl": "https://...",
    "level": "City Partner",
    "joinedAt": "2026-01-10"
  },

  "payment": {
    "bankAccountConfigured": true,
    "bankName": "Access Bank",
    "accountName": "John Doe",
    "accountNumberMasked": "0123******"
  },

  "notifications": {
    "email": true,
    "push": true,
    "sms": false,

    "competitionUpdates": true,
    "walletUpdates": true,
    "withdrawalUpdates": true,
    "marketingUpdates": false
  },

  "security": {
    "twoFactorEnabled": true,
    "lastPasswordChange": "2026-05-10"
  },

  "sessions": {
    "activeSessionCount": 2
  },

  "preferences": {
    "currency": "NGN",
    "timezone": "Africa/Lagos"
  }
}
```

---

# 🧩 Identity Management (Read-Only Layer)

## GET /ambassador/profile (Deprecated conceptually)

### Important Rule

This endpoint should NOT be treated as a mutation source.

It is simply a **projection of consumer identity**.

---

## Recommended Replacement Behavior

Instead of:

* `/ambassador/profile` (editable identity ❌)

Use:

* `/consumer/profile` (source of truth)
* `/ambassador/settings` (read-only identity view)

---

# 💳 Payment & Withdrawal Configuration

## GET /ambassador/payment-details

Returns ambassador withdrawal configuration.

---

## POST /ambassador/payment-details

Registers withdrawal account.

---

## PATCH /ambassador/payment-details

Updates withdrawal account.

---

## Rule

Payment details belong to the **Wallet System**, not identity.

Ambassador system only configures them.

---

# 🔔 Notification Preferences

Fully ambassador-owned configuration.

---

## GET /ambassador/notifications/settings

Returns notification preferences.

---

## PATCH /ambassador/notifications/settings

Updates notification preferences.

---

# 🔐 Security Settings (Delegated Ownership)

Security is shared responsibility between:

* Auth System
* Ambassador System (UI layer only)

Security is owned entirely by Auth System.
Ambassador system is only a client that requests security operations.”

---

## GET /ambassador/security

Returns:

* 2FA status
* password metadata (last changed)

---

## Actions

* change password → Auth Service
* enable/disable 2FA → Auth Service
* session control → Auth Service

---

# 🖥️ Session & Account Activity

## GET /ambassador/security/sessions

Returns active sessions from Auth System.

---

## DELETE /ambassador/security/sessions/:id

Terminates session via Auth Service.

---

# ⚙️ Preferences Layer

Ambassador-specific UX configuration.

Examples:

* currency display
* timezone
* future personalization flags

---

# 🧠 Key Architectural Correction

## Before (problematic assumption)

* Ambassador system partially owns identity
* Profile is editable inside ambassador domain
* Settings mixes identity + configuration ownership

---

## After (correct separation)

| Domain            | Responsibility                  |
| ----------------- | ------------------------------- |
| Consumer System   | Identity (source of truth)      |
| Ambassador System | Configuration + extension layer |
| Wallet System     | Financial accounts              |
| Auth System       | Security + sessions             |

---

# 🧩 Backend Responsibilities (Refined)

The Ambassador Settings Service is responsible for:

* aggregating multi-domain data
* managing ambassador preferences
* managing notification settings
* delegating identity updates to consumer system
* delegating security actions to auth system
* delegating financial updates to wallet system

---

# 🕒 Time Window Awareness

Settings remain **non-time-windowed**.

However:

Allowed time-aware fields:

* last password change
* last login
* session activity timestamps

Everything else is current-state configuration.

---

# 🧠 API Design Philosophy (Refined)

Responses are structured into **ownership layers**:

## 1. Identity Layer (Read-only projection)

* name
* email
* phone
* avatar
* level

## 2. Financial Configuration Layer

* bank details
* withdrawal configuration

## 3. Communication Layer

* notification settings

## 4. Security Layer

* 2FA status
* session metadata

## 5. Preference Layer

* UI/UX personalization settings

---

# 🚀 Final Insight

The Settings module is NOT:

> a user profile system

It IS:

> a cross-system configuration aggregator for ambassador behavior inside a multi-domain architecture

---

# 🧠 Why This Matters

This refinement ensures:

* no duplication of identity logic across repos
* clean separation between consumer and ambassador systems
* safer scaling into multiple platforms (admin, merchant, rider, etc.)
* predictable ownership boundaries across services
