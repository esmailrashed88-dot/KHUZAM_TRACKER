# AL KHUZAM · Delivery Tracker

SUSTRA shop-drawing delivery tracker for the AL KHUZAM project: modules 1–3, buildings Z / A / B, concrete and reinforcement target dates, drawing counts, and what is due now.

## Publish on GitHub Pages

1. Create a new repository (for example `khuzam-tracker`) and upload `index.html` and this `README.md`.
2. Go to **Settings → Pages → Build and deployment**, set Source to **Deploy from a branch**, branch **main**, folder **/ (root)**, then click **Save**.
3. After about a minute the app is live at `https://<your-user>.github.io/khuzam-tracker/`.

All three modules come preloaded from `SUSTRA_KHUZAM_TIMLINE_INTERNAL_22-9-2026.xlsx`:

- **Module 1** has buildings Z, A and B, each with its own dates.
- **Modules 2 and 3** have their own building Z, with the same dates as Module 1 Z. Their buildings A and B are **references to Module 1**: they show Module 1's drawings and status, and a delivery marked in either place counts for both.

You can replace any module's schedule from the app with **Replace schedule (Excel)**. When you do, the A and B references stay in place unless the new file includes those buildings.

## Storage

With no settings, everything is saved **in the browser that is using the app**. Each person and each device has their own copy, so this mode is only for one person on one computer.

To share live data with the team, connect Supabase.

### Connect Supabase (shared, live)

1. In Supabase, open **SQL Editor** and run:

```sql
create table if not exists public.khuzam_store (
  collection text not null,
  id         text not null,
  data       jsonb not null,
  updated_at timestamptz default now(),
  primary key (collection, id)
);

alter table public.khuzam_store enable row level security;

create policy "khuzam read"  on public.khuzam_store for select using (true);
create policy "khuzam write" on public.khuzam_store for insert with check (true);
create policy "khuzam update" on public.khuzam_store for update using (true);

alter publication supabase_realtime add table public.khuzam_store;
```

2. Copy the **Project URL** and the **anon public key** from **Project Settings → API**.
3. Open `index.html`, find `const CONFIG=` near the bottom, and fill in both values:

```js
const CONFIG={SUPABASE_URL:"https://xxxx.supabase.co",SUPABASE_ANON_KEY:"eyJ..."};
```

4. Commit the change. The first time the app opens it writes Module 1 into Supabase, and the badge at the top reads **Live · Shared**.

With these policies, anyone who has the link can read the data and mark deliveries. If you need logins later, the policies can be limited to signed-in users, the same way the IBIMS Task Tracker does it.

## Excel format

The app reads the same layout as the TIMELINE sheet:

- Column A is the level, column B is the element.
- Row 1 has the building name, for example `BUILDING (A)`.
- Row 2 has `CONCRETE DIMENSIONS` or `REINFORCEMENT`.
- Row 3 has `TARGET DATE` followed by `NOTES` or `EXP.NO.DWGS`.

Drawing counts are taken from whichever sheet has an `EXP.NO.DWGS` column. Excel sometimes turns `08/10/2026` into 10 August; the app detects this and corrects it.
