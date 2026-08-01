# Deploy this project to Vercel → v2.sih.com.np

This copy of the Laravel + Livewire hospital site is **ready to deploy** to a brand-new Vercel
project bound to **`v2.sih.com.np`**. It ships with the full CI/CD workflow and the Vercel
free-stack wiring (Neon Postgres, Vercel Blob, Vercel KV) — you just need to create the Vercel
project and add the environment variables, exactly like we did for `v1.sih.com.np`.

> Everything is already in the repo: `vercel.json`, `api/index.php`, `.vercelignore`, the
> GitHub Actions workflow (`.github/workflows/deploy-vercel.yml`), and the free-stack guide
> (`VERCEL-FREE-STACK.md`).

---

## Step 1 — Push this project to GitHub

The repo is already created: **`https://github.com/uttam-kharel/react-convert-laravel`** (empty).
Push from this folder:

```bash
git init
git add .
git commit -m "Initial commit — Laravel + Livewire hospital site (v2.sih.com.np)"
git branch -M main
git remote add origin https://github.com/uttam-kharel/react-convert-laravel.git
git push -u origin main
```

---

## Step 2 — Create the Vercel project (one-time)

1. Go to **vercel.com → Add New → Project**.
2. Import the **`react-convert-laravel`** repository.
3. Vercel auto-detects nothing (custom PHP runtime) — that's fine, keep the defaults and click **Deploy**.
   - The build is driven by `vercel.json` + `api/index.php` (no build command needed for the PHP runtime;
     the frontend assets are already committed in `public/build`).
4. After the first deploy succeeds, note the project name (e.g. `react-convert-laravel`).

> ⚠️ **Important:** this must be a **new project** — a custom domain can only attach to one
> Vercel project, and `v1.sih.com.np` already belongs to the old project.

---

## Step 3 — Add production environment variables

In **Vercel → Project → Settings → Environment Variables**, add (scope: **Production**):

| Name | Value | Notes |
|---|---|---|
| `APP_URL` | `https://v2.sih.com.np` | used by the app for absolute links |
| `APP_KEY` | copy from local `.env` (or `php artisan key:generate` output) | required by Laravel |
| `APP_ENV` | `production` | |
| `APP_DEBUG` | `false` | |
| `DATABASE_URL` | *your Neon Postgres connection string* | optional — persistent DB (see `VERCEL-FREE-STACK.md`) |
| `BLOB_READ_WRITE_TOKEN` | `vercel_blob_rw_...` | optional — image/CV uploads |
| `KV_URL` | `tls://default:...@...:6379` | optional — cache (TLS required by Upstash) |

Save → **Redeploy**.

---

## Step 4 — Add GitHub repository secrets (for CI/CD)

The workflow deploys on push to `main`, but it needs these secrets:
**GitHub → repo `react-convert-laravel` → Settings → Secrets and variables → Actions:**

| Secret | Value |
|---|---|
| `VERCEL_TOKEN` | Vercel API token (vercel.com → Account → Tokens) |
| `VERCEL_ORG_ID` | `team_...` — same team as the v1 project |
| `VERCEL_PROJECT_ID` | **the NEW project's** id (`prj_...` from Vercel → Project → Settings → General) |
| `DATABASE_URL` *(optional)* | Neon connection string — enables the auto-migration step |

> ⚠️ `VERCEL_PROJECT_ID` must point at the **new** project, not the v1 one.

---

## Step 5 — Bind the custom domain `v2.sih.com.np`

### In Vercel
1. **Vercel → Project → Settings → Domains → Add**.
2. Add `v2.sih.com.np`.
3. Vercel shows the DNS target: **`cname.vercel-dns.com`** (and possibly a TXT record for verification).

### In Cloudflare (your DNS provider for `sih.com.np`)
Add these records:

| Type | Name | Content | Proxy |
|---|---|---|---|
| CNAME | `v2` | `cname.vercel-dns.com` | DNS only (grey cloud) — Vercel manages its own TLS |

- If Vercel asks for a TXT verification record, add it as instructed and delete it once verified.
- Remove any **A record** for `v2` if one exists (Vercel uses CNAME).

> `v1.sih.com.np` already exists as a CNAME to the same target — a CNAME can't coexist with other
> records on the same hostname, but `v1` and `v2` are different hostnames, so both work side by side.

---

## Step 6 — Verify

```bash
curl -I https://v2.sih.com.np/            # expect HTTP/2 200
curl -s  https://v2.sih.com.np/up         # expect {"status":"ok",...}
curl -s  https://v2.sih.com.np/admin/login  # expect 200 (login page)
```

- Push to `main` → GitHub Actions deploys → Vercel aliases `v2.sih.com.np`.
- HTTPS certificate is issued automatically by Vercel once DNS propagates (minutes to a few hours).
- Run `php artisan vercel:setup` locally after adding the free-stack env vars to confirm all three services are wired.

---

## Gotchas

- **Custom domain is one-per-project:** don't reuse the v1 project; create a fresh one.
- **CNAME at apex:** `sih.com.np` (without `v2.`) can't be a plain CNAME (needs an A/ALIAS/CNAME-flattening record). We only bind the `v2` subdomain.
- **`.env` is gitignored** — never commit it. All production config lives in Vercel env vars / GitHub secrets.
- **Committed SQLite** (`database/database.sqlite`) ships with demo data so the site works immediately; once you add `DATABASE_URL`, the app switches to Postgres automatically (see `VERCEL-FREE-STACK.md`).
