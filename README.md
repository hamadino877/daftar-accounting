# دفتر | DAFTER — Cloud-ready accounting ledger

Arabic RTL bookkeeping MVP. After cloud configuration, each user account has an isolated data row stored in Supabase and synchronized between devices.


## 1) Supabase project setup
1. Create a Supabase project at https://supabase.com/dashboard.
2. Open SQL Editor and run the SQL below.
3. Open Project Settings / API (or Connect panel) and copy the Project URL and publishable/anon key.
4. Edit `cloud-config.js` and replace `YOUR_SUPABASE_PROJECT_URL` and `YOUR_SUPABASE_PUBLISHABLE_OR_ANON_KEY` placeholders.
5. Auth > Providers > Email: enable Email. For simpler initial testing you may disable email confirmation; for real users, keep confirmation enabled and configure SMTP.


## SQL schema (run in Supabase SQL Editor)
```sql
create table if not exists public.daftar_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);
alter table public.daftar_data enable row level security;
revoke all on public.daftar_data from anon;
grant select, insert, update on public.daftar_data to authenticated;
drop policy if exists "Users can read own Daftar data" on public.daftar_data;
drop policy if exists "Users can insert own Daftar data" on public.daftar_data;
drop policy if exists "Users can update own Daftar data" on public.daftar_data;
create policy "Users can read own Daftar data" on public.daftar_data for select to authenticated using (auth.uid() = user_id);
create policy "Users can insert own Daftar data" on public.daftar_data for insert to authenticated with check (auth.uid() = user_id);
create policy "Users can update own Daftar data" on public.daftar_data for update to authenticated using (auth.uid() = user_id) with check (auth.uid() = user_id);
```


## 2) Run locally
Open terminal inside this folder:
```bash
python -m http.server 8000
```
Then open http://localhost:8000.

## 3) GitHub upload
- Create a new private or public GitHub repository.
- Upload/commit the files in this folder at the repository root (`index.html`, `app.js`, `style.css`, `cloud-config.js`, `README.md`).
- Never commit a service-role/secret key. The publishable/anon key is intended for client use with proper RLS policies.

## 4) Vercel deployment
- In Vercel, Add New > Project and import your GitHub repository.
- Framework preset: Other; build command blank; output directory `.` (root).
- Deploy. On each push to the connected production branch, Vercel rebuilds/deploys automatically.
- Test registration/login, add test product, reload, sign in on another device/browser and confirm the data appears.

## Notes / limitations
- Data stored as one JSONB document per user (simple single-user account MVP). Concurrent editing from multiple devices may overwrite recent changes; avoid simultaneous use until transaction/locking architecture is added.
- A client who creates their own account gets a separate private ledger. Do not share a single account password between customers.
- Before using for real accounting, test every purchase/sale/payment/return path, secure email confirmations, add audit logging and periodic backups. Cloud persistence is not a substitute for backups.
- `cloud-config.js` must contain the project URL and publishable key only; never put the Supabase service role/secret key in frontend code.
