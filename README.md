# Event → Queue → Push (Redis • Webhook • Firebase)

A full-stack assignment that demonstrates a minimal pipeline to handle **third-party webhooks**, queue jobs in **Redis**, process them with a **worker**, and send **Firebase Cloud Messaging (FCM)** notifications.  
Also includes a **Next.js dashboard** to inspect and replay events.

---

## 🚀 Objective
- Accept incoming webhooks (`order.created`)
- Validate signature, timestamp, and dedupe by event_id
- Enqueue events into Redis
- Worker consumes queue → enriches event with FCM token → sends push notification
- Dashboard to view last 20 events, status, and replay failures

---

## 🛠️ Tech Stack
- **Next.js** (TypeScript) — API routes & dashboard
- **Redis** — queue, dedupe, metrics, DLQ
- **Firebase Admin SDK** — FCM notifications
- **Jest** — tests
- **TailwindCSS** — minimal UI styling

---

## 📂 Project Structure

/README.md
/.env.example
/pages/api/webhook/order.created.ts
/pages/api/health.ts
/pages/api/metrics.ts
/pages/admin/events.tsx
/worker.ts
/src/lib/redis.ts
/src/lib/hmac.ts
/src/lib/fcm.ts
/tests/*.test.ts


---

## ⚙️ Setup

### 1. Clone the repo
```bash
git clone https://github.com/<your-username>/event-queue-push.git
cd event-queue-push

 2. Install dependencies
npm install

3. Configure environment

Copy .env.example → .env.development.local and fill values:
WEBHOOK_SECRET=replace_with_secret
REDIS_URL=redis://localhost:6379
FIREBASE_PROJECT_ID=...
FIREBASE_CLIENT_EMAIL=...
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
FCM_DRY_RUN=true

4. Run services
Start Next.js:
npm run dev

## Start worker (in another terminal):
npm run worker

📡 API Endpoints
POST /api/webhook/order.created
Accepts signed webhook events.
Validates: HMAC-SHA256 signature, timestamp (≤ 5 min old), dedupe by event_id.
Enqueues: to orders-stream or orders-list.
Rate limit: 10 requests / 10s / IP.

Example payload:
{
  "event_id": "evt_12345",
  "type": "order.created",
  "data": { "order_id": "ord_987", "userId": "user_42", "amount": 1599 }
}

GET /api/health
{ "ok": true, "redis": "up" }

GET /api/metrics
{ "received": 10, "deduped": 2, "sent": 7, "failed": 1, "dlq": 0 }

⚒️ Worker
Consumes Redis events
Looks up fcm:token:<userId>
Sends push notification via FCM:
Title: New Order
Body: Order <order_id> placed by <userId>

🖥️ Dashboard

Route: /admin/events

Features:
Show last 20 events with status (queued | processing | sent | failed)
Replay button for failed events
Search by event_id

## Run with:
npm test

📊 Flow Diagram
[Webhook Source] 
     ↓
 POST /api/webhook/order.created 
     ↓ (verify HMAC, timestamp, dedupe)
 [Redis Queue]
     ↓
   Worker → [Firebase FCM]
     ↓
 Success → update status: sent
 Failure → retry → DLQ

Admin UI: /admin/events → Inspect & Replay


📌 Deliverables

API routes: /api/webhook/*, /api/health, /api/metrics
Worker script worker.ts
Next.js admin dashboard /admin/events
Postman collection / curl examples
Jest test suite
