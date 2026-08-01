# Deploy this project to Vercel → v2.sih.com.np

> ✅ **Status (2026-08-02): deployed & mostly live.** The Vercel project `react-convert-laravel`
> is created, deployed to **https://react-convert-laravel.vercel.app**, env vars + GitHub secrets
> are set, and the domain `v2.sih.com.np` is attached & verified. **Only step 5 (Cloudflare CNAME)
> remains** — once added, `https://v2.sih.com.np` serves this project automatically.
> Everything below is the complete record of what was done.

This copy of the Laravel + Livewire hospital site is **deployed** to a brand-new Vercel
project bound to **`v2.sih.com.np`**. It ships with the full CI/CD workflow and the Vercel
free-stack wiring (Neon Postgres, Vercel Blob, Vercel KV) — you just need to create the Vercel
project and add the environment variables, exactly like we did for `v1.sih.com.np`.

> Everything is already in the repo: `vercel.json`, `api/index.php`, `.vercelignore`, the
> GitHub Actions workflow (`.github/workflows/deploy-vercel.yml`), and the free-stack guide
> (`VERCEL-FREE-STACK.md`).

---

## Step 1 — Push this project to GitHub ✅ done

The repo is **already created and pushed**: **`https://github.com/uttam-kharel/react-convert-laravel`**
(branch `main`, initial commit `3e7f46d`). If you ever need to redo it:

```bash
git init
git add .
git commit -m "Initial commit — Laravel + Livewire hospital site (v2.sih.com.np)"
git branch -M main
git remote add origin https://github.com/uttam-kharel/react-convert-laravel.git
git push -u origin main
```

---

## Step 2 — Create the Vercel project ✅ done

**Created via the Vercel API:** project `react-convert-laravel`
(`prj_Jn68MIQIDMy31sApwUsp69wx44xs`, team `team_TKO4eklVKvpy5hlk9KtncXoD`).

```bash
TOKEN=$(node -e "console.log(JSON.parse(require('fs').readFileSync(process.env.HOME+'/.local/share/com.vercel.cli/auth.json','utf8')).token)")
curl -s -X POST -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"name":"react-convert-laravel","framework":null}' \
  "https://api.vercel.com/v9/projects?teamId=team_TKO4eklVKvpy5hlk9KtncXoD"
```

> ⚠️ **Build command must be empty** (Vercel's default `npm run build` fails because `.vercelignore`
> excludes `vendor/`, which the Vite Livewire import needs). v1 has an empty build command too.
> Set it via the API: `curl -X PATCH -d '{"buildCommand":""}' https://api.vercel.com/v9/projects/<id>?teamId=<team>`.

Manual alternative: **vercel.com → Add New → Project → import `react-convert-laravel` → Deploy**,
then Settings → Build & Development → clear the Build Command.

---

## Step 3 — Add production environment variables ✅ done

**Set on the new project via the Vercel API** (all production-scoped, encrypted):
`APP_URL=https://v2.sih.com.np`, `APP_KEY=<from local .env>`, `APP_ENV=production`, `APP_DEBUG=false`,
`SESSION_DRIVER=cookie`, `CACHE_STORE=array`, `QUEUE_CONNECTION=sync`, `LOG_CHANNEL=stderr`.

> 🔑 **The critical fix:** `SESSION_DRIVER` must be **`cookie`** (not `database`). On Vercel's
> read-only serverless filesystem the committed `database.sqlite` can be **read** but **not written**,
> so `SESSION_DRIVER=database` 500s every request with `attempt to write a readonly database`.
> `cookie` sessions persist in the browser and never touch the filesystem.

