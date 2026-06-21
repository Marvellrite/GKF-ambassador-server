# GokaFoods Ambassador System Integration Architecture

## Overview

The GokaFoods Ambassador System is designed as an **independent application domain** that operates alongside the main consumer system while sharing a unified identity layer.

Even though the ambassador system may live in a separate repository, deployment, and frontend application, it remains tightly coupled through a shared authentication and role system.

The core idea is simple:

> One user identity, multiple product experiences.

---

# 🧠 Identity Layer (Single Source of Truth)

The entire ecosystem is anchored on a unified user identity model.

### Recommended User Schema

```ts id="id0s9x"
User {
  _id,
  email,
  password,
  roles: ["consumer", "ambassador"]
}
```

---

## Key Principle

* Every user is always a **consumer**
* Some users optionally become **ambassadors**
* Ambassador system is simply an **additional role capability**, not a separate identity

This ensures:

* No duplicate accounts
* No identity fragmentation
* One login unlocks multiple systems

---

# 🧩 System Separation Model

| System            | Responsibility                         |
| ----------------- | -------------------------------------- |
| Consumer System   | Orders, payments, food experience      |
| Ambassador System | Referrals, levels, earnings, analytics |

Both systems:

* Share the same authentication identity
* Operate as independent frontends
* Consume the same role-based token

---

# 🔐 Authentication Strategy (Shared Identity)

Both applications rely on a shared authentication mechanism.

After login, the backend issues a JWT containing:

```ts id="jwt1"
{
  userId,
  email,
  roles: ["consumer", "ambassador"]
}
```

This token becomes the **access passport** across both systems.

---

# 🚪 Portal Access Model

Since the ambassador system may live in a separate repository and frontend, navigation between systems cannot rely on internal UI switching.

Instead, access is handled via **portal routing and session synchronization**.

---

# 🧠 4. Silent Session Sync (Magic Layer)

To ensure seamless experience without repeated logins, two integration strategies are supported.

Frontend engineers may choose either approach depending on infrastructure constraints.

---

## Option A (Best): Shared Cookie Domain

### Concept

Both applications share authentication state via a single cookie domain.

### Implementation

Set authentication cookie like:

```text id="cookie1"
Domain: .gokafoods.com
```

---

### Result

* Consumer app and ambassador app automatically share login state
* No token passing required
* No re-authentication needed

---

### Flow

```text id="flow1"
User logs in → cookie stored on .gokafoods.com

Consumer App → authenticated
Ambassador App → automatically authenticated
```

---

### Advantages

* Cleanest architecture
* No redirect complexity
* Most secure (httpOnly cookies)
* Best UX

---

### Requirements

* Same parent domain
* Proper CORS configuration
* Secure cookie settings

---

## Option B (Fallback): Token Handoff Redirect

### Concept

Authentication state is transferred explicitly via secure redirect.

---

### Flow

From Consumer App:

```text id="redirect1"
https://ambassador.gokafoods.com/auth/sync?token=xxx
```

---

### Ambassador App Behavior

1. Extract token from URL
2. Validate token with backend
3. Create local session
4. Redirect to ambassador dashboard

---

### Flow Diagram

```text id="flow2"
Consumer App
   ↓
User clicks "Go to Ambassador Portal"
   ↓
Redirect with token
   ↓
Ambassador App receives token
   ↓
Validates session
   ↓
Creates local login state
```

---

### Advantages

* Works across different domains
* No shared cookie dependency
* Easier for microfrontend separation

---

### Tradeoffs

* Requires secure token handling
* Slightly more complex flow
* Must handle token expiry carefully

---

# 🧠 Role-Based Access Control

Both systems must enforce role validation consistently.

### Middleware Example

```ts id="mw1"
if (!user.roles.includes("ambassador")) {
  redirectToConsumerApp();
}
```

---

### Access Rules

| Role       | Access                        |
| ---------- | ----------------------------- |
| consumer   | Consumer system only          |
| ambassador | Both systems                  |
| both roles | Full portal switching enabled |

---

# 🔄 Portal Navigation Strategy

Since UI is not shared between apps, navigation is explicit.

## From Consumer → Ambassador

* Button: “Go to Ambassador Portal”
* Action: redirect or cookie-based session transfer

## From Ambassador → Consumer

* Button: “Back to Shopping”
* Action: redirect to consumer app

---

# 🧩 Recommended System Behavior

Regardless of integration option chosen:

### Always ensure:

* Single login identity
* Shared JWT structure
* Role-based routing enforcement
* No duplicated user accounts
* Independent frontend deployments

---

# 🧠 Security Considerations

### Must enforce:

* httpOnly cookies (preferred for Option A)
* Short-lived tokens for Option B
* Token validation on every portal entry
* Role verification on backend (never trust frontend)
* Secure redirect whitelist for token handoff

---

# 🧩 Final Architecture Summary

## Identity Layer

```ts id="final1"
User {
  _id,
  email,
  password,
  roles: ["consumer", "ambassador"]
}
```

---

## Application Layer

```text id="final2"
Consumer App → gokafoods.com

Ambassador App → ambassador.gokafoods.com
```

---

## Authentication Layer

* Shared JWT or cookie session
* Central auth validation system

---

## Integration Layer

Choose one:

### Option A

Shared cookie domain (.gokafoods.com)

### Option B

Token handoff redirect (secure sync endpoint)

---

# 🧠 Final Principle

The ambassador system is not a separate identity.

It is:

> A role-extended experience layer built on top of a shared consumer identity.

This ensures:

* Scalability
* Clean separation of concerns
* Independent deployments
* Unified user experience
* Future extensibility (admin, merchant, partners, etc.)

---

# 🚀 Future Extension Ready

This architecture naturally supports adding new systems:

* Admin portal → `roles: ["admin"]`
* Merchant portal → `roles: ["merchant"]`
* Delivery agent system → `roles: ["rider"]`

Without changing the identity model.

---

# Conclusion

Both integration options are valid:

* **Option A (Shared Cookie Domain)** → best UX, recommended for tightly coupled domains
* **Option B (Token Handoff Redirect)** → best for fully independent deployments

The final choice depends on infrastructure constraints, not architecture limitations.
