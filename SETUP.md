# Termux Snippet Vault — Setup Guide

## 1. Bikin tabel di Supabase

Buka project Supabase lu (`supabase-lime-queen`) → **SQL Editor** → jalankan ini:

```sql
create table snippets (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  command text not null,
  description text,
  category text not null default 'uncategorized',
  created_at timestamptz default now(),
  updated_at timestamptz default now(),
  user_id uuid references auth.users(id)
);

alter table snippets enable row level security;

create policy "select own snippets"
  on snippets for select
  using (auth.uid() = user_id);

create policy "insert own snippets"
  on snippets for insert
  with check (auth.uid() = user_id);

create policy "update own snippets"
  on snippets for update
  using (auth.uid() = user_id);

create policy "delete own snippets"
  on snippets for delete
  using (auth.uid() = user_id);
```

## 2. Ambil API Key

Di Supabase Dashboard → **Project Settings** → **API**, copy:
- **Project URL**
- **anon public key**

Buka `index.html`, cari baris ini di bagian atas `<script>`:

```js
const SUPABASE_URL = "https://YOUR-PROJECT.supabase.co";
const SUPABASE_ANON_KEY = "YOUR-ANON-PUBLIC-KEY";
```

Ganti dengan punya lu.

## 3. Setting Auth (biar gak publik)

Di Supabase Dashboard → **Authentication** → **Providers** → pastikan **Email** aktif.

Kalau lu mau matiin "email confirmation" (biar langsung bisa login abis daftar tanpa cek email dulu, cocok buat single-user pribadi):
- **Authentication** → **Settings** → matiin **"Confirm email"**

## 4. Deploy

Karena ini plain HTML/JS (gak perlu build step), paling gampang:

**Opsi A — Vercel (disaranin, sejalan sama Supabase integration lu)**
1. Push folder ini ke repo GitHub baru (upload `index.html` lewat GitHub web interface)
2. Di Vercel: **New Project** → import repo itu → deploy (gak perlu ubah build settings apa-apa, karena ini static file)

**Opsi B — GitHub Pages**
1. Upload `index.html` ke repo, aktifin GitHub Pages dari branch itu
2. Selesai, langsung bisa diakses

## 5. Pemakaian

- Buka web-nya → daftar akun (email + password) sekali doang
- Login → tambah snippet lewat tombol **+ SNIPPET**
- Search & filter kategori otomatis muncul dari data yang lu input
- Tombol **COPY** di tiap card langsung copy command ke clipboard, siap paste ke Termux

## Catatan

- Data private per-user berkat Row Level Security — lu login pake akun lain gak bakal keliatan snippet punya lu.
- Kategori bebas teks (gak perlu tabel terpisah) — ketik apa aja pas nambah snippet, otomatis jadi filter chip baru.
