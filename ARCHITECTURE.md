# Eunilite — Technical Architecture

## Overview

Eunilite is built as a modern fullstack application with a React Native mobile app, Next.js web app, and a Node.js REST API backed by PostgreSQL.

---

## System Architecture

```
┌─────────────────┐     ┌─────────────────┐
│   Mobile App    │     │    Web App      │
│  React Native   │     │   Next.js       │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └──────────┬────────────┘
                    │ HTTPS / REST API
         ┌──────────▼────────────┐
         │     API Server        │
         │   Node.js + Express   │
         └──────────┬────────────┘
                    │
         ┌──────────▼────────────┐
         │     PostgreSQL DB     │
         │   (AWS RDS - CPT)     │
         └───────────────────────┘
```

---

## Core Database Tables

```sql
-- Groups
groups (id, name, type, contribution_amount, payout_day, cycle_length, created_at)

-- Members
members (id, group_id, user_id, turn_order, joined_at, status)

-- Users
users (id, name, phone, email, bank_account, created_at)

-- Contributions
contributions (id, group_id, member_id, amount, month, paid_at, method, status)

-- Payouts
payouts (id, group_id, member_id, amount, scheduled_date, paid_at, status)

-- Rules
group_rules (id, group_id, late_penalty, grace_days, approval_required)
```

---

## API Endpoints (Planned)

```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh

GET    /groups
POST   /groups
GET    /groups/:id
PUT    /groups/:id

POST   /groups/:id/members/invite
GET    /groups/:id/members
DELETE /groups/:id/members/:memberId

GET    /groups/:id/contributions
POST   /groups/:id/contributions

GET    /groups/:id/payouts
POST   /groups/:id/payouts/:payoutId/confirm

GET    /groups/:id/schedule
GET    /groups/:id/reports
```

---

## Payment Flow

```
Member initiates payment
        ↓
Ozow / Peach Payments gateway
        ↓
Webhook confirms payment
        ↓
API marks contribution as paid
        ↓
All members notified
        ↓
If all paid → trigger payout
        ↓
Bank transfer to recipient
        ↓
Receipt sent via WhatsApp + email
```

---

## Third-party Services

| Service | Purpose |
|---|---|
| Ozow | Instant EFT payments |
| Peach Payments | Card payments + debit orders |
| WhatsApp Business API | Reminders and notifications |
| Auth0 | Authentication |
| AWS S3 | File storage |
| AWS RDS | PostgreSQL hosting |
| SendGrid | Transactional emails |
| Sentry | Error tracking |

---

## Security

- All API endpoints require JWT authentication
- Sensitive data encrypted at rest (AES-256)
- HTTPS only — no HTTP
- POPIA compliant data handling
- Bank details stored via tokenisation only
- Regular security audits planned pre-launch
