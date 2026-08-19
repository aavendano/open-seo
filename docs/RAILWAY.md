# Railway Self-Hosting

Deploy OpenSEO from this repo on Railway with **hosted auth** (per-user accounts) and **Postgres**.

## Architecture

```
Internet → OpenSEO (public domain) → Postgres (private)
```

Do not use the marketplace template (`Lukem121/openseo`) for this setup — that template uses `AUTH_MODE=local_noauth` plus an Auth Gate password.

## Services

| Service | Role |
|---------|------|
| **OpenSEO** | This repo, `Dockerfile.selfhost` |
| **Postgres** | Railway Postgres plugin |

## OpenSEO variables

| Variable | Value |
|----------|--------|
| `AUTH_MODE` | `hosted` |
| `BETTER_AUTH_URL` | `https://your-domain.example` |
| `BETTER_AUTH_SECRET` | Random string, 32+ characters |
| `ALLOWED_HOST` | Your public hostname (no `https://`) |
| `DATABASE_PROVIDER` | `postgres` |
| `DATABASE_URL` | `${{Postgres.DATABASE_URL}}` |
| `POSTGRES_DATABASE_URL` | `${{Postgres.DATABASE_URL}}` |
| `CLOUDFLARE_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE` | `${{Postgres.DATABASE_URL}}` |
| `CLOUDFLARE_INCLUDE_PROCESS_ENV` | `true` |
| `DATAFORSEO_API_KEY` | Base64 of `email:password` (see [DATAFORSEO_API_KEY.md](./DATAFORSEO_API_KEY.md)) |
| `BYPASS_EMAIL_VERIFICATION` | `true` (optional; skip email provider during setup) |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Required by preflight for `hosted`; use real OAuth creds or placeholders until Google sign-in is needed |
| `PORT` | `8080` |
| `OPENSEO_TELEMETRY_DISABLED` | `1` (optional) |

## Railway settings

- **Custom domain** on OpenSEO (not Auth Gate)
- **RAM:** 4 GB+ recommended (first start runs migrations + Vite build)
- **Healthcheck:** `/api/health` with timeout **300 s** (`railway.toml` sets this). Railway probes with Host `healthcheck.railway.app`; `vite.config.ts` allows that hostname.
- **Start command:** `sh docker-entrypoint.sh` (default from `railway.toml`)

## First deploy

1. Push this repo to GitHub.
2. In Railway, connect the OpenSEO service to `aavendano/open-seo` (branch `main`).
3. Set variables above.
4. Deploy and wait 2–3 minutes for the first build.
5. Open `/sign-up` to create the first user.

## Notes

- Postgres migrations run automatically on start when `DATABASE_PROVIDER=postgres`.
- The D1 volume (`/app/.wrangler`) is still used for KV/R2/DO local state in Docker mode; user auth data lives in Postgres when `DATABASE_PROVIDER=postgres`.
