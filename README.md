# Ethereal House — Perks and Wellness Hub

A premium two-sided Korean wellness and skincare web application built for the Honduran and Central American market. The customer side guides users through their personalized Korean skincare journey using an AI-powered Package Finder, browses a curated catalog of eighteen real K-Beauty brands, manages a shopping cart, and navigates membership perks. The admin side gives the team a real-time dashboard for revenue, orders, reservations, layaway plans, promotions, and analytics.

Built with Next.js, Supabase, PostHog, and Tailwind CSS. Deployed on Google Cloud Run.

## Live URL

https://ethereal-house-perks-and-sales-hub-144258355603.us-east1.run.app

## Tech Stack

- **Framework:** Next.js 14 (App Router)
- **Database & Auth:** Supabase (PostgreSQL + Supabase Auth)
- **Analytics:** PostHog
- **Styling:** Tailwind CSS
- **Deployment:** Google Cloud Run

## Features

- Soft K-Beauty pink palette with custom lotus flower logo and animated lotus petal background
- Public landing experience with no account required for browsing
- Sign up, log in, log out via Supabase Auth
- Eight-step Package Finder generating personalized Korean skincare protocols
- Full Korean product catalog with eighteen brands including Beauty of Joseon, Anua, Skin1004, Dr Jart, Medicube, Laneige, Cosrx, Numbuzin, and more
- Tiered membership system (Explorer, Member, VIP) with automatic tier upgrades
- WhatsApp ordering as primary checkout for Honduran market
- 48-hour reservation system (Honduran apartar pattern)
- Layaway plan system for premium bundles
- Admin dashboard with revenue tracking, orders, promotions, and customer lookup
- PostHog analytics tracking custom events including added_to_cart, package_finder_completed, perk_claimed, and whatsapp_order_initiated

## Local Setup

### 1. Clone the repository

```bash
git clone https://github.com/reyesa2pba/ETHEREAL-HOUSE.git
cd ETHEREAL-HOUSE
```

### 2. Install dependencies

```bash
npm install
```

### 3. Create environment variables

Create a `.env.local` file at the project root with the following:

```
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
VITE_POSTHOG_KEY=your_posthog_project_api_key
VITE_POSTHOG_HOST=https://us.i.posthog.com
```

### 4. Set up the Supabase database

Go to your Supabase project, open the SQL Editor, and run this query:

```sql
-- Users table
create table if not exists users (
  id uuid references auth.users(id) on delete cascade primary key,
  email text unique not null,
  full_name text,
  role text default 'customer',
  tier text default 'Explorer',
  points integer default 0,
  total_spend numeric default 0,
  delivery_zone text,
  currency_preference text default 'USD',
  referral_code text unique,
  created_at timestamp with time zone default now()
);

-- Cart items
create table if not exists cart_items (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id) on delete cascade not null,
  product_name text not null,
  product_brand text,
  product_price numeric,
  quantity integer default 1,
  created_at timestamp with time zone default now()
);

-- Saved packages
create table if not exists saved_packages (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id) on delete cascade not null,
  package_title text not null,
  products jsonb,
  total_cost numeric,
  created_at timestamp with time zone default now()
);

-- Enable Row Level Security
alter table users enable row level security;
alter table cart_items enable row level security;
alter table saved_packages enable row level security;

-- Policies
create policy "Users read own data" on users for select using (auth.uid() = id);
create policy "Users update own data" on users for update using (auth.uid() = id);
create policy "Users insert own data" on users for insert with check (auth.uid() = id);
create policy "Users manage own cart" on cart_items for all using (auth.uid() = user_id);
create policy "Users manage own packages" on saved_packages for all using (auth.uid() = user_id);
```

### 5. Run the development server

```bash
npm run dev
```

Open http://localhost:3000 in your browser.

## Creating an Admin Account

1. Sign up a normal account through the app interface
2. Open the Supabase Table Editor
3. Navigate to the `users` table
4. Find your account row
5. Change the `role` field from `customer` to `admin`
6. Refresh the app and log in. You will see the admin dashboard.

## PostHog Events Tracked

- `pageview` and `pageleave` (automatic)
- `added_to_cart` (product_id, brand, price)
- `removed_from_cart` (product_id)
- `package_finder_completed` (skin_type, concerns, routine_level, budget, recommended_package_title)
- `perk_claimed` (tier, promotion_id)
- `whatsapp_order_initiated` (source, item_id)
- `tier_upgraded` (old_tier, new_tier)
- `reservation_created` (item_id)
- `wishlist_added` (product_id)

## Feature Cut from Scope

The AI-powered multispectral skin diagnostic camera kiosk and real-time inventory synchronization with Korean supplier APIs were intentionally descoped from this MVP. These features were cut because they require external hardware integration, formal partnership agreements with Tier-1 Korean laboratories, and a cold-chain logistics layer that exceeded the timeline for a single-week MVP build. The quiz-based Package Finder captures the same diagnostic intent and produces equivalent personalized output, and the manual WhatsApp ordering flow handles the transaction layer effectively for the Honduran market where WhatsApp is the dominant commerce channel. Both features remain documented as Phase 2 roadmap items.

## Disclaimer

All Korean product and brand information in this catalog is sourced from public brand research and presented as a mockup catalog representing the future live inventory of Ethereal House. This MVP was built as an academic project for BUS-5703 at Palm Beach Atlantic University.

## Built With

- Google AI Studio (initial scaffold)
- Claude (architecture, business logic, and documentation)
- Supabase, PostHog, Vercel, Tailwind
