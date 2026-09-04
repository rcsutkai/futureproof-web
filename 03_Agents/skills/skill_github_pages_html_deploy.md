---
name: github-pages-deploy
description: Publish, update or unpublish a single-file HTML page on futureproof's GitHub Pages site (prezentacia.fproof.eu, repo rcsutkai/futureproof-web). Use whenever the user asks to publish, deploy, put online, update or take down a presentation, onboarding page, guide or any HTML page — trigger phrases include "publish this page", "deploy to prezentacia", "put this on GitHub Pages", "unpublish", "take the page down". Covers both /presentations/ and /onboarding/ path families, futureproof page standards (noindex, no analytics, no client identifiers), the 60KB MCP push limit, robots.txt, CNAME protection and README logging.
---

# Deploy HTML page → GitHub Pages (futureproof)

**Version:** 1.4 · **Owner:** futureproof s.r.o. / Roman Csutkai · **Updated:** 2026-09-01
**Canonical home:** `Vault\03_Agents\skills\skill_github_pages_html_deploy.md` — this Claude Skill and any Cowork copy are mirrors; edit the canonical file first, then re-sync.

## Version history
| v | Date | Changes |
|---|---|---|
| 1.4 | 2026-09-01 | Added /onboarding/ path family. Named default execution mode (direct GitHub MCP from chat; Cowork only for local-file/multi-file jobs). Added page-standards layer (no client identifiers on public pages). Unified README table with Family column. Canonical-home header. Reconciled with onboarding publishes. |
| 1.3 | 2026-05-29 | Removed GA4 injection (GDPR — no analytics without consent banner). Publish/unpublish guide. >60KB fallback. |
| 1.2 | 2026-04-23 | File size check; push_to_github.py fallback. |
| 1.1 | — | Initial. |

## Fixed configuration — never change
- **Repo:** `rcsutkai/futureproof-web`, branch `main`
- **Domain:** `prezentacia.fproof.eu` — `/CNAME` must contain exactly this line; verify, never modify
- **Path families:** `/presentations/[SLUG]/index.html` (client decks) · `/onboarding/[SLUG]/index.html` (contractor process pages). New families only on Roman's explicit decision.
- **Slug rules:** lowercase, hyphens only, no spaces or special characters

## Execution mode
**Default: deploy directly via the GitHub MCP connection from the current chat** — a single-file push needs no separate Cowork task. Use **Cowork** only when the job involves local files Claude-in-chat cannot read, multiple files, or a long-running batch. Whichever mode runs, it follows this same procedure.

## Page standards (check before every publish)
1. `<meta name="robots" content="noindex, nofollow">` as first child of `<head>` — inject if missing.
2. **No analytics.** GA4 is never injected: these are private, link-only pages and analytics would require a GDPR cookie-consent banner. (This supersedes any older "GA4 on all properties" note — see the 2026-05-29 tropic-square incident in the deployment log: a GA4 tag was added, corrected to the futureproof account, then the whole page was pulled for non-compliance. That removal is the reason this rule exists.)
3. **No client-confidential identifiers** on public pages: no PO numbers, no client-side personal contacts/emails, no rates from client contracts. Pages are unauthenticated — noindex hides them from search, not from anyone with the link. If found, stop and ask Roman before publishing.
4. Self-contained single file (inline CSS/JS; Google Fonts links allowed). Brand: Poppins/Open Sans, #4A79FF accent per futureproof CTX.
5. Do not modify any other page content beyond these checks.

## Procedure
1. **Size check.** Under 60 KB → MCP push. Over 60 KB → MCP `push_files` truncates (model output budget, not a GitHub limit): use local git (`clone → cp → commit → push`) or `push_to_github.py`; resume at step 3.
2. **Commit** `index.html` to `/[family]/[SLUG]/index.html`. Message: `publish: [SLUG]`.
3. **robots.txt** in repo root: ensure it contains `User-agent: *` and a `Disallow: /[family]/[SLUG]/` line (create file if absent). One-time simplification allowed: a single `Disallow: /onboarding/` covers the whole family.
4. **CNAME:** verify `/CNAME` = `prezentacia.fproof.eu`; create only if missing; never edit.
5. **README:** append to the unified table `| Title | Family | Slug | Published | URL |`. Create the table if absent; migrate any old presentations-only table headers on first touch. **Before adding a row, also verify existing rows are still true** — a README row is a claim that the file exists on `main`; check with `get_file_contents` if in doubt, since unpublish events do not remove themselves from the README automatically.
6. **Verify live** — open `https://prezentacia.fproof.eu/[family]/[SLUG]/`; Pages deploys in ~1–2 min. If not live after a few minutes, report — do not re-push.
7. **Report:** committed path, live URL, commit link, and confirmation of standards checks 1–3.

## Unpublish / republish
Remove from `main` = offline in ~2 min: `git rm -r [family]/[SLUG] && git commit -m "unpublish: [SLUG]" && git push` (or MCP delete_file). Source HTML always stays in the Vault; republish = same procedure again. `ls` of the family folders shows what is live. **Every unpublish must also update the README row** (mark unpublished or remove it) in the same session — an orphaned README row is the single most common drift in this repo (see tropic-square, 2026-05-29).

## One-time DNS (already done)
Websupport DNS: CNAME `prezentacia` → `rcsutkai.github.io`, TTL 600; GitHub Pages custom domain set to `prezentacia.fproof.eu` with HTTPS. Only relevant if the domain ever moves.

## Deployment log
| Date | Family | Slug | Note |
|---|---|---|---|
| 2026-05-20 | presentations | tropic-square-fw-sourcing | Published |
| 2026-05-28 | presentations | tropic-square-fw-sourcing | GA4 tag corrected to futureproof account G-5567R3XGKM |
| 2026-05-29 | presentations | tropic-square-fw-sourcing | **Unpublished** — GA4 non-compliant (no consent banner on a client-facing page). File no longer exists on `main`. README currently still lists it as live — needs Roman's decision: mark unpublished, or republish without analytics per the page-standards rule above. |
| 2026-09-01 | onboarding | hcl | HCL/MSD contractor onboarding, Step 1 |
| 2026-09-04 | onboarding | hcl-billing | HCL/MSD Step 2 — time tracking & invoicing |
