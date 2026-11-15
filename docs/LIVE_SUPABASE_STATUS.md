# BrandGuard — live Supabase verification

Validated in this workspace on **2026-11-15**. This is the new, separate BrandGuard
project; the existing **LiFE Mitra Gujarat project was not modified**.

| Item | Verified state |
|---|---|
| Project | BrandGuard, `jtidayzktuzpjydshsva` |
| Region | Mumbai, `ap-south-1`; creation requested the Free plan |
| Project API | `https://jtidayzktuzpjydshsva.supabase.co` |
| Session pooler | `aws-1-ap-south-1.pooler.supabase.com:5432` |
| Database / login | `postgres` / `postgres.jtidayzktuzpjydshsva` |
| Database transport | `sslmode=verify-full`, official Supabase public root CA |
| Backend runtime | FastAPI → actual PostgreSQL + Supabase Auth/Storage |

Dashboard: https://supabase.com/dashboard/project/jtidayzktuzpjydshsva

## Provisioned and read back

- `brandguard_initial_core`, migration history **20261115192757**.
- `brandguard_private_storage`, migration history **20261115192825**.
- `bg_entities` / `bg_documents`: RLS enabled, no direct anon/authenticated SELECT.
- `brandguard-files`: private, 5 MB limit, TXT/Markdown/PDF MIME allowlist.
- Owner-folder uploads; owner or trusted reviewer/admin downloads.
- Email/password and signup enabled, email confirmation retained, access JWT lifetime
  1,800 seconds and refresh-token rotation enabled.
- No public demo accounts were seeded into this cloud project.

## Actual tests, not just configuration

**22 direct Auth/REST/Storage checks passed.** Temporary creator/reviewer/other-creator
accounts signed in; identities, owner/reviewer downloads, anonymous/cross-user denials,
wrong-folder upload denial, direct table REST denial and refresh/logout were tested.

**55 FastAPI application checks passed against live Supabase.** Coverage:

- Real password login and trusted application roles; forged ID headers rejected;
  client-editable metadata did not elevate privileges.
- Reviewer-managed shared guidelines, fact sources and versioned enforceable rules.
- Actual cloud document upload/extraction/download, reviewer access and other-user denial.
- SQL-backed drafts/attachments, revision conflicts and private history/analytics.
- Real scan, independent human approval/rejection, exact-content verification,
  changed-rules approval invalidation and stale-version rejection.
- Decision notifications/read state; explicit AI abstention without provider access.
- Independent remote SQL read and transaction rollback.
- **Actual API process stopped and restarted**: draft revisions/attachments, exact
  submitted text, review decision and protected receipt remained present.
- Refresh, logout denial of both refreshed and older JWTs in the same app session,
  revoked-refresh rejection, authenticated password change and old/new password login.
- Temporary users, storage objects, application records and test audit events removed.

**2 TLS checks passed:** the real pooler certificate chain/hostname was verified and
libpq reported TLS in use; an incorrect hostname aimed at the same IP was rejected.
The CA is public, downloaded over HTTPS from the official download location, not
blindly copied from the pooler's presented certificate chain.

Sanitized evidence: `validation/cloud/`. The workspace has the detailed validation
harness/logs in `/home/user/brandguard-supabase-validation/` (not needed at runtime).
Admin-confirmed temporary accounts were used: **these are not email-delivery tests**.

## Runtime and secret handoff

This workspace's configured backend is `BrandGuard-Supabase/backend/`. Its private
`backend/.env` uses only the project publishable key, database credential and runtime
settings. The setup-only service/admin key is **not required by normal runtime**.
The completed `.env`, setup credentials and user tokens are deliberately **not in
this ZIP**. Transfer private configuration only to a trusted backend/secrets manager;
never add it to Flutter, Git or a public URL.

The included `backend/certs/supabase-prod-ca-2021.crt` is a **public certificate**,
not a private key. Provenance and fingerprints are in `backend/certs/README.md`.

The live workspace API/preview is a temporary development process, **not permanent
hosting**. Local and device setup is described in the root README.

## Still pending — not claimed complete

1. **Permanent FastAPI hosting / reachable HTTPS `API_BASE`.** Supabase does not
   automatically deploy this Python API.
2. **Real email confirmation and recovery delivery/callbacks.** Site URL is still
   `http://localhost:3000`, redirect allowlist is empty, and no custom SMTP or real
   recipient delivery has been validated. Keep confirmation enabled. Set the real
   controlled Site URL and exact `/auth/reset` allowlist after permanent hosting.
3. **Real AI-provider access and quality validation.** No provider key or live model
   request exists. AI stays disabled/abstained; deterministic rules and human review work.
4. Cloud backup/restore recovery drill, load/security audit and native device/release
   builds. Local backup/restore and mocked provider contracts are not substitutes.

This is verified Supabase **application integration**, not a claim that every
production, email or AI requirement has been completed.
