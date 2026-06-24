# Multi-Vendor E-Commerce Marketplace

Full-stack marketplace with **Customer**, **Seller**, and **Admin** roles. Built with FastAPI, MongoDB, React, Redux Toolkit, Tailwind CSS, and Razorpay (test mode).

## Features

- JWT auth with role-based access (customer / seller / admin)
- Product catalog, cart, multi-seller checkout, Razorpay order payments
- Weekly seller subscriptions with upload quotas
- Extra quota requests (pay → admin approve)
- Seller & admin analytics dashboards
- Reviews, wishlist, notifications

## Prerequisites

- Python 3.11+ (3.14 supported with latest pydantic wheels)
- Node.js 18+
- MongoDB (local or Atlas)
- [Razorpay Test Keys](https://dashboard.razorpay.com/app/keys)

## Quick Start

### 1. Backend

```bash
cd backend
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Edit .env with MONGODB_URI, JWT_SECRET, RAZORPAY_KEY_ID, RAZORPAY_KEY_SECRET
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

API docs: http://localhost:8000/docs

**Default admin** (seeded on startup):

- Email: `admin@marketplace.com`
- Password: `Admin@12345`

### 2. Frontend

```bash
cd frontend
npm install
cp .env.example .env
# Set VITE_API_BASE_URL and VITE_RAZORPAY_KEY_ID
npm run dev
```

App: http://localhost:5173

## Razorpay Test Mode

1. Create a Razorpay test account and copy Key ID + Secret into `backend/.env`
2. Put **Key ID only** in `frontend/.env` as `VITE_RAZORPAY_KEY_ID`
3. Use **India domestic** test methods (many accounts block international cards):
   - **UPI (test):** `success@razorpay` in test mode, or Razorpay's UPI test flow
   - **Card (domestic test):** see [Razorpay test cards](https://razorpay.com/docs/payments/payments/test-card-details/) — e.g. Mastercard `5267 3181 8797 5449`, expiry any future date, CVV any
   - Error `international_transaction_not_allowed` means the card/network is treated as non-Indian; switch method or enable international cards in Razorpay Dashboard → Settings (if available for your account)


   5500 6700 0000 1002

### Seller subscription payment (API flow)

| Step | Endpoint | Role |
|------|----------|------|
| 1 | `POST /api/v1/subscriptions/subscribe` body `{ "planKey": "basic" }` | Seller JWT |
| 2 | Razorpay Checkout in browser (uses `key`, `razorpayOrderId` from step 1) | — |
| 3 | `POST /api/v1/subscriptions/verify` with `razorpay_order_id`, `razorpay_payment_id`, `razorpay_signature`, `planId` | Seller JWT |

Alternative (same backend logic): `POST /api/v1/payments/create-order` with `{ "paymentType": "subscription", "planId": "..." }` then `POST /api/v1/payments/verify`.

## E2E Test Flows

### Seller subscription + upload quota

1. Register as **Seller**
2. Go to **Seller → Subscription** → subscribe to Basic (₹99/week, 10 uploads)
3. Or call `POST /api/v1/subscriptions/subscribe` with `{ "planKey": "basic" }` (also: `standard`, `premium`, `advanced`)
4. Complete Razorpay test payment via `POST /api/v1/subscriptions/verify`
4. **Seller → Products** → add products (quota decreases)
5. 11th upload should fail with quota error

### Customer purchase

1. Register as **Customer**
2. Browse home → add items to cart → **Checkout** → pay via Razorpay
3. View **Orders** for tracking

### Quota request

1. Exhaust seller upload quota
2. **Seller → Quota** → request +20 uploads → pay
3. Login as **admin** → **Quota Requests** → Approve
4. Seller can upload again

### Admin block seller

1. **Admin → Users** → Block a seller
2. Seller cannot create products (403)

## Project Structure

```
backend/app/     FastAPI routes, services, MongoDB
frontend/src/    React pages, RTK Query, dashboards
```

## API Prefix

All routes: `/api/v1`

## License

MIT
