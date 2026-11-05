# Have You Eaten

Your Campus. Your Food. Delivered. Food ordering for Nigerian university campuses (launching at UNN).

Stack: Next.js 15, TypeScript, Tailwind, **Neon Postgres**, Paystack, Vercel.

## Setup

1. `npm install` then `cp .env.example .env.local`
2. Neon console > your project > **Connect**: copy the **pooled** connection string into `DATABASE_URL`.
3. The database schema, security rules and seed data in `db/migrations/` are already applied to the
   Neon project. To rebuild elsewhere, run `0001` to `0004` in order in the Neon SQL Editor.
4. `npm run dev`, then open http://localhost:3000/api/health. Expect `{"ok":true,"universities":1,"campuses":3}`.

## How security works
- Signed-in users' queries run through `withUser()` in `src/lib/db.ts`: a transaction as the low-privilege
  `app_user` role with `app.user_id` set. Postgres row-level security then limits every query to that user's rows.
- `sql()` is the trusted owner connection (bypasses RLS). Only payment webhooks, server-side order pricing
  and admin jobs use it.
- Guard triggers block role self-promotion, editing order amounts, confirming unpaid orders and invalid status jumps.

## Login (Stage 3)
- Email + password. Passwords are hashed with scrypt; sessions are random tokens stored only as SHA-256 hashes (30 days, httpOnly cookie).
- 5 failed logins per email (or 30 per IP) in 15 minutes locks login for 15 minutes.
- Not built yet: email verification and password reset (need an email provider, planned before launch).
- Sign up as student, vendor or rider. Admins cannot self-register.

## Marketplace (Stage 4)
- `/marketplace` (campus, search, category, price and availability filters), `/vendors/[slug]`, `/food/[id]`, `/student/favourites`.
- Public pages show only vendors with status `approved`. Those queries use the trusted connection, so each one filters on status explicitly.
- Favourites run through `withUser()`, so row-level security enforces that students only touch their own.
- No vendors exist until Stage 5. To preview the marketplace, run `db/dev/demo_vendors.sql` in the Neon SQL Editor, and `db/dev/remove_demo.sql` to delete it later.
- Cart and ordering come in Stage 6, so the food page has no Add to cart button yet.

## Make yourself admin
Register normally, then run the `update public.profiles set role = 'admin' ...` line at the bottom of `db/migrations/0003_seed.sql` with your email.

## Credentials needed later
| From | Stage | Variable |
|---|---|---|
| Paystack (free test keys) | 7 | `PAYSTACK_SECRET_KEY` |
| Flutterwave (optional) | 7 | `FLUTTERWAVE_SECRET_KEY`, `FLUTTERWAVE_WEBHOOK_HASH` |
| Vercel project settings | 12 | everything in `.env.example` |
