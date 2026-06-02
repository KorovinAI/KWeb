# KWeb Agency — Web Build Stack & Architecture Standard

**Status:** Living standard · **Last updated:** 2026-06-02 · **Owner:** Ilia (Korovin AI)
**Applies to:** Every client website built in `websites/`.
**How to use this doc:** Read it before starting or modifying *any* client site. §1–12 are the *standard* (what/why). **§13 is the reference implementation + per-client setup checklist** (the how — what to actually copy to spin up a new site). Deviations are allowed but must have a documented reason. When the stack evolves, update this file (and note why).

> Cross-checked against May 2026 market reality across four research rounds (CMS, hosting, database, SEO/AEO, future-proofing). **First reference build: the Korovin AI agency site itself** — `websites/agency/`, migrated to this stack May 2026 (see §13 and `output-files/korovinai-payload-migration-*`). **Now LIVE at `agency.korovin.ai`** (June 2026, on DigitalOcean) — the RBAC, CDN-cache, and admin-email fixes discovered during go-live are folded back into §8 and §13.5b/13.8/13.9.

---

## 1. What we're building toward

Korovin AI is a **one-stop B2B growth agency**: build the site (simple → complex), rank it (SEO + AEO), drive traffic (ads + content), and convert it into pipeline (leads → revenue). Every client site must be able to grow into: **form ingestion, visitor tracking, and an AI chatbot.** All three require a server + database — so the stack is server-first by design.

---

## 2. The standard stack

| Layer | Standard | Why |
|---|---|---|
| **Framework** | Next.js 16 (App Router) + **Payload CMS 3.x** | One stack spans brochure → portal → app. Payload is an *application framework* (auth, RBAC, hooks, jobs, custom endpoints), not just a CMS. MIT-licensed, self-hostable. |
| **Rendering** | Server-rendered. Public pages **static/ISR**; admin CSR | Static-fast HTML + instant edits via on-demand revalidation. Never client-fetch public content (see §7). |
| **Database** | **MongoDB Atlas** | Best fit for Payload's nested content + zero-migration simplicity. Atlas **Vector Search** (native, Voyage auto-embedding) is an asset for the chatbot roadmap. |
| **Hosting (single site / start)** | **DigitalOcean App Platform** (Sydney region) | Predictable flat billing, AU latency, no cold starts. Node 22, ~$10/mo. |
| **Hosting (fleet, later)** | **Coolify** on Sydney VPS (DO/Vultr/Linode) | ~$1–2/site at scale; full control. Trade: you own the ops. |
| **Email** | **Resend** (transactional), from a `updates.<domain>` subdomain | Form notifications + auth mail. Free 3k/mo. Domain-auth via SPF/DKIM/MX. See §8 + §13. |
| **Forms** | Payload Form Builder → leads in DB + Resend email (CRM optional) | Own the leads first-party; no CRM required. See §8. |
| **Auth / RBAC** | Payload auth + a `role` field (admin/editor), enforced via access helpers | Owner-vs-editor split on every site (client edits content; agency keeps settings/users). See §8 + §13.5b. |
| **Spam** | Honeypot (now) + Cloudflare **Turnstile** (added per client) | Free, privacy-friendly. |
| **Tracking** | PostHog (first-party proxy) + first-party event log + MS Clarity | Own visitor data; tie visitors → leads. See §8. |
| **Chatbot** | Vercel AI SDK + Atlas Vector Search (RAG) | Reusable module, server-side. See §8. |

**Do NOT use** for client production: static export (`output:'export'`) for any site needing forms/tracking/chatbot; free **spin-down** hosting tiers (cold starts destroy SEO + conversion — §5); Vercel Hobby (prohibits commercial use); Supabase as a Payload datastore.

---

## 3. Core architectural principles

1. **Server-rendered, not static.** Forms, tracking, and chatbots are impossible or insecure on a static site.
2. **Public pages static/ISR; admin CSR.** Content, metadata, and JSON-LD must be in the initial HTML response. Reserve client-side rendering for the admin/logged-in areas only.
3. **Own the data, first-party.** Leads and visitor events live in *our* database — durable against ad-blockers, cookie deprecation, vendor lock-in.
4. **Vendor-neutral where it counts.** Payload is MIT + self-hostable; content is portable. Any third-party (CRM, calendar, analytics) is swappable per deployment.
5. **Performance is a feature.** Lighthouse ≥ 95, LCP < 2.5s, TTFB < 200ms (warm). Static/ISR + always-on host keeps it as fast as a static site.

