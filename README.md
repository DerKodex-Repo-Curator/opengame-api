# Open Game API

The backend API service for the Open Game System (OGS), built on Cloudflare Workers with D1.

## Overview

Open Game API provides the server-side infrastructure for the OGS ecosystem. The initial MVP focuses on **push notification delivery** -- allowing game servers to send notifications to users who have registered their devices through the OGS companion app.

## API Endpoints

### Health Check

```
GET /api/v1/health
```

Returns `{ "status": "ok" }`.

### Device Registration

```
POST /api/v1/devices/register
Content-Type: application/json

{
  "ogsDeviceId": "unique-device-id",
  "platform": "ios" | "android",
  "pushToken": "apns-or-fcm-token"
}
```

Registers or updates a device for push notifications. Called by the OGS companion app. No API key required.

**Response:**
```json
{
  "deviceId": "unique-device-id",
  "registered": true
}
```

### Send Notification

```
POST /api/v1/notifications/send
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "deviceId": "unique-device-id",
  "notification": {
    "title": "Game Update",
    "body": "It's your turn!",
    "data": { "gameId": "abc123" }
  }
}
```

Sends a push notification to a registered device. Requires a valid API key.

**Response:**
```json
{
  "id": "generated-uuid",
  "status": "sent"
}
```

## Development

### Prerequisites

- Node.js (v18+)
- Wrangler CLI (`npm install -g wrangler`)
- Cloudflare account

### Setup

```bash
npm install
```

### Local Development

```bash
npm run dev
```

The API runs at `http://localhost:8787`.

### Database Migration

Apply the D1 schema:

```bash
npm run db:migrate
```

### Type Checking

```bash
npm run typecheck
```

### Deploy

```bash
npm run deploy
```

## Architecture

- **Runtime:** Cloudflare Workers
- **Framework:** Hono
- **Database:** Cloudflare D1 (SQLite)
- **Language:** TypeScript

### Project Structure

```
src/
  index.ts              # App entry point, route mounting
  types.ts              # Shared TypeScript types
  middleware/
    auth.ts             # API key validation middleware
  providers/
    push.ts             # Push notification provider interface (APNs/FCM stubs)
  routes/
    devices.ts          # Device registration endpoint
    notifications.ts    # Notification send endpoint
schema.sql              # D1 database schema
```

## License

MIT
