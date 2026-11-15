# AI setup and researched default

## Decision made for this project

Default configured model: **OpenAI `gpt-5.4-mini`**, changeable through `AI_MODEL`.
It is a documented model with Chat Completions and structured-output support:
https://developers.openai.com/api/docs/models/gpt-5.4-mini . The older `gpt-4.1-mini`
and `gpt-5-mini` pages were also checked; they displayed deprecation labels, so
neither was kept as this upgrade's default.

The integration uses the documented strict JSON-schema response format, but still
validates source IDs, quotations, exact claim spans and status consistency on the
server. Schema compliance does not establish truth or remove the need for review.
Reference: https://developers.openai.com/api/docs/guides/structured-outputs .

**No provider account/key was available. Real model quality, latency, availability,
billing and a live request have NOT been verified.** Contract tests use mocks.
This is a selected configurable default, not a claim of measured model superiority.

## Activation — trusted backend environment only

```dotenv
AI_PROVIDER=openai
AI_API_KEY=YOUR_OWN_SERVER_ONLY_PROVIDER_KEY
AI_BASE_URL=https://api.openai.com/v1
AI_MODEL=gpt-5.4-mini
```

Restart backend after changing environment. Set account spending/rate limits before
using real content. Do not put these secrets in Flutter `--dart-define`, source,
commits, screenshots, ZIPs or chat. Supabase authentication/database access does not
supply an AI provider key or billing account.

## Actual retrieval and comparison flow

1. Reviewer adds manual source text or uploads TXT/MD/text PDF in **Fact Sources**.
2. Uploaded text starts **unverified**. Reviewer inspects it and checks verification.
3. On each check, API filters enabled, reviewer-verified facts for the selected brand.
4. It splits source text into overlapping 3,000-character passages with a 2,700
   character stride, ranks lexical word overlap and retrieves at most five passages.
   Retrieval covers later document sections too; there is no embedding/vector service.
5. Only if provider opt-in/key and relevant evidence exist, API submits the exact
   content (up to 12,000 characters) plus retrieved passages to the model.
6. Strict response schema requires supported/needs_review/abstained, findings and
   source quotations. Server rejects unknown passage IDs, fabricated quotes, claim
   strings not in submitted content, inconsistent verdicts, refusals and incomplete output.
7. Findings/receipts preserve cited fact IDs, exact source quotes, passage positions
   and SHA-256 of the source text. Creator sees evidence and human reviewer decides.

`status=supported` is a **model assessment against retrieved evidence**, not a proof
that every possible claim is correct. Lexical retrieval can miss relevant facts;
source trust/recency and semantic judgments still require people. Shared fact edits
are audited, but do not have the same automatic approval-invalidation contract as
versioned regex-rule edits. A stored source hash records what was compared.

## Explicit abstention

No relevant verified evidence, disabled provider, missing key, oversized AI input,
provider timeout/failure/refusal, invalid citations or malformed output → **abstained**.
Deterministic checks still run. Zero deterministic findings plus AI abstention must
never be presented as factual verification or automatic approval.

## Privacy, cost and deployment boundaries

- AI is disabled by default; enabling it is an explicit decision to send submitted
  text and selected source passages to your configured external provider.
- Request asks for `store: false`; that is **not** a promise of zero provider logs,
  zero retention or a specific legal/data-residency arrangement. Review provider
  terms and your organization's policy before using confidential/customer content.
- Calls have a 25-second timeout and 3,000 output-token cap. There is no background
  queue, batch processing, caching or spend dashboard in this source.
- Workflow database writes are currently serialized, including scan processing;
  use a job queue/optimistic finalization design before high-volume deployment.
- No web search, URL crawling, independent source authentication or scanned-image
  OCR is silently substituted. URLs entered in the library are source metadata.

## Live acceptance checklist

After securely configuring your own key and project:
1. Add and verify an authoritative source containing a known statement/number.
2. Check matching text, contradictory text and unrelated/unsupported claims.
3. Confirm real output includes exact source quotations and reasonable assessments.
4. Remove/disable evidence and key separately: both must abstain, not pass.
5. Simulate provider timeout/refusal: deterministic scan must remain available.
6. Check provider usage/costs, privacy settings and human-review behavior.

Do not label the AI feature live-complete until those checks pass with real access.
