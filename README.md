# Bitespeed Identity Reconciliation (MongoDB Version)
--
> **GitHub:** `https://github.com/rajd333-lgtm/bitespeed`
> **Live Endpoint:** `POST https://bitespeed-zjh4.onrender.com/identify`

---
## Setup

1. Create .env file
2. Add:

DATABASE_URL="your-mongodb-connection-string-with-database-name"

3. Run:
   npm install
   npx prisma generate
   # server listens on port 5000 by default
   npm run dev

NOTE: Do NOT run prisma migrate with MongoDB.

## Endpoint

POST /identify

Body:
{
  "email": "string",
  "phoneNumber": "string"
}
---

## 🔄 Workflow / Reconciliation Flow

```
  Client
    │
    │  POST /identify
    │  { email?, phoneNumber? }
    ▼
┌───────────────────────────────────────────────────────┐
│                   Input Validation                    │
│   email or phoneNumber must be present                │
│   → 400 Bad Request if both are null/empty            │
└───────────────────────┬───────────────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────────────┐
│         Query: find contacts WHERE                    │
│         email = ? OR phoneNumber = ?                  │
└───────────────────────┬───────────────────────────────┘
                        │
          ┌─────────────┴─────────────┐
          │                           │
    No matches                   Matches found
          │                           │
          ▼                           ▼
  ┌───────────────┐      ┌────────────────────────┐
  │  CASE A       │      │  Resolve each matched  │
  │               │      │  contact → root primary │
  │  INSERT new   │      └────────────┬───────────┘
  │  PRIMARY      │                   │
  │  contact      │       ┌───────────┴────────────┐
  └───────┬───────┘       │                        │
          │         1 unique primary        2+ primaries found
          │               │                        │
          │               ▼                        ▼
          │       ┌───────────────┐      ┌─────────────────────┐
          │       │  CASE B       │      │  CASE C  (MERGE)    │
          │       │               │      │                     │
          │       │  Is incoming  │      │  Sort by createdAt  │
          │       │  (email,phone)│      │  oldest = TRUE      │
          │       │  pair new?    │      │  PRIMARY            │
          │       └──────┬────────┘      │                     │
          │         ┌────┴────┐          │  Demote all other   │
          │        YES       NO          │  primaries →        │
          │         │         │          │  SECONDARY          │
          │         ▼         │          │                     │
          │   INSERT new      │          │  Re-link their      │
          │   SECONDARY       │          │  secondaries to     │
          │   linked to       │          │  true primary       │
          │   primary         │          └──────────┬──────────┘
          │         │         │                     │
          └────┬────┘         │         ┌───────────┘
               │              │         │  Still new info?
               │              │         │  → INSERT SECONDARY
               │              │         │
               └──────────────┴─────────┘
                                        │
                                        ▼
                           ┌────────────────────────┐
                           │   Build Response        │
                           │                         │
                           │   primaryContatctId     │
                           │   emails[]  (primary    │
                           │             first)      │
                           │   phoneNumbers[]        │
                           │   secondaryContactIds[] │
                           └────────────┬────────────┘
                                        │
                                        ▼
                                  HTTP 200 + JSON
```
---

## 🧪 Test with curl

```bash
# New customer
curl -X POST https://bitespeed-zjh4.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"email":"lorraine@hillvalley.edu","phoneNumber":"123456"}'

# Same phone, new email
curl -X POST https://customer-id.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"email":"mcfly@hillvalley.edu","phoneNumber":"123456"}'

# Phone only
curl -X POST https://customer-id.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber":"123456"}'
```

---
