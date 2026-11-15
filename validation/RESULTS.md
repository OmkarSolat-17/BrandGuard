# Validation — BrandGuard Supabase Integrated

Validation date: **2026-11-15**. Separate source/release; the earlier Fullstack-Upgrade
and Submission-Fixed archives are preserved unchanged. No secret runtime configuration
is bundled. See `docs/LIVE_SUPABASE_STATUS.md` for current activation details.

## Executed for this Supabase copy

| Check | Result | Evidence |
|---|---|---|
| Direct real Auth/REST/Storage | **22 passed** | `cloud/live-auth-storage-checks.json` |
| Actual FastAPI → live Supabase workflows | **55 passed**, including real API process restart | `cloud/cloud-workflow-results.json` and phase logs |
| TLS certificate/hostname checks | **2 passed**; `verify-full`, incorrect hostname rejected | `cloud/tls-verification.json` |
| Final remote cleanup read-back | **0** temporary accounts, storage objects, application entities and temporary rule brands | `cloud/cleanup-verification.json` |
| Backend regressions + secure product tests | **74 passed** (36 inherited + 38 secure) | `backend-tests.txt` |
| Flutter tests | **49 passed**, full invocation on this copy | `flutter-tests.txt` |
| Flutter analyzer | **No issues found** | `flutter-analyzer.txt` |
| Flutter dependencies | `pub get --enforce-lockfile` passed | `flutter-pub-get.txt` |

Toolchain: Python 3.13.18, Flutter 3.50.0, Dart 3.14.0. One nonfatal Starlette/httpx
deprecation warning occurred in backend tests. Native device builds are not covered.

The initial legacy regression run inherited a configured runtime data-directory path
and could not locate JSON test seeds. The fixture was corrected to use immutable
bundled seeds and isolate Auth/DB/AI settings from `.env`. The final rerun above passed
all 74 tests with the real cloud `.env` still present; tests did not write cloud data.

## What live application checks prove

- Real server password login for Creator/Reviewer/other Creator, trusted app roles;
  editable user metadata and spoofed demo headers cannot grant privileges.
- Reviewer library/rules/facts CRUD; actual regex scan; exact-text approvals,
  stale-version conflicts, changed-rules invalidation and independent review decisions.
- Real private cloud upload, extraction, owner/reviewer download, cross-user denial.
- SQL-backed draft revisions/attachments, receipt permissions, private analytics and
  history, notification/read state; direct remote SQL read and transaction rollback.
- Actual Uvicorn process stop/restart preserved drafts, attachments, exact content,
  review decision and protected receipt.
- Real refresh/logout and backend token-session denial, revoked-refresh rejection,
  authenticated password update, old-password rejection and new-password login.
- Temporary cloud test accounts/objects/rows, including reviewer library audit events,
  were removed; independent final SQL read-back confirmed cleanup.

The 22 direct component checks additionally cover real private-bucket policies,
anonymous/cross-user denial, wrong-folder upload denial and blocked PostgREST reads.
Test accounts were temporarily admin-confirmed, **not verified by email delivery**.

## Local/mocked vs live distinction

Backend automated tests also cover hashing, local sessions, backup/restore round-trip,
concurrency, malformed/oversized uploads, recovery form contracts and evidence-validation
failure cases. Some Supabase and all AI-provider contract tests are mocks; they are
not used as substitutes for the separate real cloud checks listed above.

Flutter unit/widget tests use mocked HTTP clients for screen/session contracts.
They exercise sign-in, remembered email, refresh/logout races, responsive navigation,
exact-text scanning, server-draft contracts, role controls and evidence presentation.
They do not constitute device/browser end-to-end validation.

The inherited Fullstack source previously passed a **37-check actual local HTTP smoke**
and **JavaScript web build**; its original `http-smoke.txt` and `flutter-web-build.txt`
are retained as inherited evidence, not falsely labeled as new cloud/device runs.
The mobile source is unchanged in this Supabase release.

## Still NOT verified/completed

- Permanent Python API hosting and production domain. Workspace API is temporary.
- Real signup confirmation/recovery delivery, real recipient callbacks, SMTP and
  live recovery-link expiry/reuse. Current remote Site URL/allowlist remain undeployed.
- Any real AI/model request, cost/quality/rate-limit tests or data-residency certification.
  Without provider credentials, factual AI status is honestly `abstained`.
- Cloud backup/restore recovery drill, production load/security audit.
- Embeddings/vector search, OCR, URL crawling, push/email notifications, multi-tenancy.
- APK/IPA, real device picker, browser E2E, Wasm or production-security certification.

## Archive integrity

Release tooling checks CRC, path safety, extracted-byte equality, SHA-256 manifest,
one Flutter root, preserved previous ZIP hashes and absence of real setup secrets.
`.env`, signing secrets, runtime DBs, object uploads, dependencies and generated build
outputs are excluded. The public Supabase CA certificate is intentionally included.
`file-manifest.json` hashes all included files except itself.
