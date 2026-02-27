# Bitespeed Identity Reconciliation (MongoDB Version)

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
