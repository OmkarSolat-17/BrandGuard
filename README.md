# BrandGuard — Supabase Integrated

**Main mobile app: `mobile/` (Flutter). API: `backend/` (FastAPI). Ek hi Flutter root hai.**
Purane delivered Fullstack-Upgrade aur Submission-Fixed ZIPs unchanged hain.

## Ab kya actually working/tested hai?

Naya separate **BrandGuard Supabase project connect aur application-test ho gaya hai**:
PostgreSQL persistence, real Auth, private cloud documents, Creator → Reviewer flow,
versioned rules/approvals, drafts, receipts, analytics/history aur notifications.
Existing **LiFE Mitra Gujarat untouched** hai. Project creation mein Free plan request hua.

**22 direct cloud checks + 55 actual FastAPI workflow checks pass**, including API
process restart ke baad saved data. Secure TLS mein CA aur hostname verification bhi
pass hua. **74 backend tests, 49 Flutter tests aur clean analyzer** is copy par rerun hue.
Details: [Live Supabase status](docs/LIVE_SUPABASE_STATUS.md).

**Abhi pending:** permanent API hosting, actual signup/recovery email delivery and
callbacks, aur real AI-provider access/call. Inko complete claim nahi kiya gaya.
AI disabled hone par fake factual pass nahi aata: explicit abstention milta hai.

**ZIP secrets-free hai.** Workspace mein `backend/.env` privately configured hai,
lekin credentials ZIP mein nahi hain. Apne trusted backend par woh private config
separately transfer karein; certificate/data-directory paths ke liye setup guide dekhein.
Without a completed `.env`, extracted source deliberately **local SQLite/demo mode**
mein chalta hai—downloaded ZIP apne aap live cloud credentials contain nahi karta.
See [Supabase run/setup](docs/SUPABASE_SETUP.md). No runtime DB/user files are bundled.

AI ko explicitly enable kiye bina fake factual pass nahi milega: result `abstained`
hoga. Provider integration is model-assisted evidence comparison, not a guarantee
of truth and not autonomous publication approval. Human review remains necessary.

## Sabse jaldi local run

### 1. Backend

ZIP **extract** karein. Root par Windows mein **`START_BACKEND.bat`** double-click,
ya terminal mein:

```sh
python start_backend.py
```

Python **3.11+** aur first dependency setup ke liye internet chahiye. Launcher
`backend/.venv` banata hai aur pinned requirements install karta hai. Terminal open
rakhein. Check: <http://127.0.0.1:8000/health> — API docs: <http://127.0.0.1:8000/docs>.
SQLite database aur local signing secret first run par `backend/data/` mein bante hain.

### 2. Mobile

```sh
cd mobile
flutter pub get --enforce-lockfile
flutter run
```

Tested SDK family: **Flutter 3.50.0 / Dart 3.14.0**; project requires Dart `^3.14.0`.
Android emulator default API: `http://10.0.2.2:8000`. iOS simulator/desktop:
`http://127.0.0.1:8000`. Physical phone ke liye same Wi-Fi par computer ka LAN IP:

```sh
flutter run --dart-define=API_BASE=http://YOUR_COMPUTER_LAN_IP:8000
```

Phone ka `localhost` computer nahi hota. Firewall mein dev port allow karein.
Production mein reachable **HTTPS API** use karein. Flutter web defaults to its own
origin (deploy behind a same-origin API proxy). Local Chrome development:

```sh
flutter run -d chrome --dart-define=API_BASE=http://127.0.0.1:8000
```

Backend CORS allows localhost development and explicit `CORS_ORIGINS` only.
Server is required **for login too**; public demo values cannot bypass it.

### Local demo credentials

| Role | Email | Password |
|---|---|---|
| Creator | creator@brandguard.demo | Demo@12345 |
| Reviewer | reviewer@brandguard.demo | Demo@12345 |

Passwords are deliberately public **only for local demos**. Server stores PBKDF2
hashes; mobile does not verify a hardcoded password. Do not expose seeded demo
accounts to the internet. Use a new database/project and `SEED_DEMO_USERS=false`
for real deployment. Public signup can create **creator** accounts only.
Reviewer/admin provisioning is a trusted server action, not a client dropdown.

