# POS App — Phase 1: Foundation

Authentication, roles (Admin / Manager / Cashier), PostgreSQL, dashboard, and
navigation, built with Next.js 14 (App Router), NextAuth, and Prisma.

## Prerequisites

- Docker + Docker Compose installed
  (that's the only requirement for the steps below — Node doesn't need to be
  installed locally since the app runs inside the container)

## 1. Run it

From the project root:

```bash
docker compose up --build
```

This will:
1. Start PostgreSQL 16 in a container and wait until it's healthy.
2. Build the Next.js app image, install dependencies, generate the Prisma
   client, and build the app.
3. Run `prisma migrate deploy` to create the `users` table.
4. Run the seed script to create three demo accounts (see below).
5. Start the app at **http://localhost:3000**.

First run takes a couple of minutes (image build + npm install). Subsequent
runs are fast.

## 2. Verify login + roles work

Go to http://localhost:3000 — you'll be redirected to `/login`. Sign in with
any of the seeded accounts:

| Role    | Email               | Password     |
|---------|---------------------|--------------|
| Admin   | admin@pos.local     | Admin123!    |
| Manager | manager@pos.local   | Manager123!  |
| Cashier | cashier@pos.local   | Cashier123!  |

Checks to run manually:

- [ ] Log in as each account — you land on `/dashboard`.
- [ ] The sidebar only shows the sections that role is allowed to open
      (Cashier sees "Cashier Tools" only; Manager sees Cashier + Manager
      Tools; Admin sees all three plus Admin Panel).
- [ ] While logged in as **Cashier**, manually visit
      `http://localhost:3000/dashboard/admin` — you should be redirected to
      `/unauthorized`, not shown the page.
- [ ] Click **Log out** — you're returned to `/login` and can no longer load
      `/dashboard` (it redirects back to `/login`).
- [ ] Refresh the page while logged in — session persists (JWT-based, 8h
      expiry).

## 3. Verify the database connection

```bash
docker compose exec db psql -U pos_user -d pos_db -c "SELECT email, role FROM users;"
```

You should see the three seeded users listed with their roles. If this
command returns rows, the app successfully connected to Postgres, ran
migrations, and seeded data — no connection errors.

You can also check the app container logs for the migration/seed step:

```bash
docker compose logs app | grep -i seed
```

## 4. Stopping / resetting

```bash
docker compose down          # stop containers, keep DB data
docker compose down -v       # stop containers AND wipe DB data (fresh start)
```

## Running without Docker for the app (optional)

If you'd rather run `npm run dev` directly on your machine against the
dockerized Postgres:

```bash
docker compose up db -d      # just the database
cp .env.example .env
npm install
npx prisma migrate dev
npx tsx prisma/seed.ts
npm run dev
```

## What's included in this phase

- **Auth**: NextAuth (Credentials provider), bcrypt-hashed passwords, JWT
  sessions, `/login` page, logout button.
- **Roles**: `Role` enum in Postgres (`ADMIN`, `MANAGER`, `CASHIER`) stored on
  the `User` model, included in the session/JWT.
- **PostgreSQL**: via Prisma ORM, schema in `prisma/schema.prisma`,
  Dockerized with a persisted volume.
- **Dashboard**: shared landing page at `/dashboard` plus role-specific pages
  (`/dashboard/admin`, `/dashboard/manager`, `/dashboard/cashier`).
- **Navigation**: sidebar that adapts to the signed-in user's role.
- **Route protection**: `src/middleware.ts` enforces role rules server-side
  (defined centrally in `src/lib/rbac.ts`) — not just hidden nav links.

## Committing to Git

A local git repo has already been initialized and committed for you (see
chat). To push it to a remote:

```bash
git remote add origin <your-repo-url>
git push -u origin main
```