---

## 4. The two-tier delivery model

| | **Tier 1 — Productized** | **Tier 2 — Bespoke** |
|---|---|---|
| What | Templated sites, shared codebase, differ by content/brand | Complex / app-heavy: portals, gated content, chatbots, custom integrations |
| Architecture | **One multi-tenant Payload instance** (`@payloadcms/plugin-multi-tenant`), many domains | **Dedicated Payload instance per client** (fork this template) |
| Cost | ~$5–10/mo *amortised across tenants* | ~$10–40/mo per instance |
| Onboarding | Add a tenant, no code | Fork + the §13 checklist (a few hours) |
| Margin | High | Premium-priced |

**The lever that makes this scale: a shared internal module/component library** (forms, tracking, chatbot, SEO scaffolding — built once, reused). The agency site `websites/agency/` is the seed of that library. **Rule:** never stretch multi-tenancy to a genuinely bespoke design — draw the tier line in the sales process.

---

## 5. Hosting standard + decision tree

- **Start every site → DigitalOcean App Platform**, Sydney. Always-on, flat ~$10/mo (Basic 1 GB / 1 vCPU / 1 container), Node 22. **1 GB minimum** — 512 MB risks `next build` OOM.
- **Fleet (5+ clients) → Coolify on Sydney VPS.** ~5–10 small Payload sites per box; ~$1–2/site marginal. You own backups/patching/monitoring.
- **Client-owned vs agency-managed:** the *same codebase* deploys to either — only account ownership/billing differs.
  - **Client-owned:** provision in the client's own Atlas + DO + domain; hand over keys. Zero ongoing agency liability.
  - **Agency-managed (MSP):** **one** Atlas org + **one** DO account, **many projects** (one per client). Each client gets a scoped Payload editor login. Bill as a retainer.
  - Multi-tenant (Tier 1) is agency-managed only. Keep *separate-instance* as the default to preserve the owned-or-managed flip.

**Never** use a free spin-down tier for a public site — low-traffic sites cold-start on most visits (~60s load → ~90% bounce + SEO penalty). The ~$10/mo always-on tier buys the warm server.

---

## 6. Database standard

