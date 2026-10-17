[banco_joao_castro_cds.sql](https://github.com/user-attachments/files/33607454/banco_joao_castro_cds.sql)
-- DJ JOÃO CASTRO CDs | Supabase PostgreSQL
-- Execute no SQL Editor do seu próprio projeto Supabase.
create extension if not exists pgcrypto;
create table if not exists public.admins (
 user_id uuid primary key references auth.users(id) on delete cascade,
 created_at timestamptz not null default now()
);
create table if not exists public.cds (
 id uuid primary key default gen_random_uuid(),
 title text not null check (length(trim(title))>0),
 dj text not null default 'DJ João Castro',
 genre text not null default 'Remix',
 description text not null default '',
 cover_url text,
 zip_url text,
 created_at timestamptz not null default now(),
 updated_at timestamptz not null default now()
);
create table if not exists public.tracks (
 id uuid primary key default gen_random_uuid(),
 cd_id uuid not null references public.cds(id) on delete cascade,
 title text not null,
 audio_url text not null,
 position integer not null default 0 check (position>=0)
);
create index if not exists tracks_cd_order_idx on public.tracks(cd_id,position);
create index if not exists cds_created_idx on public.cds(created_at desc);
create or replace function public.is_cd_admin() returns boolean language sql stable security definer set search_path='' as $$
 select exists(select 1 from public.admins a where a.user_id=(select auth.uid()));
$$;
revoke all on function public.is_cd_admin() from public;
grant execute on function public.is_cd_admin() to authenticated;
alter table public.admins enable row level security;
alter table public.cds enable row level security;
alter table public.tracks enable row level security;
drop policy if exists "read own admin" on public.admins;
create policy "read own admin" on public.admins for select to authenticated using (user_id=(select auth.uid()));
drop policy if exists "public read cds" on public.cds;
create policy "public read cds" on public.cds for select to anon,authenticated using (true);
drop policy if exists "admin insert cds" on public.cds;
create policy "admin insert cds" on public.cds for insert to authenticated with check (public.is_cd_admin());
drop policy if exists "admin update cds" on public.cds;
create policy "admin update cds" on public.cds for update to authenticated using (public.is_cd_admin()) with check (public.is_cd_admin());
drop policy if exists "admin delete cds" on public.cds;
create policy "admin delete cds" on public.cds for delete to authenticated using (public.is_cd_admin());
drop policy if exists "public read tracks" on public.tracks;
create policy "public read tracks" on public.tracks for select to anon,authenticated using (true);
drop policy if exists "admin insert tracks" on public.tracks;
create policy "admin insert tracks" on public.tracks for insert to authenticated with check (public.is_cd_admin());
drop policy if exists "admin update tracks" on public.tracks;
create policy "admin update tracks" on public.tracks for update to authenticated using (public.is_cd_admin()) with check (public.is_cd_admin());
drop policy if exists "admin delete tracks" on public.tracks;
create policy "admin delete tracks" on public.tracks for delete to authenticated using (public.is_cd_admin());
insert into storage.buckets(id,name,public) values ('cds','cds',true) on conflict(id) do update set public=true;
drop policy if exists "admin upload cd files" on storage.objects;
create policy "admin upload cd files" on storage.objects for insert to authenticated with check (bucket_id='cds' and public.is_cd_admin());
drop policy if exists "admin update cd files" on storage.objects;
create policy "admin update cd files" on storage.objects for update to authenticated using (bucket_id='cds' and public.is_cd_admin()) with check (bucket_id='cds' and public.is_cd_admin());
drop policy if exists "admin delete cd files" on storage.objects;
create policy "admin delete cd files" on storage.objects for delete to authenticated using (bucket_id='cds' and public.is_cd_admin());
-- Depois de criar o usuário no Authentication > Users, copie o UUID e rode:
-- insert into public.admins(user_id) values ('COLE-O-UUID-AQUI') on conflict do nothing;
-- Nunca coloque a senha ou service_role no HTML.