## Working workflow

1. Sign in as Creator. Save a draft, reopen it from **Saved Drafts**, or start a check.
2. Enter brand `Acme` and content such as `TODO free forever`. Check runs server rules.
3. Upload real TXT/Markdown/text-PDF files; optionally insert extracted text into
   the editor. Only the **exact editor text** is scanned. Save the draft to persist
   attachment references. Objects remain private, accessible through authenticated API.
4. Findings include the real AI status and protected processing receipt. A clean
   scan still waits for human review. Applying edits and submitting creates a new
   scanned version; stale content versions are rejected.
5. Sign out. Sign in as Reviewer. Inspect the exact submitted text in **Review Queue**,
   approve/reject, and see actual in-app activity. Self-review is blocked.
6. Reviewer can CRUD **Brand Guidelines**, **Enforceable Rules**, **Fact Sources**.
   Rules change the backend immediately and version snapshots are preserved.
   Existing approvals become stale when their brand rules change.
7. Upload a fact-source document as Reviewer. It starts **unverified**. Review its
   extracted text and explicitly verify it before AI retrieval can use it.
8. Dashboard, Analytics & Reports, Version History, Audit Ledger, Processing
   Receipts and Notifications read actual server data, not local sample totals.
   Reports can be selected/copied; notifications are pull-to-refresh, not push.

## Supabase, AI and administration

- **[Supabase setup](docs/SUPABASE_SETUP.md)**: DB/Auth/Storage/roles/email recovery.
- **[AI choice and limitations](docs/AI_SETUP.md)**: researched configurable default,
  provider opt-in, evidence grounding, privacy and live-test requirements.
- **[Project guide](docs/PROJECT_GUIDE.md)**: folder map, endpoints, backups/security.
- **[Validation](validation/RESULTS.md)**: actual checks and explicit untested areas.
- Backend `.env.example` contains placeholders only. Completed `.env` must stay local.
  **Secret/service-role/AI/database credentials chat mein paste mat karein.**

Trusted server terminal, from `backend/` with its environment activated:

```sh
python manage.py init-db
python manage.py create-user --email reviewer@example.com --name Reviewer --role reviewer
python manage.py backup --output /PRIVATE_PATH/brandguard-backup.json
```

Passwords are prompted, not command-line arguments. Backups contain sensitive
hashes/session data; protect/encrypt them and back up document objects separately.

## Re-run checks

```sh
cd backend
python -m pip install -r requirements-dev.txt
python -m pytest -q
# On a running API, with a fresh isolated demo database:
python smoke_test.py --base-url http://127.0.0.1:8000
```

```sh
cd mobile
flutter analyze
flutter test --concurrency=1
flutter build web --no-wasm-dry-run
```

## Scope / honest limits

- Supabase application integration is live-verified; **permanent API hosting, real
  email callbacks/delivery and live AI are still pending**, not silently counted as done.
- Database rows store keyed JSON payloads transactionally. Domain tables are not
  fully normalized; workflow writes are serialized for correctness, not high throughput.
- Shared library is single-workspace; no per-organization tenancy or realtime sync.
- AI retrieval is lexical overlapping passages, **not embeddings/vector search**.
- Documents: TXT/MD/text PDFs only; 5 MB, 30 PDF pages, first 100,000 extracted
  characters; AI submission limit 12,000 characters. No image/scanned-PDF OCR,
  URL crawling, malware scanning, email/push notification delivery, or background jobs.
- Draft attachments are private stored documents/reference metadata. Files are not
  silently scanned without inserting their extracted text. Removing a reference
  does not delete the original object; object retention needs an admin policy.
- Password-reset email needs configured Supabase SMTP/redirect allowlist. Local
  mode returns an explicit unavailable error instead of pretending to send email.
- Tokens remain in app memory, not plain preferences; app restart requires login.
  Offline logout clears local identity; remote revocation cannot be guaranteed offline.
- Separate “Request Changes” state is not implemented; rejection plus note is used.
- No Android/iOS device/release build certification or production security certification.