| Name | Value | Notes |
|---|---|---|
| `APP_URL` | `https://v2.sih.com.np` | used by the app for absolute links |
| `APP_KEY` | copy from local `.env` (or `php artisan key:generate` output) | required by Laravel |
| `APP_ENV` | `production` | |
| `APP_DEBUG` | `false` | |
| `SESSION_DRIVER` | `cookie` | **serverless-safe sessions** — do NOT use `database` |
| `CACHE_STORE` | `array` | no persistent cache writes |
| `QUEUE_CONNECTION` | `sync` | jobs run inline |
| `LOG_CHANNEL` | `stderr` | logs appear in Vercel Function Logs |
| `DATABASE_URL` | *your Neon Postgres connection string* | optional — persistent DB (see `VERCEL-FREE-STACK.md`) |
| `BLOB_READ_WRITE_TOKEN` | `vercel_blob_rw_...` | optional — image/CV uploads |
| `KV_URL` | `tls://default:...@...:6379` | optional — cache (TLS required by Upstash) |

Save → **Redeploy**.

---

## Step 4 — Add GitHub repository secrets (for CI/CD) ✅ done

All three are **already set on the repo** via the GitHub API (HTTP 201):

| Secret | Value (set) |
|---|---|
| `VERCEL_TOKEN` | the Vercel API token |
| `VERCEL_ORG_ID` | `team_TKO4eklVKvpy5hlk9KtncXoD` |
| `VERCEL_PROJECT_ID` | `prj_Jn68MIQIDMy31sApwUsp69wx44xs` |
| `DATABASE_URL` *(optional)* | add later — enables the auto-migration step |

> ⚠️ `VERCEL_PROJECT_ID` must point at the **new** project, not the v1 one.

---

## Step 5 — Bind the custom domain `v2.sih.com.np` ✅ Vercel side done — ⏳ Cloudflare record pending

### In Vercel ✅ done (via API, 2026-08-02)
The domain was previously left over on the **v1 project** (the old alias). It was removed from v1
(`DELETE /v9/projects/livewire-app/domains/v2.sih.com.np` → `{}` = success) and attached to this
project: `POST /v10/projects/<id>/domains` → **verified: true**.

### In Cloudflare (your DNS provider for `sih.com.np`) — ⏳ **do this now**
Add these records:

| Type | Name | Content | Proxy |
|---|---|---|---|
| CNAME | `v2` | `cname.vercel-dns.com` | DNS only (grey cloud) — Vercel manages its own TLS |

- If Vercel asks for a TXT verification record, add it as instructed and delete it once verified.
- Remove any **A record** for `v2` if one exists (Vercel uses CNAME).

> `v1.sih.com.np` already exists as a CNAME to the same target — a CNAME can't coexist with other
> records on the same hostname, but `v1` and `v2` are different hostnames, so both work side by side.

---

## Step 6 — Verify ✅ (main URL live; custom domain after step 5)

```bash
curl -I https://react-convert-laravel.vercel.app/   # HTTP/2 200 — verified 2026-08-02
curl -s  https://react-convert-laravel.vercel.app/up  # {"status":"ok",...}
curl -s  https://react-convert-laravel.vercel.app/admin/login  # 200 (login page)
# after the Cloudflare CNAME:
curl -I https://v2.sih.com.np/                      # HTTP/2 200
```

- Push to `main` → GitHub Actions deploys → Vercel aliases `v2.sih.com.np` (already attached).
- HTTPS certificate is issued automatically by Vercel once DNS propagates (minutes to a few hours).
- Run `php artisan vercel:setup` locally after adding the free-stack env vars to confirm all three services are wired.

---

## Gotchas

- **Custom domain is one-per-project:** don't reuse the v1 project; create a fresh one.
- **CNAME at apex:** `sih.com.np` (without `v2.`) can't be a plain CNAME (needs an A/ALIAS/CNAME-flattening record). We only bind the `v2` subdomain.
- **`.env` is gitignored** — never commit it. All production config lives in Vercel env vars / GitHub secrets.
- **Committed SQLite** (`database/database.sqlite`) ships with demo data so the site works immediately; once you add `DATABASE_URL`, the app switches to Postgres automatically (see `VERCEL-FREE-STACK.md`).
