# Share n' Save Automatic Sync

## Provider

Use Supabase. It has a free plan, works from a static PWA, and gives the app a small Postgres-backed JSON document for sync. No Google Drive is needed.

Reference: https://supabase.com/pricing

The app now supports:
- pull on startup
- push after adding, deleting, importing, restoring, or renaming
- retry when the browser comes back online
- refresh when the tab becomes visible
- polling every 30 seconds
- merge for custom items, custom-item deletions, and category names/colors

## Supabase Setup

1. Create a free Supabase project.
2. Open SQL Editor.
3. Run this SQL:

```sql
create extension if not exists pgcrypto;

create table if not exists public.share_n_save_sync (
  sync_id text primary key,
  access_key_hash text not null,
  payload jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.share_n_save_sync enable row level security;
revoke all on table public.share_n_save_sync from anon, authenticated;

create or replace function public.share_n_save_pull(
  p_sync_id text,
  p_access_key text
)
returns table(payload jsonb, updated_at timestamptz)
language plpgsql
security definer
set search_path = public
as $$
declare
  key_hash text := encode(digest(p_access_key, 'sha256'), 'hex');
begin
  if exists (
    select 1 from public.share_n_save_sync
    where sync_id = p_sync_id and access_key_hash <> key_hash
  ) then
    raise exception 'Invalid sync access key';
  end if;

  return query
  select s.payload, s.updated_at
  from public.share_n_save_sync s
  where s.sync_id = p_sync_id and s.access_key_hash = key_hash;
end;
$$;

create or replace function public.share_n_save_push(
  p_sync_id text,
  p_access_key text,
  p_payload jsonb
)
returns table(payload jsonb, updated_at timestamptz)
language plpgsql
security definer
set search_path = public
as $$
declare
  key_hash text := encode(digest(p_access_key, 'sha256'), 'hex');
begin
  if exists (
    select 1 from public.share_n_save_sync
    where sync_id = p_sync_id and access_key_hash <> key_hash
  ) then
    raise exception 'Invalid sync access key';
  end if;

  insert into public.share_n_save_sync(sync_id, access_key_hash, payload, updated_at)
  values (p_sync_id, key_hash, p_payload, now())
  on conflict (sync_id) do update
  set payload = excluded.payload,
      updated_at = now()
  where public.share_n_save_sync.access_key_hash = key_hash;

  return query
  select s.payload, s.updated_at
  from public.share_n_save_sync s
  where s.sync_id = p_sync_id and s.access_key_hash = key_hash;
end;
$$;

grant execute on function public.share_n_save_pull(text, text) to anon, authenticated;
grant execute on function public.share_n_save_push(text, text, jsonb) to anon, authenticated;
```

4. Go to Project Settings -> API.
5. Copy the Project URL and anon public key.
6. In Share n' Save, click Sync.
7. Paste the Project URL and anon public key.
8. Click Generate code.
9. Click Save sync.
10. On another device, paste the same Project URL, anon key, sync code, and access key.

## How It Works

The app stores one JSON payload per sync code. The access key is hashed in Supabase, and direct table access is revoked from anon users. The PWA calls two database functions:

- `share_n_save_pull`
- `share_n_save_push`

Custom items are merged by `updatedAt`. Custom-item deletions use tombstones, so a deletion on one device removes the same custom item on the next sync. Category names and colors merge by their own `updatedAt` value.

For hidden base items/categories/subcategories, the newer synced snapshot wins during normal sync. In a true simultaneous conflict, hidden sets are merged conservatively.

## Free-Tier Notes

This app stores a tiny JSON document and polls every 30 seconds while open, so it should be comfortably small for a personal free Supabase project. If Supabase changes its free-tier limits later, the app can keep the same sync interface and swap the two RPC calls for another free backend.
