# Supabase — activated project and deployment handoff

**BrandGuard's live integration is now verified.** See
[LIVE_SUPABASE_STATUS.md](LIVE_SUPABASE_STATUS.md) for the exact checks and remaining
hosting/email/AI limits. The existing LiFE Mitra Gujarat project was not modified.

## Existing project — do not create another one

- Ref: `jtidayzktuzpjydshsva`
- API: `https://jtidayzktuzpjydshsva.supabase.co`
- Dashboard: https://supabase.com/dashboard/project/jtidayzktuzpjydshsva
- Session pooler: `aws-1-ap-south-1.pooler.supabase.com:5432`
- Database: `postgres`; username: `postgres.jtidayzktuzpjydshsva`
- Core/storage migrations already applied and read back. No reapplication is needed
  for this project. Supplied SQL files document provisioning for a separate new environment.

## 1. Private backend environment

The workspace backend has its completed `.env`, mode 0600. It is intentionally
excluded from releases, together with database passwords and setup keys.
Copy it separately only to your trusted API server, or configure equivalent secrets
in the host's secret manager. **Do not put them in mobile or paste them into chat.**
The project URL/publishable key alone are not a database connection or a deployed API.

For a new trusted environment, use `backend/.env.example` and these settings:

```dotenv
AUTH_PROVIDER=supabase
SEED_DEMO_USERS=false
SUPABASE_URL=https://jtidayzktuzpjydshsva.supabase.co
SUPABASE_PUBLISHABLE_KEY=YOUR_PROJECT_PUBLISHABLE_KEY
DATABASE_URL=postgresql+psycopg://postgres.jtidayzktuzpjydshsva:URL_ENCODED_DB_PASSWORD@aws-1-ap-south-1.pooler.supabase.com:5432/postgres?sslmode=verify-full&sslrootcert=certs/supabase-prod-ca-2021.crt&connect_timeout=10
SUPABASE_STORAGE_BUCKET=brandguard-files
AI_PROVIDER=disabled
```

Percent-encode special password characters. Use the included PUBLIC CA file; see
`backend/certs/README.md` for provenance/fingerprints. The relative path above works
when launched from `backend/` or through the root launcher. For another working
directory/container, set an absolute **server-local** certificate path. Do not downgrade
to `sslmode=require` or disable certificate checks to work around trust problems.

Normal runtime needs **no service/admin key**. The database role is a trusted
backend credential; per-user application authorization is enforced in FastAPI.
Backend domain tables are intentionally unavailable to mobile/PostgREST users.

## 2. Run backend and mobile

```sh
# Extract the ZIP; restore the private backend/.env securely, then:
python start_backend.py
```

Or, with pinned dependencies installed:

```sh
cd backend
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 1
```

Check `/health`. Sign in with a real cloud account, then inspect `/api/v1/system/status`:
`auth_provider=supabase`, `database=postgresql`, `storage=supabase`.
The cloud project has **no public demo credentials**. Local demo logins only apply
when using the default local SQLite mode, not this live cloud configuration.

```sh
cd mobile
flutter pub get --enforce-lockfile
flutter run --dart-define=API_BASE=https://YOUR_PERMANENT_API_DOMAIN
```

For a phone on the same Wi-Fi during development, use the computer's reachable LAN
IP and development port instead. A phone's localhost is not the computer. For web,
prefer a same-origin API reverse proxy, or configure the explicit frontend origin in
`CORS_ORIGINS`. The workspace live preview is temporary, not permanent hosting.

## 3. Roles and private storage

Public signup always results in a Creator; client-editable `user_metadata` is not
trusted for role assignment. Confirmation remains required. Reviewers/admins are
assigned only through trusted dashboard/Admin API **app_metadata**:

```json
{"brandguard_role":"reviewer"}
```

Allowed application roles: creator, reviewer, admin. Supabase's own `role=authenticated`
is a different field. Do not create publicly shared cloud demo accounts.

One-time trusted administration, if needed:

```sh
cd backend
python manage.py create-user --email reviewer@YOUR_DOMAIN --name Reviewer --role reviewer
```

This command additionally needs a temporary server-only `SUPABASE_SECRET_KEY` or
`SUPABASE_SERVICE_ROLE_KEY`. It prompts for a password, creates an email-confirmed
account, and sets trusted metadata. Remove the admin key from normal runtime afterwards.
Dashboard-created reviewers should log in once so notification recipients are known.

`brandguard-files` is private. User bearers, not a service key, authorize actual
Storage requests. Keys are `user_id/random_uuid.ext`; owner upload and owner-or-trusted-
reviewer reads were tested. Direct table REST reads, anonymous file access and other-
creator reads were denied. No client overwrite/delete grant is provided.

## 4. Permanent hosting and real email — still required

Supabase hosts PostgreSQL/Auth/Storage, **not this FastAPI Python server**. Deploy the
backend on a controlled Python/container host with TLS and secrets. Include the public
CA file, set `CORS_ORIGINS` for your frontend and use the reachable HTTPS API in Flutter.
Do not expose seeded local demo mode publicly. Configure request limits, operational
monitoring and backups without logging tokens, credentials or document content.

After obtaining the permanent API domain:

```dotenv
AUTH_REDIRECT_URL=https://YOUR_PERMANENT_API_DOMAIN/auth/reset
CORS_ORIGINS=https://YOUR_FRONTEND_DOMAIN
```

Set Supabase Authentication → URL Configuration:
- Site URL: an actual controlled frontend/confirmation page.
- Redirect URLs: the exact `AUTH_REDIRECT_URL` above.

Current remote Site URL is still localhost:3000 and its allowlist is empty; these
were not replaced with a fabricated or temporary production URL. Real confirmation
and recovery emails have **not** been sent/validated. Configure suitable SMTP/delivery
and test a real recipient, confirmation link, recovery link, expiry and reuse.

The supplied `/auth/reset` form implements Supabase's default implicit recovery
flow, accepting the access token from the URL fragment, removing it from the address
bar and holding it only in memory. It submits to `PUT /api/v1/auth/password`.
Custom PKCE/code/token_hash templates require an additional exchange handler and
are not claimed implemented. Authenticated password update itself passed live tests.

Logout through FastAPI persists a token/session denylist, including older JWTs in
the same session, and revokes the provider refresh token. A previously issued JWT
may still work directly with Supabase until expiry outside this backend. Offline
logout cannot guarantee remote revocation; do not promise otherwise.

## 5. Maintenance and validation

`validation/cloud/` contains sanitized evidence: 22 direct component checks,
55 actual application workflow checks including an API restart, TLS verification
and final temporary-data cleanup. Test users were admin-confirmed; that does not
prove real signup email delivery. AI requests remain disabled without provider access.

Local automated tests are isolated from the completed `.env`; run `python -m pytest -q`
from `backend/`. **Do not run seeded `smoke_test.py` against this live project.**
Back up database records and private Storage objects separately. The local backup/
restore round-trip was tested; a live cloud recovery drill is still pending. Backups
contain sensitive user/session data and must be protected.

References:
- https://supabase.com/docs/guides/platform/ssl-enforcement
- https://supabase.com/docs/guides/auth/users
- https://supabase.com/docs/guides/auth/passwords
- https://supabase.com/docs/guides/auth/signout
- https://supabase.com/docs/guides/storage/security/access-control