- **MongoDB Atlas.** **Free M0 is acceptable for a low-traffic single site** *if* mitigated: content lives in git, and every lead is emailed (so no lead is lost despite M0 having no backups). M0 also pauses after 30 days of zero connections — a non-issue for a live site, trivially solved with a keep-alive ping.
- **Upgrade to Flex (~$8/mo, adds backups)** when the site holds data that *only* exists in the DB and matters (e.g. a busy lead pipeline you don't want to risk), or when it outgrows M0's 512 MB / shared throughput. It's a 1-click change, no migration.
- **Multi-client:** one Atlas org, **project-per-client** (1 free M0 each, up to ~250 projects).
- **Chatbot tier:** production Atlas Vector Search needs **M10+** (not free). Budget into chatbot pricing.
- **Dev/prod separation:** ideally a separate database per environment (e.g. `agency` prod + `agency_dev` local) on the same cluster. *(The agency reference build currently shares one DB across dev+prod — a known shortcut to split later.)*
- **Standardise ONE database engine across the fleet.** Mongo wins here (Payload content + chatbot embeddings co-locate; Atlas auto-embedding removes the pipeline). Postgres + pgvector is viable but loses that.

---

## 7. Rendering, SEO & AEO rules

**The core truth:** a crawler indexes the HTML it *receives* — static vs server-rendered are SEO-equivalent. The risk is never "server vs static" — it's "server HTML vs client-side JS."

1. **Public pages: static or ISR (server HTML). Never client-fetch.** This is the whole ballgame.
2. **AEO is mandatory** (we sell it): **AI crawlers — GPTBot, ClaudeBot, PerplexityBot — do NOT execute JavaScript.** A client-rendered SPA is invisible to them. Server-rendered Payload is exactly right.
3. **Findability checklist (every site):** `generateMetadata` per route · dynamic `sitemap.ts` · `robots.ts` · canonical URLs · JSON-LD (Organization/Article/FAQ/Breadcrumb) · semantic HTML · Open Graph.
4. **Speed:** static/ISR + always-on host protects Core Web Vitals (a confirmed ranking factor).
5. **AEO content signals:** question-led headings, direct 40–80-word answers, schema matching visible copy, author E-E-A-T, freshness. Ship `llms.txt` (cheap; not yet a proven ranking factor).
6. **On any migration: preserve URLs 1:1** (301 anything changed) + port the full metadata surface, or you drop rankings.

---

## 8. Capability modules (build once, reuse across clients)

**Forms / lead capture** — *implemented; see §13.5*
- `@payloadcms/plugin-form-builder` → `forms` + `form-submissions` collections. Leads stored **first-party in the client's DB** — viewable/exportable in `/admin`. **No CRM required** — this is the default.
- **Email notification (Resend):** an `afterChange` hook on `form-submissions` emails every lead to the site's contact address, from `notifications@updates.<domain>`, **reply-to = the submitter**. So you get the lead in your inbox *and* own a DB copy.
- **Spam:** honeypot field (zero-dep, shipped) + Cloudflare **Turnstile** verified in a `beforeChange` hook (added per client).
- **CRM (optional, per client):** an `afterChange` hook can also push to HubSpot (pass the `hubspotutk` cookie or attribution is lost), Pipedrive, Salesforce, or a generic webhook/Zapier.

**Roles & access control (RBAC)** — *implemented; see §13.5b*
- A `role` select field (`admin` | `editor`, default `editor`) on the `Users` collection, plus shared helpers in `access/roles.ts` (`isAdmin`, `isAdminOrEditor`, `isAdminOrSelf`, `isAdminFieldLevel`).
- **admin** = full control (manage users, edit Site Settings, delete content). **editor** = create/edit content + media, view leads; *no* user management, *no* Site Settings, *no* delete. The role field itself is field-locked to admins (no self-promotion).
- This is the **client-vs-agency boundary**: hand the client an *editor* login so they manage copy safely, while the agency keeps *admin* over settings/users/integrations. Applies to every site, single-tenant or multi-tenant.

**Visitor tracking / attribution**
- **PostHog** behind a first-party reverse proxy on a *neutral* subdomain (avoid analytics/tracking/posthog — ad-blockers target them).
- First-party event log → DB with a 13-month `HttpOnly` cookie. Stamp an anon visitor ID; join it to the form submission → traffic → lead → revenue loop.
- **MS Clarity** (free) for heatmaps/replay. **Consent Mode** defaulting to `denied` in the EU (server-side tracking is *not* a consent exemption).

**AI chatbot (RAG over client content)**
- **Vercel AI SDK** (`useChat` + `streamText` + tool retrieval). LLM call is **server-side**. Retrieval: **Atlas Vector Search** over the client's own content (Voyage auto-embedding). Cost: tokens trivial (~$0.003/convo); the real cost is the **Atlas M10+** tier — bill as an add-on.

---

## 9. Cost model (per site/month)

| Scenario | Infra cost | Notes |
|---|---|---|
| Single site (agency reference) | **~$10** | DO 1 GB ($10) + Atlas M0 (free) + Resend (free) |
| Tier 1 templated tenant | ~$5–10 *amortised* | One multi-tenant box serves many |
| Tier 2 bespoke instance | ~$10–40 | Feature-rich = 2–4 GB RAM |
| + production DB backups | + ~$8 Atlas Flex | When DB-only data matters |
| + Chatbot | + Atlas M10+ (~tens of $) + trivial tokens | Add-on |

Hosting is a **margin line, not a cost line** — pass it through as a retainer/add-on. Edits propagate in ~3s (time-based ISR; on-demand revalidation alone is *not* enough behind DigitalOcean's CDN — see §13.8) with **no per-deploy cost**.

---

## 10. When to reach beyond the stack

- **Heavy e-commerce** → headless **Shopify** + Stripe (Payload's e-commerce plugin is beta).
- **Real-time / OLAP / ML / high-throughput** → a dedicated sidecar service alongside Payload, not inside it.
- Payload doesn't auto-scale — scaling = standard Node practice (caching, ISR/CDN, horizontal instances, DB tuning).

---

## 11. Risks & watch-items

1. **Operational sprawl is the #1 risk.** N bespoke clients = N deploys/DBs/patches/backups. Mitigate: Coolify + templated config + automated backups + the shared module library.
2. **Team capacity, not the stack, is the bottleneck.** Productize ruthlessly.
3. **Payload under Figma** (acquired 2025) — possible OSS-vs-SaaS paywall creep over 2–3 yrs. Bounded by MIT (worst case: fork). 2026 velocity is high.
4. **Free tiers don't reach production** for chatbot (Atlas M10+) or analytics (PostHog Cloud). Budget from day one.
5. **Payload CLI quirk:** `generate:importmap`/`generate:types` can fail on bleeding-edge Node + odd paths (the agency env hit this on Node 26 + an iCloud path). The dev runtime auto-regenerates `importMap.js`, so it's not blocking — don't depend on the standalone CLI.

---

## 12. Decision record (why these choices)

- **CMS = Payload.** Evaluated Sveltia/Decap (editor UX too plain), Tina (most engineering + per-project cost), Sanity (content leaves git, visual editing needs a server, lock-in), CloudCannon/Pages CMS (git-static only — can't host the app roadmap). Payload spans simple→complex (app framework) + natively supports forms/auth/jobs/chatbot.
- **Why not static export.** Forms, tracking, chatbot all need a server + DB; static is a dead-end for the roadmap.
- **Host = DigitalOcean** over Railway (Railway: verified ~8h May-2026 outage, no AU region, thin managed DB). DO has Sydney + flat pricing.
- **DB = MongoDB Atlas**, **free M0 acceptable for low-traffic single sites** (mitigated by git + email). Flex when DB-only data matters.
- **Email = Resend** (3k/mo free, good deliverability; auth via a `updates.` subdomain so it never touches the domain's Google Workspace MX).
- **Research dates:** May 2026. Re-validate pricing/free-tiers before each new client commit.

---

## 13. Reference implementation & per-client setup

The **Korovin AI agency site (`websites/agency/`)** is the canonical build of this stack. To spin up a new client site, **fork it** and follow §13.6.

### 13.1 Versions (reference build)
Payload **3.85.x** (keep all `@payloadcms/*` packages on the same version), Next **16.2.6**, React 19, Node **22.x** on the host (`engines.node` in `package.json`). Tailwind v4.

### 13.2 Repo / app structure
```
payload.config.ts            # serverURL, collections, globals, admin.livePreview, email (resendAdapter), formBuilderPlugin (with RBAC access on forms/submissions)
collections/                 # Users (+ role field), Media, + content collections (each content one has a `slug`)
globals/                     # page singletons (Home/About/Contact/SiteSettings)
access/roles.ts              # RBAC helpers: isAdmin / isAdminOrEditor / isAdminOrSelf / isAdminFieldLevel
hooks/
  revalidate.ts              # afterChange/afterDelete → revalidatePath (on-demand ISR)
  leadNotification.ts        # afterChange on form-submissions → Resend email
lib/content.ts               # Payload Local API loader (async), returns UI-shaped data
components/                  # ContactForm (client), JsonLd, RefreshRouteOnSave, design system
app/
  (frontend)/                # ALL public pages — async server components reading lib/content
                             #   layout.tsx sets `export const revalidate = 3` (CDN freshness — §13.8)
  (payload)/                 # admin + API boilerplate (from Payload blank template) + admin/importMap.js
  sitemap.ts  robots.ts
.env (gitignored)  .env.example
```
**Multiple root layouts:** there is **no** `app/layout.tsx`; `app/(frontend)/layout.tsx` and `app/(payload)/layout.tsx` each provide their own `<html>`. This lets the admin and the site have separate shells.

### 13.3 Content layer pattern (the keystone)
`lib/content.ts` wraps the Payload **Local API** (`getPayload({ config })`) and returns the **same shapes the UI expects** — Payload arrays-of-objects (`[{deliverable}]`) are flattened back to `string[]`, groups passed through, dates coerced to ISO strings. This means **the design/components never change** when swapping the data source; pages just become `async`. Define a typed helper per collection/global (`getService`, `getAllInsights`, `getHomePage`, …) + `listSlugs` for `generateStaticParams`.

### 13.4 Content model → DB → frontend
- Collections/globals mirror the content types. `string[]` fields become `array` of a single named subfield. Nested objects use `group`. Rich-text bodies can be `textarea` (simple) or `richText` (lexical) — agency uses `textarea` for now.
- Seed content via a temporary dev-only route that uses the Local API (`payload.create` / `updateGlobal`), then delete the route. (`importMap.js` is auto-regenerated by the dev runtime; the broken `payload generate` CLI is not needed.)

### 13.5 Forms + email (implemented)
- `formBuilderPlugin({ fields, formOverrides, formSubmissionOverrides: { hooks: { afterChange: [leadNotification] } } })`.
- A "Contact" form (seeded). `ContactForm.tsx` renders its fields + a honeypot, POSTs `{ form, submissionData }` to `/api/form-submissions` (public create allowed; admin-only read).
- `email: resendAdapter({ defaultFromAddress: 'notifications@updates.<domain>', defaultFromName, apiKey })`.
- `leadNotification` (`afterChange`) emails the lead to Site Settings → Contact Email, reply-to the submitter, in a try/catch so mail failure never blocks the DB write.

### 13.5b Roles & access control (implemented)
- `access/roles.ts` exports `Access`/`FieldAccess` helpers: `anyone` (public read), `isAdmin`, `isAdminOrEditor`, `isAdminOrSelf` (admins → any user; others → own record only), `isAdminFieldLevel`. A user with no/unknown role is treated as least-privilege.
- `Users` gets a required `role` select (`admin` | `editor`, default `editor`). The `role` field is **field-locked to admins** (`access.create/update = isAdminFieldLevel`) so an editor can't promote themselves.
- **Access matrix applied across the app:**
  | Resource | read | create | update | delete |
  |---|---|---|---|---|
  | Content collections (Services/Insights/Authors), Media | `anyone` | admin/editor | admin/editor | **admin only** |
  | Content globals (Home/About/Contact) | `anyone` | — | admin/editor | — |
  | **Site Settings** global | `anyone` | — | **admin only** | — |
  | Users | self/admin | **admin only** | self/admin | **admin only** |
  | `form-submissions` (leads) | **admin/editor** | **`anyone`** (public submit) | admin only | admin only |
- The one rule you must not break: **`form-submissions.create` stays public** (`anyone`) or the contact form 403s for visitors. Verify with an unauthenticated POST after any access change.
- **Bootstrapping the two accounts** (no password hashing by hand needed): create the first user at `/admin`, then set roles. To add an admin programmatically, Payload hashes with `pbkdf2(password, salt=randomBytes(32).hex, 25000, 512, 'sha256')` → store `hash`+`salt`; or just create the user in `/admin` and set its role. Direct Mongo writes are fine for the plain `role` field.

### 13.6 Per-client setup checklist (fork → live)
1. **Fork** the agency repo (or copy `websites/agency/`), rename, point at the new GitHub repo.
2. **MongoDB Atlas:** new project → free M0 cluster (Sydney/Singapore) → DB user → Network Access `0.0.0.0/0` → connection string → `DATABASE_URI` (append `/dbname`).
3. **`.env`:** `DATABASE_URI`, `PAYLOAD_SECRET` (`openssl rand -hex 32`), `RESEND_API_KEY`, `NEXT_PUBLIC_SERVER_URL=https://<domain>`. (`payload.config.ts` reads `NEXT_PUBLIC_SERVER_URL` into `serverURL` — required for admin emails, §13.9.)
4. **Content + accounts:** edit collections/globals as needed; seed the client's content; create the first user at `/admin` and set its `role=admin`; add an `editor` login for the client (§13.5b).
5. **Resend:** add domain **`updates.<clientdomain>`** → paste DKIM/SPF/MX into the client's DNS (subdomain only — never touch their Google Workspace MX) → API key. Set the from-address to `notifications@updates.<clientdomain>`.
6. **DigitalOcean App Platform:** create app → GitHub repo + branch → **Web Service** (not static) → **Build `npm run build`, Run `npm start`**, port 8080 → **Basic 1 GB ($10/mo), 1 container**, Sydney → add the 4 env vars (encrypt the 3 secrets) → **skip "Add a database"** (Atlas is external via env var) → create.
7. **Spam:** Cloudflare Turnstile widget → keys → wire into `ContactForm` + a `beforeChange` verify hook.
8. **DNS cutover:** point the domain → the DO app; DO auto-provisions SSL. Keep the old host as instant rollback until verified.
9. **Verify:** all routes 200, `/admin` login, a live form submission lands in the DB **and** emails the client, sitemap/robots/JSON-LD present, Lighthouse ≥ 95. **Then confirm `curl -sI https://<domain>/` shows `Cache-Control: s-maxage=3`** (not `31536000`) so CMS edits propagate (§13.8), and that a password-reset email link is absolute (§13.9).

### 13.7 Deployment gotchas (DigitalOcean)
- Resource type must be **Web Service** (DO may suggest Static Site — change it).
- **Build command = `npm run build`** (DO sometimes pre-fills the run command in the build field — fix it).
- **1 GB instance, 1 container** (2 containers doubles cost for no benefit; 512 MB risks build OOM).
- `npm start` (`next start`) respects DO's `PORT` env automatically.
- Atlas must allow `0.0.0.0/0` (DO egress IPs are dynamic) — pair with a strong DB password.

### 13.8 ⚠️ Critical gotcha — CMS edits behind DigitalOcean's CDN
**Symptom:** you publish an edit in `/admin`, the DB updates, but the live page never changes (or changes minutes later, unpredictably).
**Cause:** DigitalOcean App Platform fronts the app with a **Cloudflare CDN**. A fully-static Next.js page emits `Cache-Control: s-maxage=31536000` (cache for a *year*). On-demand revalidation (`revalidatePath` in the Payload hooks) refreshes **Next's own origin cache only — it cannot purge DigitalOcean's external CDN**, so the edge keeps serving the year-old copy. (This "just works" on Vercel because Vercel's CDN is integrated with revalidation; DO's is not.)
**Fix:** add a short **time-based ISR window** so the CDN re-validates frequently — `export const revalidate = 3` on `app/(frontend)/layout.tsx` (cascades to all public pages). The emitted header becomes `s-maxage=3, stale-while-revalidate=…`; combined with the on-demand hooks (which keep the origin instantly fresh), edits reach visitors in **~3s**.
- **Verify the fix by header, not by eye:** `curl -sI https://<domain>/ | grep -i cache-control` must show `s-maxage=3`, *not* `s-maxage=31536000`. This is reproducible locally (`npm run build && npm start`, then curl) — never debug it by push-and-check.
- **Tuning:** 3s feels instant to an editor and is gentle on the free-tier DB. Lower (1s) gives no perceptible gain and ~3× the DB/origin load under crawler traffic. Going fully dynamic (`revalidate = 0`) makes edits instant but drops edge caching entirely.
- Keep the on-demand `revalidate.ts` hooks **and** the time-based window — they're complementary (origin-fresh + edge-refresh).

### 13.9 Admin emails (password reset) need `serverURL`
**Symptom:** the Payload password-reset / verification email link is host-less (e.g. `admin/reset/<token>`), so the browser treats `admin` as a hostname → `ERR_NAME_NOT_RESOLVED`.
**Cause:** Payload builds absolute email links from `config.serverURL`; if unset, the link has no scheme+host.
**Fix:** set `serverURL: process.env.NEXT_PUBLIC_SERVER_URL || 'http://localhost:3000'` at the top of `buildConfig`. Set `NEXT_PUBLIC_SERVER_URL=https://<domain>` in the host env.
- **Immediate unblock without redeploy:** the reset *token* is stored on the user doc (`resetPasswordToken`); build the URL by hand as `https://<domain>/admin/reset/<token>`.

---

*Related: session summary `output-files/korovinai-payload-migration-session-summary-2026-05-31.md` · migration plan `output-files/korovinai-payload-migration-plan-2026-05-31.md` · original brief/plan `output-files/korovinai-website-*-2026-05-24.md`.*
