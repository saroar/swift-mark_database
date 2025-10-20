# Swiftmart – Dev Setup (Mac M1)

A fast local stack with:

- **Database**: Postgres (PostGIS-ready)
- **Admin**: Directus (zero-code data model & CRUD)
- **API**: Rust (Actix + SQLx)
- **Frontend**: Next.js

> Status: ✅ You already have DB + Directus running.

---

## 1) Requirements

- Docker Desktop (Apple Silicon / M1)
- Node.js 18+ (for frontend, Directus CLI if you use it)
- Rust (for API)

---

## 2) Services & URLs

- **Directus** (Admin): http://localhost:8055  
  - Default login (from `docker-compose.yml`):
    - **Email:** `admin@swiftmart.local`
    - **Password:** `admin1234`
- **Postgres**: `postgres://postgres:dev@127.0.0.1:5432/swiftmart`  
- **API (Actix)**: http://127.0.0.1:8080  
- **Frontend (Next.js)**: http://localhost:3000

---

## 3) Start / Stop / Logs

From `swiftmart/database`:

```bash
# Start DB + Directus
docker compose up -d

# Stop
docker compose down

# Follow logs (Directus)
docker compose logs -f directus

# Follow logs (Postgres)
docker compose logs -f postgres
```

> **M1 note:** We pin Postgres to an amd64 image for simplicity in dev. This is fine.

---

## 4) First-time Directus Setup

If the default login does not work:
- Run the following command to reset the admin password inside the Directus container:

```bash
docker exec -it swiftmart-directus npx directus users update admin --password "admin1234"
```

Now log in with:
- Email: `admin@swiftmart.com`
- Password: `admin1234`

---

## 5) Create Collections (no SQL)

Open Directus Admin → **Settings → Data Model** → Create collections:

### category
- id (PK, Text)
- name_en (Text)
- icon (Text)

### promo
- id (PK, Text)
- title_en (Text)
- badge_en (Text)
- image (Text/URL)

### product
- id (PK, Text)
- title_en (Text)
- weight (Text)
- price_cents (Integer)
- discount_percent (Integer, nullable)
- kcal (Integer)
- protein (Float)
- fat (Float)
- carbs (Float)
- short_en (Text long)
- long_en (Text long)
- composition_en (Text long)
- shelf_life_en (Text)
- storage_en (Text)
- images (JSON array) or Directus Files
- category_id (Many-to-One → category.id)

### suggestion
- product_id (Many-to-One → product.id)
- suggested_id (Text)
- title_en (Text)
- image (Text/URL)
- Unique constraint: (product_id, suggested_id)

---

## 6) Backend API (Actix + SQLx)

### Env

Create `backend-api/.env`:

```bash
DATABASE_URL=postgres://postgres:dev@127.0.0.1:5432/swiftmart
RUST_LOG=info
PORT=8080
```

### Run API

```bash
cd backend-api
cargo run
```

### Endpoint

- GET `/catalog` → returns `{ categories, promos, products }`

---

## 7) Frontend (Next.js)

### Env

`frontend-web/.env.local`:

```bash
NEXT_PUBLIC_API_BASE=http://127.0.0.1:8080
```

### Run

```bash
cd frontend-web
npm install
npm run dev
```

Open http://localhost:3000

---

## 8) Cheatsheet

- **Reset admin password:**  
  ```bash
  docker exec -it swiftmart-directus npx directus users update admin --password "admin1234"
  ```

- **Check Directus health:**  
  ```bash
  curl http://localhost:8055/server/health
  ```

- **psql into DB:**  
  ```bash
  docker exec -it swiftmart-postgres psql -U postgres -d swiftmart
  ```

- **Change API port:** edit `backend-api/.env` → `PORT=XXXX`

- **Change DB URL:** edit `backend-api/.env` → `DATABASE_URL=...`

---

## 9) Troubleshooting

- **Wrong username/password:** reset with the `npx directus users update ...` command above.
- **`PUBLIC_URL` warning:** add `PUBLIC_URL=http://localhost:8055` in Directus service env.
- **Postgres platform mismatch:** we run `linux/amd64` under emulation for PostGIS compatibility on M1.
- **CORS errors:** ensure Actix uses `Cors::permissive()` in dev.

---


docker exec -i swiftmart-postgres psql -U postgres -d swiftmart < migrations/20250824_create_suggestion.sql

docker exec -it swiftmart-postgres psql -U postgres -d swiftmart


Option 1 — Check from the Web UI
	1.	Log in to your Directus Admin (http://localhost:8055/admin)
	2.	Click your profile avatar → About
	3.	You’ll see something like:



# Re-login to get a fresh token

TOKEN=$(curl -s -X POST 'http://localhost:8055/auth/login' \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@swiftmart.com","password":"admin1234"}' \
  | jq -r '.data.access_token')
echo "TOKEN length: ${#TOKEN}"

# Make that role an admin
curl -i -s -X PATCH "http://localhost:8055/roles/$ADMIN_ROLE" \
  -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"admin": true}'



curl -s 'http://localhost:8055/items/currency?fields=id,code,name,symbol,status' \
  -H "Authorization: Bearer $TOKEN" | jq




docker exec -it swiftmart-postgres psql -U postgres -d swiftmart -c "\d product_price"


curl -s -X POST 'http://localhost:8055/items/product/bulk' \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @/Users/alif/Developer/swiftmart/database/items/product_bulk.json | jq