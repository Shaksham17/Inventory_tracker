# Inventory Tracker

Inventory Tracker is a small business inventory workspace with a React dashboard and an Express API backed by MongoDB. The implemented core covers JWT authentication and server-side roles, product and directory CRUD, warehouse-level balances, atomic stock adjustments with a movement ledger, a database-backed dashboard, current inventory valuation, and deterministic demand/reorder insights.

## Requirements

- Node.js 20 or newer and npm
- MongoDB 6+ replica set for transaction-backed stock adjustments. A local single-node replica set is suitable for development.

## Run locally

1. Start MongoDB. For transaction support, run it as a replica set and initialize it once with `rs.initiate()`.
2. Copy `backend/.env.example` to `backend/.env` and set a long random `JWT_SECRET`. Set `MONGODB_URI` to your MongoDB URI.
3. Copy `frontend/.env.example` to `frontend/.env` if you need to change the API URL.
4. From the repository root, run `npm install`, `npm --prefix backend install`, and `npm --prefix frontend install`.
5. Run `npm run dev`. The frontend is at http://localhost:5173 and API at http://localhost:9999.
6. To fill the dashboard with representative development data, run `npm --prefix backend run seed:demo` from the repository root. It adds 12 demo products, 4 categories, 2 warehouses, stock balances, a supplier, a customer, and dated stock movements. Sign in using an existing admin account. If no admin exists, the script creates `demo@northstar.test` with password `DemoInventory!2026` (override with `DEMO_ADMIN_PASSWORD` in `backend/.env`) and prints the password. Change it after signing in. The seed is disabled in production and skips if `DEMO-` products already exist.
7. Without the seed, the first registered account is an admin. Later public registrations receive the sales role. An admin can update roles through `PATCH /api/users/:id`.

The API refuses to start without `JWT_SECRET`. Keep environment files out of version control and never put secrets in frontend variables.

## Structure

- `frontend/`: Vite, React, React Router, Axios, Recharts, React Hook Form, Lucide icons, custom responsive dark/light UI.
- `backend/src/index.js`: Express app, auth/authorization, REST endpoints, analytics and stock operations.
- `backend/src/models.js`: Mongoose schemas and indexes.
- `backend/src/forecast.js`: transparent deterministic inventory forecast formula.

## API overview

All API routes use the `/api` prefix. Auth routes: `POST /auth/register`, `POST /auth/login`, `GET /auth/me`, `POST /auth/change-password`. Protected resources include `/users`, `/products`, `/categories`, `/suppliers`, `/customers`, `/warehouses`, `/inventory`, `/stock-movements`, `/dashboard`, `/analytics`, `/ai/insights`, `/notifications`, and `/audit-logs`. Send `Authorization: Bearer <JWT>` for protected calls. Product listing accepts `q`, `category`, `page`, and `limit`. Stock adjustment is `POST /inventory/adjust` with product, warehouse, signed delta, reason, and movement type.

Stock quantity is stored per `InventoryBalance` and changed through a transaction that also creates an immutable `StockMovement`. Cost valuation is stock on hand × product cost. Sales trend currently reflects recorded sale and return movement units; it is not financial revenue. Forecasts estimate average daily demand from the last 90 days of sale movement quantities and recommend 30 day coverage plus the minimum stock threshold. Products with fewer than 10 sold units are marked limited confidence. No LLM call is currently implemented; `LLM_API_*` variables are reserved for an optional provider integration and deterministic recommendations remain available.

## Verification

Run backend unit tests with `npm --prefix backend test` and build the frontend with `npm --prefix frontend run build`. Stock adjustment integration requires a MongoDB replica set.

## Current scope

Purchase/sales order lifecycle, transfer workflow, audits, expiry/batch tracking, scanner, CSV import, richer financial reporting, seed fixtures, and a configured LLM assistant remain future work. Their navigation entries are not presented as completed functionality.
