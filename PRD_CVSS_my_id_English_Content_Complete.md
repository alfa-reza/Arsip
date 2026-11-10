# PRODUCT REQUIREMENTS DOCUMENT
# CVSS.my.id — English-only Multi-page Website & Calculator Redesign

**Date:** 10 November 2026  
**Target repository:** https://github.com/alfa-reza/cvss.my.id  
**Reference-only MCP repository:** https://github.com/alfa-reza/cvss-calculator-mcp  
**Hosting target:** Cloudflare Pages, static HTML output  
**Remote protocol endpoint (protected):** https://cvss.my.id/mcp  
**Language scope:** English only; Indonesian localization intentionally excluded  
**Implementation companion:** `Arena_Prompt_CVSS_my_id_EN.md`  
**Source:** supplied ZIP `cvss.my.id-main.zip` (authoritative embedded snapshot for the appendices). Compare with checked-out repo HEAD before implementing, but retain this complete content reference.

## 0. Executive mandate and precedence

This is an **IMPLEMENTATION REQUIREMENTS DOCUMENT** for an Arena.ai anonymous code duel. Build a full working multi-page security developer tool website, **not** a one-page mockup. The target is only the website repository and its included browser calculator / TypeScript CVSS core. The separate Go MCP project is immutable/reference-only. Web pages introduce and document MCP but are not the MCP service itself. A critical requirement is to retain all existing public routes, mathematical correctness, complete English documentation, and Cloudflare Pages static hosting while greatly improving information design, type, accessibility, and calculator task flows.

Requirement precedence: this PRD > old website `AGENTS.md`/`DESIGN.md` for layout/framework, but old security/privacy/scoring/attribution constraints remain binding. Use Next.js static export, not Astro/Starlight. Do not deploy or mutate production without authorization. Do not claim verified behavior without observed tests. Never fabricate backend functionality or a complete feature.

### 0.1 Non-negotiable prohibitions

- **NO HERO SECTION anywhere** — including a disguised top-of-page promotional billboard. Pages start with a useful compact contextual header, breadcrumb or task navigation, then actual material.
- **NO NEW IMAGES** — no photos, hero art, logo generation, stock visual, screenshot gallery, illustration, artificial graphics or background art. The only brand graphic is a replaceable text/logo placeholder; functional UI icons and score charts/gauges are allowed.
- **NO decorative WebGL/3D, particle effects, gradient glowing blobs, giant empty whitespace, fake terminal demos, carousel decorations, or generic AI marketing panels.**
- **NO GENERIC AI-SLOP**: avoid center-aligned gradient slogan, three equally weighted promo cards, emoji badges, gratuitous rounded tiles, glassmorphism, overly futuristic neon, vague `Unlock / Transform / Supercharge` copy, default shadcn demo aesthetics.
- **NO ONE-PAGE SITE**. Independent `/`, `/tools/mcp`, calculator and 31 deep docs HTML routes are mandatory.
- **NO fabricated data**. Keep real CVSS scoring and exact MCP product claims; separate the browser's broader metric groups from the Go MCP's Base-only calculations.
- **NO Indonesian translation or bilingual UI** for this scope. Do not copy docs from `apps/website/src/content/docs/id/`. Optional legacy `/id` redirects are permitted only when they do not interfere with static routing.
- **NO runtime application server, auth, database, telemetry, unapproved third-party scripts, dynamic scoring API or MCP endpoint replacement.**

### 0.2 Resolution when the environment is constrained

Deliver real code. If the agent has restricted GitHub/fetch access, work from the fully copied English docs and page-reference source appendices in THIS file. If a single doc truly cannot be reconstructed, preserve its independent URL, create a plainly labeled `placeholder.md` with brief placeholder text, and log it accurately — but the 31 English files in Appendix A **are already supplied**, so they do not qualify for such fallback merely because GitHub fetch fails. Never substitute placeholders for the homepage, MCP overview, calculators, or the scoring engine.

## 1. Product vision and user tasks

CVSS.my.id is an open-source developer/security utility with three separate pillars:

1. **Browser CVSS calculator**: accurate local CVSS 2.0, 3.0, 3.1 and 4.0 assessments including the version-specific metric groups supported by `@cvss/core` in the website repository.
2. **MCP introduction / setup**: user-friendly explanation, copyable setup commands and documentation for an independent Go MCP server (`calculate_cvss`, Base Score only), with local stdio and remote Streamable HTTP transport.
3. **English technical documentation**: CVSS reference material, vector rules, tool operation, connection steps for 13 named MCP clients, internal search and directly retrievable Markdown source.

Primary jobs: (a) paste an existing CVSS vector and receive a trustworthy calculation, (b) build a vector with explanation of each metric, (c) copy/share/export the correct result, (d) determine whether browser calculator or MCP fits the job, (e) get an AI agent running quickly with a safe local or remote MCP connection, (f) read original documentation in a comfortable typography system, (g) retrieve exact content using non-browser AI fetch systems.

## 2. Baseline facts to preserve

- Current monorepo layout: `apps/website` (Astro/Starlight), `apps/calculator` (Vite + TypeScript), `packages/cvss-core` (real calculation/parser/serializers), `tests/*`, `.github/workflows/ci.yml`.
- `apps/calculator/src/app.ts` persists **draft per version in sessionStorage** (do not silently move to cloud); URL vector in `?vector=` or supported hash overrides stored draft after validation.
- Invalid free-text vector draft remains visible, with out-of-sync messaging and safe copy/result behavior; previous valid score must not be misrepresented as current. Incomplete mandatory Base metrics are `Incomplete`, **not 0.0**.
- CVSS v4.0 distinguishes Base, Threat, Environmental and Supplemental groups; 3.x has Temporal, and CVSS v2 has historical metric behavior. Use original `@cvss/core`, not simplified score lookups or hardcoded demo values.
- Current MCP remote URL `https://cvss.my.id/mcp`, independent Cloudflare Worker with Streamable HTTP; opening it in a normal browser is not an HTML documentation request.
- Current published MCP remote policy is `5 requests per minute per IP`; treat as documentation claim inherited from source, not as newly live-verified policy.
- MCP local install command `curl -fsSL https://get.cvss.my.id/install.sh | bash`. MCP optional skill install command `npx skills add alfa-reza/cvss-calculator-mcp --skill vulnerability-cvss-assessment`; agent setup reference `https://get.cvss.my.id/LLM.txt`.
- Link to both repositories correctly; preserve Apache-2.0 website attribution and the original THIRD_PARTY_NOTICES / FIRST CVSS attribution.
- Existing Cloudflare calculator redirect-loop regression must be tested: a directory HTML route plus an opposing `_redirects` rule can bounce between `/calculator/v4.0` and `/calculator/v4.0/` forever.

## 3. Hard route inventory (every route is a separate page)

### 3.1 Product routes

| Public URL | Required page purpose | Proposed implementation file |
|---|---|---|
| `/` | Compact product index, links to the real tools and official docs; no hero | `app/page.tsx` |
| `/tools/mcp` | Independent MCP product and setup workspace; commands, tool contract, guides | `app/tools/mcp/page.tsx` |
| `/calculator/` | Calculator version chooser or functional redirect with non-JS fallback | `app/calculator/page.tsx` |
| `/calculator/v4.0` | Fully functional CVSS 4.0 assessment | `app/calculator/[version]/page.tsx` with `generateStaticParams()` |
| `/calculator/v3.1` | Fully functional CVSS 3.1 assessment | same dynamic static route |
| `/calculator/v3.0` | Fully functional CVSS 3.0 assessment | same dynamic static route |
| `/calculator/v2.0` | Fully functional CVSS 2.0 assessment | same dynamic static route |
| `/mcp` | **NOT a website page**; existing Worker protocol endpoint; protected from Pages catch-all | Cloudflare route binding, outside Next.js HTML output |

Route variations with/without trailing slash must work or 301 once without looping. Use real static per-route HTML; do not serve a generic SPA shell for every page. No need for `/id` rendered pages. Existing `/id/...` incoming links may have redirects (test separately; never rewrite `/mcp`).

### 3.2 Documentation routes and content mapping

The original English docs are 31 Markdown files and must remain individually addressable. Proposed new repo structure:

```text
app/docs/page.tsx                       # minimal docs index, not a hero
app/docs/[...slug]/page.tsx              # static generateStaticParams from content docs
content/docs/**/*.md                     # exact 31 English Markdown files from Appendix A
public/docs/**/*.md                      # auto-generated verbatim raw Markdown for fetching
public/llms.txt                          # short, stable index of docs and product URLs
public/llms-full.txt                     # complete aggregated EN docs text, no truncation
public/robots.txt                        # standard crawler discovery
app/sitemap.ts                           # only if emitted as static XML; otherwise public/sitemap.xml
```

For each `/docs/{slug}/` HTML page there MUST be direct Markdown at `/docs/{slug}.md` containing **complete document content**, preserving source frontmatter and code examples. The docs root follows `/docs/` + `/docs/index.md` (avoid collisions with `/docs.md`). The docs router must prerender HTML from the same source documents, not fetch itself on the client. Keep `content/docs/...` the source-of-truth and generate `public/docs/...` files in the build step. **Important filename transformation for index documents:** `content/docs/index.md` -> `public/docs/index.md`, `content/docs/mcp/index.md` -> `public/docs/mcp.md`, `content/docs/mcp/clients/index.md` -> `public/docs/mcp/clients.md`; ordinary files retain their relative paths. Generate from the table rather than blindly copying folders. A raw doc should not be a JS-rendered HTML file; it is a fetchable `.md` object. Do not create a wildcard `_redirects` fallback that intercepts these files.

| Source file (relative to `apps/website/src/content/docs/docs/`) | New authoring file in Next project | Public HTML route | Public raw Markdown |
|---|---|---|---|
| `calculator.md` | `content/docs/calculator.md` | `/docs/calculator/` | `/docs/calculator.md` |
| `cvss/examples.md` | `content/docs/cvss/examples.md` | `/docs/cvss/examples/` | `/docs/cvss/examples.md` |
| `cvss/metric-glossary.md` | `content/docs/cvss/metric-glossary.md` | `/docs/cvss/metric-glossary/` | `/docs/cvss/metric-glossary.md` |
| `cvss/metric-keys.md` | `content/docs/cvss/metric-keys.md` | `/docs/cvss/metric-keys/` | `/docs/cvss/metric-keys.md` |
| `cvss/overview.md` | `content/docs/cvss/overview.md` | `/docs/cvss/overview/` | `/docs/cvss/overview.md` |
| `cvss/v2.md` | `content/docs/cvss/v2.md` | `/docs/cvss/v2/` | `/docs/cvss/v2.md` |
| `cvss/v3.0.md` | `content/docs/cvss/v3.0.md` | `/docs/cvss/v3.0/` | `/docs/cvss/v3.0.md` |
| `cvss/v3.1.md` | `content/docs/cvss/v3.1.md` | `/docs/cvss/v3.1/` | `/docs/cvss/v3.1.md` |
| `cvss/v4.md` | `content/docs/cvss/v4.md` | `/docs/cvss/v4/` | `/docs/cvss/v4.md` |
| `cvss/vectors.md` | `content/docs/cvss/vectors.md` | `/docs/cvss/vectors/` | `/docs/cvss/vectors.md` |
| `getting-started.md` | `content/docs/getting-started.md` | `/docs/getting-started/` | `/docs/getting-started.md` |
| `index.md` | `content/docs/index.md` | `/docs/` | `/docs/index.md` |
| `mcp/clients/claude-code.md` | `content/docs/mcp/clients/claude-code.md` | `/docs/mcp/clients/claude-code/` | `/docs/mcp/clients/claude-code.md` |
| `mcp/clients/claude-desktop.md` | `content/docs/mcp/clients/claude-desktop.md` | `/docs/mcp/clients/claude-desktop/` | `/docs/mcp/clients/claude-desktop.md` |
| `mcp/clients/cline.md` | `content/docs/mcp/clients/cline.md` | `/docs/mcp/clients/cline/` | `/docs/mcp/clients/cline.md` |
| `mcp/clients/codex.md` | `content/docs/mcp/clients/codex.md` | `/docs/mcp/clients/codex/` | `/docs/mcp/clients/codex.md` |
| `mcp/clients/cursor.md` | `content/docs/mcp/clients/cursor.md` | `/docs/mcp/clients/cursor/` | `/docs/mcp/clients/cursor.md` |
| `mcp/clients/gemini-cli.md` | `content/docs/mcp/clients/gemini-cli.md` | `/docs/mcp/clients/gemini-cli/` | `/docs/mcp/clients/gemini-cli.md` |
| `mcp/clients/github-copilot.md` | `content/docs/mcp/clients/github-copilot.md` | `/docs/mcp/clients/github-copilot/` | `/docs/mcp/clients/github-copilot.md` |
| `mcp/clients/hermes-agent.md` | `content/docs/mcp/clients/hermes-agent.md` | `/docs/mcp/clients/hermes-agent/` | `/docs/mcp/clients/hermes-agent.md` |
| `mcp/clients/index.md` | `content/docs/mcp/clients/index.md` | `/docs/mcp/clients/` | `/docs/mcp/clients.md` |
| `mcp/clients/jetbrains.md` | `content/docs/mcp/clients/jetbrains.md` | `/docs/mcp/clients/jetbrains/` | `/docs/mcp/clients/jetbrains.md` |
| `mcp/clients/kilo-code.md` | `content/docs/mcp/clients/kilo-code.md` | `/docs/mcp/clients/kilo-code/` | `/docs/mcp/clients/kilo-code.md` |
| `mcp/clients/opencode.md` | `content/docs/mcp/clients/opencode.md` | `/docs/mcp/clients/opencode/` | `/docs/mcp/clients/opencode.md` |
| `mcp/clients/vscode.md` | `content/docs/mcp/clients/vscode.md` | `/docs/mcp/clients/vscode/` | `/docs/mcp/clients/vscode.md` |
| `mcp/clients/zed.md` | `content/docs/mcp/clients/zed.md` | `/docs/mcp/clients/zed/` | `/docs/mcp/clients/zed.md` |
| `mcp/index.md` | `content/docs/mcp/index.md` | `/docs/mcp/` | `/docs/mcp.md` |
| `mcp/local.md` | `content/docs/mcp/local.md` | `/docs/mcp/local/` | `/docs/mcp/local.md` |
| `mcp/remote.md` | `content/docs/mcp/remote.md` | `/docs/mcp/remote/` | `/docs/mcp/remote.md` |
| `mcp/security.md` | `content/docs/mcp/security.md` | `/docs/mcp/security/` | `/docs/mcp/security.md` |
| `mcp/tools.md` | `content/docs/mcp/tools.md` | `/docs/mcp/tools/` | `/docs/mcp/tools.md` |

**Note on slugs:** under `/docs/mcp/clients/`, the `index.md` maps to `/docs/mcp/clients/`, not `/docs/mcp/clients/index/`. Sibling source files are separate pages. The `.md` files should preserve source text as given in Appendix A, including frontmatter. Rendered HTML strips frontmatter into metadata, not visible body.

## 4. Product-page content and information architecture

### 4.1 Homepage `/`

Do not add a conventional hero. Begin with a **small product title line** and purpose statement; immediately present functional links to Calculator, MCP setup and Docs in an information-oriented navigational layout. The original English copy and section texts are included exactly in Appendix B (English translation strings) and Appendix C (original `HomeView.astro`). Preserve the real claims and all important actions, but rewrite the shell structure for task completion:

- Primary navigation: `Calculator`, `MCP`, `Docs`, `GitHub`; clear active state, skip link, accessible mobile navigation.
- Calculator: direct `v4.0`, `v3.1`, `v3.0`, `v2.0` links and evidence of version group support; do not label v4 Supplemental as adding score in all cases.
- MCP: local installation and remote endpoint with copyable commands; published rate policy; explicit distinction Base-only MCP vs full browser calculator.
- Docs: deep links to CVSS reference, vector syntax, client setup, and calculator instructions.
- Open source: both correct GitHub repository links, FIRST attribution, privacy/no analytics claims only where factual.
- No invented product statistics, fake testimonials, pricing tables, email capture, signup box or illustration.

### 4.2 MCP workspace `/tools/mcp`

Retain every meaningful feature from Appendix B/C: setup via agent prompt, local installer, remote endpoint, optional skill, example `calculate_cvss` JSON input/output, client setup guide, tool reference, security/privacy docs, Registry/GitHub links. Organize according to tasks: 1) choose Local or Remote; 2) copy command/config; 3) verify setup; 4) inspect tool contract; 5) troubleshoot/read doc. Include a clear pointer for novice visitors: `/mcp` is an endpoint for MCP clients, not a browser page.

**Status button:** optional on-demand reachability check from client only; do not auto-poll, make excessive requests, or pretend the status came from a successful handshake if not tested. Expose rate-limit/connection/cors errors as appropriate. Copy buttons announce success/failure and present selectable text if clipboard blocked. Keep commands exact unless the original MCP docs are later verified to differ. Use source's exact tool contract when available; no new MCP APIs.

### 4.3 Docs landing `/docs/`

A quiet technical index: sidebar navigation, search, headings and direct links. User explicitly allows this page to be minimal, even a clearly marked Markdown placeholder if reconstruction is blocked. **But the 30 other English deep docs plus this index are all embedded in Appendix A; implement them rather than pretending they cannot be copied.** Never show filler Lorem Ipsum as if real technical guidance. A placeholder must say it's a placeholder, not contain misleading advice.

## 5. Calculator — three interchangeable interface modes

The screenshots supplied by the owner show two useful patterns: (1) `cvss.guru`: approachable arc gauge, Show explanations, groups and vector clipboard; (2) `cvsscalculator.com`: expert-dark assessment display with a detailed segmented gauge, severity spectrum, score group breakdown, vector string and export. **These are experience references, not assets or permission to reproduce code, brand, screenshot art, or pixel-identical components.** Design your own interface without introducing pictures or a hero.

### 5.1 Shared calculator domain architecture

```text
URL / calculator version selector / stored draft
  -> per-version draft model + vector parser + validation
  -> @cvss/core calculateScore (single source of truth)
  -> view model { completeness, validity, score(s), severity,
                  nomenclature, metric groups, errors, copy/export gates }
  -> selectable Assessment / Guided / Compact React views
```

**No mode may own separate scoring logic.** Mode changes must preserve metric choices, vector text, last-valid state rules, selected version, and keyboard focus where feasible. Calculator should not require a remote MCP call; it works offline after static assets load. Prefer local state store with selectors (Zustand only when state complexity warrants it); avoid forcing whole-page re-renders on every choice.

### 5.2 Mode A: Assessment (default)

- Task-oriented top context bar with version switch, layout selector, theme, copy/share; not a marketing banner.
- Distinctive segmented (tick-mark) semicircular **score gauge** with large 0–10 value, clear severity label and exact accessible text. Gauge may animate smoothly only on valid score change; `prefers-reduced-motion` disables animation.
- Horizontal 0–10 severity spectrum with correct version boundaries; textual label and color redundancies.
- Below score: valid score group summary (Base, Threat, Environmental, Supplemental for v4; applicable historical labels for v3/v2), never pretend missing groups exist or conflate Supplemental with a direct score.
- Vector string editor with validation, length protection, Copy vector, Share link, and Export assessment. Export may be JSON/text and optionally local download, accurately reflecting actual data; no server.
- Metric groups below summary with keyboard-selectable values, inline definitions and collapsible advanced groups. Gauge should remain visible on larger desktop via sticky section if it does not hide content; on mobile it appears in normal flow.

### 5.3 Mode B: Guided (learning + careful review)

- Prominent **Show explanations** toggle and progressive disclosure of metric definitions.
- Grouped steps by required and optional metrics with `x/y complete` progress and definitions in plain English.
- Explanations should describe tradeoffs and metric effects faithfully; do not claim CVSS is simply the sum of individual metric points.
- Keep persistent/accessible current-score region, but `Incomplete` until mandatory metrics valid.
- Allows switching to expert modes without losing selection or focus unexpectedly.

### 5.4 Mode C: Compact (keyboard-first analyst)

- Dense but usable metrics form: concise labels and abbreviations, responsive multi-column selector, less vertical chrome, copy/paste vector inline.
- Extensive keyboard support, clear focus rings, one-click reset with confirmation for destructive actions, no tiny touch targets.
- Useful for a security engineer scoring many records; may offer keyboard shortcuts with discoverable cheat sheet, while not hijacking browser/assistive shortcuts.

### 5.5 All-mode interaction contracts

| State/event | Required behavior |
|---|---|
| First visit with no complete vector | Show `Incomplete`, not `0.0` claiming calculated score |
| User selects a metric | Update valid vector and all synchronized views based on the same `@cvss/core` result |
| User edits vector to invalid text | Preserve the draft, show precise parse problem, do not display old score as current or allow stale export |
| User enters valid `?vector=` | Validate and load it; preserve version match and prevent cross-version corruption |
| User changes CVSS version | Navigate to correct real version route; store separate draft per version |
| User changes UI mode | Preserve vector, score, definitions setting and version; persist chosen mode |
| User uses Copy | Produce exact relevant string; announce copy status, fall back to selectable content |
| User selects Export | Produce genuine local export of current validated assessment, no fabricated fields |
| Mobile 320px or 200% zoom | No horizontal page overflow; scrollable long code/vector only inside local container |
| Reduced motion | Gauge/section animations disabled or shortened without losing information |
| Full/partial keyboard navigation | Label associations, roving or native focus semantics, no traps |

## 6. Visual identity, typography, and anti-AI-slop guardrails

**Goal:** an editorial-quality security reference / precision engineering instrument, NOT a premium SaaS billboard. Light and dark modes are both required; choose a restrained neutral palette plus a purposeful non-purple accent, with severity colors reserved for CVSS meaning. Avoid neon glow. UI may be visually polished, but the layout must be function-first.

**Typeface:** prefer licensed self-hosted `IBM Plex Sans` for UI and documentation (continuity with old website font asset), `IBM Plex Mono` for vectors, commands, scores and code. Self-host the weights/subsets actually used and include license. No third-party font request. If IBM Plex Mono asset is unavailable, use a credible local/system monospace until legitimately added. Do not choose typography merely because it is fashionable. The baseline only includes one Plex Sans weight; source the additional required weights responsibly.

**Reading defaults:** base body 16–18px depending on context; docs body 17–18px at comfortable width (~65–75 characters), line-height ~1.65–1.8, clear `h2/h3` rhythm, accessible inline links and balanced whitespace. Heading hierarchy must not masquerade as a hero; no huge empty heading block. Code uses clear monospace, horizontal scrolling inside blocks, sticky copy only where helpful. Support browser zoom, high contrast, visible keyboard focus, forced-colors, and width at 320px. Optional `Comfortable / Compact` reading density preference should not change document content.

**Motion:** functional only: mode switch transition, active tab/accordion state, score gauge interpolation, copy feedback, navigation state. Use Motion for React selectively and with reduced-motion handling. No scroll-triggered fade-in that hides content from crawlers or the user, no always-running CSS animation, no GSAP scene for decoration.

**Logo:** use a tiny text-only `CVSS.my.id` placeholder, e.g. `components/brand/LogoPlaceholder.tsx` and one CSS token. Mark it clearly for owner replacement. Never create or fetch art assets.

## 7. Documentation UX and full Markdown fetch requirements

**Source model:** 31 individual English `.md` files under `content/docs`. Each has a static rendered HTML route and a directly retrievable identical raw Markdown URL. No MDX custom component may remove content from raw output. Preserve frontmatter, fenced code blocks, links, tables, callouts, exact examples, technical caveats, and all client-specific setup snippets.

**Desktop doc layout:** left navigable section tree (CVSS, Calculator, MCP, Client guides), stable center article column, optional right outline / On this page. Mobile navigation uses an accessible drawer, not squeezed columns. Search results should show titles, section snippets, and deep links; search index can be compiled at build time with all English source documents, no network service. `Ctrl/Cmd+K` command palette is optional but should not break typing in code/vector fields.

**Full retrieval across common AI scrapers**: pages must be legible in *raw HTML*, not exclusively rendered after client JavaScript. Exa's Contents API extracts clean full page Markdown or highlights; Tavily Extract provides `raw_content` (advanced extraction supports complex pages); Firecrawl Scrape supports Markdown output with main-content filtering and JS rendering. Building fully server-produced HTML at export time, and direct original `.md` endpoints, makes content robust across both basic HTTP fetch and rendering-based systems. This does **not guarantee indexing/GEO scores or that every third-party bot retrieves every URL** — test representative requests and avoid blocking crawlers unintentionally.

Create `llms.txt` as a concise discoverable curated link list, and `llms-full.txt` as a full aggregated English document (each doc labeled with canonical URL and a stable separator). Do not confuse `llms.txt` with an indexing standard that guarantees AI inclusion. Include `robots.txt`, sitemap, canonical and per-doc title/description, `lang="en"`, OpenGraph text metadata (not external imagery), BreadcrumbList or SoftwareApplication JSON-LD only if accurate, and consistent internal links. Never gate docs behind JavaScript, a login, `noindex`, a hard-to-dismiss overlay, or a route-wide client-only loading spinner. `.md` response MIME should be checked (`text/markdown; charset=utf-8` if host supports it; fallback acceptable if content is readable and status 200).

### 7.1 Concrete HTTP acceptance examples

```bash
# Build with the site's chosen root script; Next export output must contain HTML per route.
# In production or a Cloudflare-like preview, run:
curl -fsSL https://cvss.my.id/docs/cvss/v4/ | grep -F 'CVSS v4.0'
curl -fsSL https://cvss.my.id/docs/cvss/v4.md | grep -F 'CVSS'
curl -fsSL https://cvss.my.id/docs/mcp/clients/codex.md | grep -F 'codex mcp add'
curl -fsSL https://cvss.my.id/llms-full.txt | grep -F 'CVSS'
curl -fsSL https://cvss.my.id/sitemap.xml | grep -F '/docs/'
# The existing /mcp endpoint is tested with the genuine HTTP MCP handshake, NOT by expecting HTML.
```

These are post-deployment acceptance examples. During Arena execution, run against the locally served static output or preview instead; DO NOT alter production to make the examples pass. Tests must inspect actual response content, not just HTTP 200.

## 8. Site architecture and deployment constraints

**Stack**: Next.js App Router (static `output: 'export'`) + React + strict TypeScript + Tailwind CSS 4 + accessible Radix/shadcn primitives + Motion. Use MD/MDX build-time rendering, syntax highlighting (Shiki if practical), and local search index. No Astro, Starlight, runtime SSR, on-request dynamic Next API route, cookies, Server Actions or external database. Import `@cvss/core` from the website monorepo without changing scoring math. Next `generateStaticParams()` explicitly enumerates doc paths and calculator version paths, `dynamicParams = false` where relevant.

**Cloudflare Pages settings**: root install/build command produces Next `out/` (or a documented copy within workspace); Pages preset `Next.js (Static HTML Export)` and output directory `out` as in Cloudflare official guide. No application runtime on Pages. Document how the existing standalone Worker remains bound to `/mcp` (Cloudflare routing precedence must be verified in target account), and ensure Next build output does not emit an HTML route at `/mcp` or redirect it. Keep exact static `_headers` / `_redirects` files scoped; no catch-all SPA rewrite. Route handling, query vectors, Markdown raw paths, and slash normalization must be tested on a Cloudflare-compatible preview.

**Build contracts**: `npm install`, `npm run typecheck`, `npm test`, `npm run build`, `npm run lint` should be supported or their replacement scripts documented. Preserve FIRST corpus and CVSS version regression tests from source, adapt only frontend-static-specific expectations. Automatic CI should verify static routes, docs HTML/raw parity, link integrity, and MIME where preview permits. Avoid unneeded heavy client JS for docs; ship interactive calculator code only on calculator routes.

**Performance**: score UX feedback fast, route transitions reliable, no CLS from font switches. Lazy load optional helper explanations/visual animation code; Core Web Vitals measured instead of fictional numeric claims. Never deliberately ship heavy runtime bundles simply for benchmark showmanship. Minimize initial JS on docs, though UX implementation complexity may be substantial.

## 9. Implementable deliverable sequence

1. Read source and PRD; create a route/content inventory and identify the `@cvss/core` API contract and output formats.
2. Scaffold Next static output; create shared responsive accessible navigation, theme, typography tokens, logo placeholder, and route skeletons.
3. Implement `/`, `/tools/mcp`, `/calculator/` and 4 distinct calculator version routes with three modes wired to one real scoring engine.
4. Migrate **31 English documents** from Appendix A verbatim to `content/docs`; compile each HTML page plus raw public `.md` sibling and generate the sitemap, robots and llms files.
5. Implement docs reading UX, local indexed search, code-copy, ToC, prev/next navigation, crawlable HTML and rich metadata.
6. Verify Cloudflare static build and `/mcp` isolation, valid/invalid vector deep links, per-version drafts, score corpus, redirects, keyboard focus, mobile UX, and all links.
7. Polish meaningful interactions after P0 passes; summarize actual build/test output and incomplete blockers honestly.

## 10. Acceptance matrix (hard gates)

| ID | Priority | Objective and test |
|---|---|---|
| A01 | P0 | Website repo only changed; MCP Go repo untouched |
| A02 | P0 | `/mcp` Worker path not emitted/overridden by Next HTML or redirect |
| A03 | P0 | `/` and `/tools/mcp` are separate, functional, and copy details accurately |
| A04 | P0 | `/calculator/` plus four version URLs work in direct navigation/refresh |
| A05 | P0 | v2/3.0/3.1/4.0 all use existing correct scoring parser/serializer and FIRST vector tests |
| A06 | P0 | Assessment, Guided, Compact switch without data loss or scoring differences |
| A07 | P0 | Incomplete/invalid/stale states never display false valid scores, copies or export |
| A08 | P0 | 31 English docs each have separate prerendered HTML routes |
| A09 | P0 | All 31 English docs exist verbatim at mapped `content/docs` source destinations |
| A10 | P0 | All 31 docs have 200-OK raw `.md` versions with FULL source text |
| A11 | P0 | Zero Indonesian content or translation requirement in rendered UI/docs |
| A12 | P0 | No hero, imagery, decorative 3D, gradient blob, generic promo card grid or invented logo |
| A13 | P0 | Static export builds for Cloudflare Pages without runtime Next backend |
| A14 | P0 | No Cloudflare redirect loops, no route conflicts with `.md` or Worker |
| A15 | P0 | Docs and core page main text present in initial HTML with JS disabled |
| A16 | P0 | Commands, docs links, calculator controls, copy buttons and version selector actually work |
| A17 | P0 | Metadata, internal links, `robots.txt`, sitemap, `llms.txt`, full `llms-full.txt` present |
| A18 | P0 | Fonts are legible/self-hosted, contrast/focus/zoom/mobile checks passed |
| A19 | P1 | Client-side documentation search works with deep-link results |
| A20 | P1 | Score gauge and severity spectrum accurate, labeled, accessible |
| A21 | P1 | Local export/share is accurate, works and communicates failures |
| A22 | P1 | Motion respects `prefers-reduced-motion` and never hides content |
| A23 | P1 | True original doc code blocks, copy actions and side navigation preserved |
| A24 | P1 | Design avoids AI-slop, has distinct editorial rhythm and usable information density |
| A25 | P1 | Every build and test claim backed by actual command result |
| A26 | P1 | Owner has a documented exact logo replacement location |

### 10.1 Final Arena evaluation weights

- **40% UX, typography, accessibility and user task completion** — multi-mode calculator and docs comfort matter more than spectacle.
- **25% correctness and scope** — all routes, vector math, independent MCP boundary, complete Markdown.
- **15% crawler / AI-fetch readability and technical information architecture** — static HTML, raw Markdown, metadata, discoverability.
- **10% engineering architecture and Cloudflare reliability** — static export, tests, no redirect loops.
- **10% visual craft and purposeful motion** — polish without decorative AI-slop.

## 11. Implementation references (verified documentation)

- Next static export / supported limitations: https://nextjs.org/docs/app/guides/static-exports
- Cloudflare Pages static Next.js deployment: https://developers.cloudflare.com/pages/framework-guides/nextjs/deploy-a-static-nextjs-site/
- Cloudflare Next guidance: https://developers.cloudflare.com/pages/framework-guides/nextjs/
- Exa Contents extraction behavior: https://exa.ai/products/contents
- Tavily advanced extraction behavior: https://help.tavily.com/articles/3363168593-extracting-web-content-using-tavily
- Tavily Extract overview: https://help.tavily.com/articles/8721959612-what-is-the-tavily-extract-api
- Firecrawl scraping Markdown/HTML and content filtering: https://github.com/firecrawl/firecrawl-docs/blob/main/advanced-scraping-guide.mdx
- FIRST official CVSS versions/specifications: https://www.first.org/cvss/
- Informal AI-slop design patterns to avoid (not proof of AI authorship): https://slopfix.getbags.app/checklist

---

# APPENDIX A — COMPLETE ENGLISH DOCUMENTATION, VERBATIM

**STRICT CONTENT TRANSFER:** All 31 files below are the full original source from the uploaded website ZIP. For each block, copy every byte of the Markdown payload (excluding the outer eight-backtick delimiter and provenance labels) to the indicated `content/docs/...` path. Also publish the same payload at its mapped `public/docs/...` fetch path. Do not shorten any content, including nested code fences and frontmatter. The document manifest above is authoritative. An SHA-256 content hash is supplied for identity validation.

## A.01 — `calculator.md`

**Destination:** `content/docs/calculator.md`  
**SHA-256 (UTF-8 original):** `e94d393589875131853ec48241d71341617b5312067e8d3463893aeb4b78959b`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Full Browser Calculator Guide
description: How to use the CVSS 2.0, 3.0, 3.1, and 4.0 full browser calculator, hybrid controls, inline vector editor, and result export.
---

The [CVSS Calculator](https://cvss.my.id/calculator/) evaluates vulnerability scores and vectors according to official FIRST specifications across CVSS 4.0, 3.1, 3.0, and 2.0.

## Version Routes

Each CVSS version has a dedicated static route:

- [CVSS v4.0 Calculator](https://cvss.my.id/calculator/v4.0) (default route)
- [CVSS v3.1 Calculator](https://cvss.my.id/calculator/v3.1)
- [CVSS v3.0 Calculator](https://cvss.my.id/calculator/v3.0)
- [CVSS v2.0 Calculator](https://cvss.my.id/calculator/v2.0)

## Metric Group Tabs

Instead of rendering dozens of controls in a single vertical wall, each version organizes its metrics into logical tabs:

- **CVSS v4.0**: `Base` (11 required metrics), `Threat` (1 optional metric), `Environmental` (14 optional metrics), `Supplemental` (6 optional contextual metrics).
- **CVSS v3.1 & v3.0**: `Base` (8 required metrics), `Temporal` (3 optional metrics), `Environmental` (11 optional metrics).
- **CVSS v2.0**: `Base` (6 required metrics), `Temporal` (3 optional metrics), `Environmental` (5 optional metrics).

### Hybrid Controls

- **Segmented radio buttons**: Used when a metric has 2 to 4 concise options (such as Attack Vector, Attack Complexity, or Scope). Supports keyboard selection and visible `:focus-visible` outlines.
- **Native `<select>` dropdowns**: Used when a metric has 5 or more options or verbose labels (such as CVSS v2.0 Target Distribution or Temporal/Environmental overrides with `Not Defined`).

## Incomplete State vs Numeric Score

A score is never shown as `0.0` merely because metrics have not been selected yet.

- While any mandatory Base metric is unselected, the Score Panel displays **`Incomplete`** and lists the exact count of missing Base metrics.
- Once all required Base metrics are selected, the Score Panel immediately computes the overall numeric score, qualitative severity label, active nomenclature (such as `CVSS-B` or `CVSS-BTE` in v4.0), and sub-score breakdown.

## Score Panel

The score panel appears above the metrics on mobile and beside them on desktop. It shows a score gauge, the qualitative severity rating, a 0-10 scale, and version-appropriate sub-scores: Impact and Exploitability for CVSS v2.0 and v3.x, and Base and Nomenclature for CVSS v4.0. The panel updates automatically as metric values change.

- **Share**: Copies a link containing the current canonical vector.
- **Show explanations**: Hides or shows the metric descriptions and the help text for the selected value.
- **Severity Levels**: The section below the calculator lists the qualitative rating bands for the active CVSS version.

## Inline Vector Editor & Out-of-Sync Protection

The **Vector String** input at the top of the workspace is editable at all times:

1. **Valid vector input**: Typing or pasting a valid vector for the current version updates metric controls across all tabs and recalculates the score. For a vector from another version, follow the link shown to open its calculator.
2. **Invalid vector draft**: If you mistype a metric code or paste a malformed string, the calculator:
   - Never overwrites or deletes the text you are typing.
   - Displays a specific validation message below the field explaining the exact token or missing metric error.
   - Marks the Score Panel as **Out of Sync (Showing Last Valid Score)** and offers a **Revert Vector Input** button.
   - Disables **Copy Vector** and **Copy Result** so you cannot accidentally copy an invalid or out-of-sync score.

## Action Buttons

- **Copy Vector**: Copies the canonical CVSS vector string to your clipboard.
- **Copy Result**: Copies a formatted plain-text summary (for example, `CVSS v4.0 Score: 9.3 (Critical) [CVSS-B] - CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N`).
- **Reset Group**: Resets only the currently open tab (`Threat`, `Temporal`, `Environmental`, or `Supplemental`) back to `Not Defined` (`X` / `ND`), or clears `Base` selections when on the `Base` tab.
- **Reset All**: Clears all metrics and restores the default state for the active CVSS version.

## Reference Tables

Qualitative severity ranges and the mandatory Base metric keys for each supported version are documented in [Base Metric Keys & Severity Ranges](/docs/cvss/metric-keys/).
````````

## A.02 — `cvss/examples.md`

**Destination:** `content/docs/cvss/examples.md`  
**SHA-256 (UTF-8 original):** `1bdc1ed725d6d77d1318ad4baadfd5e6fc00fb4b95cf393b44cdffcd64db73c9`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Worked Examples
description: Verified CVSS v4.0, v3.1, v3.0, and v2.0 vector examples with Base, Threat/Temporal, Environmental, and Supplemental breakdowns.
---

Below are worked vulnerability examples across all four CVSS versions. Click any calculator link to inspect or modify the vector live in the browser calculator.

## 1. Unauthenticated Remote Code Execution (CVSS v4.0)

A network-reachable service allows unauthenticated remote code execution (`AV:N/AC:L/AT:N/PR:N/UI:N`) with full compromise of the vulnerable host (`VC:H/VI:H/VA:H`), no subsequent system impact (`SC:N/SI:N/SA:N`), and public Proof-of-Concept exploit code (`E:P`).

- **Vector**: `CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N/E:P`
- **Base Score (`CVSS-B`)**: `9.3 (Critical)`
- **Base + Threat Score (`CVSS-BT`)**: `8.9 (High)`
- **Open in Calculator**: [Inspect in v4.0 Calculator](https://cvss.my.id/calculator/v4.0?vector=CVSS%3A4.0%2FAV%3AN%2FAC%3AL%2FAT%3AN%2FPR%3AN%2FUI%3AN%2FVC%3AH%2FVI%3AH%2FVA%3AH%2FSC%3AN%2FSI%3AN%2FSA%3AN%2FE%3AP)

## 2. Container Escape Affecting Host System (CVSS v4.0 + Supplemental)

An attacker with low privileges inside a container (`AV:L/AC:L/AT:N/PR:L/UI:N`) compromises both the container (`VC:H/VI:H/VA:H`) and the underlying host (`SC:H/SI:H/SA:H`). Supplemental metrics record that the vulnerability is automatable (`AU:Y`) with provider urgency `Red` (`U:Red`).

- **Vector**: `CVSS:4.0/AV:L/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/AU:Y/U:Red`
- **Score (`CVSS-B`)**: `9.3 (Critical)` (Supplemental metrics `AU:Y/U:Red` are preserved in the vector without changing the numeric score.)
- **Open in Calculator**: [Inspect in v4.0 Calculator](https://cvss.my.id/calculator/v4.0?vector=CVSS%3A4.0%2FAV%3AL%2FAC%3AL%2FAT%3AN%2FPR%3AL%2FUI%3AN%2FVC%3AH%2FVI%3AH%2FVA%3AH%2FSC%3AH%2FSI%3AH%2FSA%3AH%2FAU%3AY%2FU%3ARed)

## 3. Stored Cross-Site Scripting (CVSS v3.1)

An attacker with low privileges (`PR:L`) injects script that executes in a victim user's browser (`UI:R`, `S:C`) with low confidentiality and integrity impact (`C:L/I:L/A:N`).

- **Vector**: `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`
- **Base Score**: `5.4 (Medium)`
- **Open in Calculator**: [Inspect in v3.1 Calculator](https://cvss.my.id/calculator/v3.1?vector=CVSS%3A3.1%2FAV%3AN%2FAC%3AL%2FPR%3AL%2FUI%3AR%2FS%3AC%2FC%3AL%2FI%3AL%2FA%3AN)

## 4. Critical Network Service Compromise (CVSS v3.1 vs v3.0)

- **v3.1 Vector**: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` -> **`10.0 (Critical)`** ([Inspect in v3.1 Calculator](https://cvss.my.id/calculator/v3.1?vector=CVSS%3A3.1%2FAV%3AN%2FAC%3AL%2FPR%3AN%2FUI%3AN%2FS%3AC%2FC%3AH%2FI%3AH%2FA%3AH))
- **v3.0 Vector**: `CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` -> **`10.0 (Critical)`** ([Inspect in v3.0 Calculator](https://cvss.my.id/calculator/v3.0?vector=CVSS%3A3.0%2FAV%3AN%2FAC%3AL%2FPR%3AN%2FUI%3AN%2FS%3AC%2FC%3AH%2FI%3AH%2FA%3AH))

## 5. Legacy Network Buffer Overflow (CVSS v2.0)

- **Vector**: `AV:N/AC:L/Au:N/C:C/I:C/A:C/E:F/RL:OF/RC:C`
- **Base Score**: `10.0 (High)` (Remember: CVSS v2.0 highest severity rating is `High`.)
- **Temporal Score**: `8.3 (High)`
- **Open in Calculator**: [Inspect in v2.0 Calculator](https://cvss.my.id/calculator/v2.0?vector=AV%3AN%2FAC%3AL%2FAu%3AN%2FC%3AC%2FI%3AC%2FA%3AC%2FE%3AF%2FRL%3AOF%2FRC%3AC)
````````

## A.03 — `cvss/metric-glossary.md`

**Destination:** `content/docs/cvss/metric-glossary.md`  
**SHA-256 (UTF-8 original):** `18e1c9a4022b4954c3f0418db82a0ea9c39d4cba5a283a1a0caa89b8e105e744`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: CVSS Metric Glossary
description: Metric keys, plain-language definitions, and allowed values for CVSS 4.0, 3.1, 3.0, and 2.0.
---

Use this glossary to look up the metric keys and allowed values used in a CVSS vector. The definitions are concise working descriptions, not replacements for the official FIRST specifications.

Use the documentation search (Ctrl+K) to find a metric key such as `AV`, `PR`, or `CR`.

## Common abbreviations

| Key | Metric | Used in |
| --- | --- | --- |
| `AV` | Attack Vector (Access Vector in v2.0) | v2.0, v3.0, v3.1, v4.0 |
| `AC` | Attack Complexity (Access Complexity in v2.0) | v2.0, v3.0, v3.1, v4.0 |
| `PR` | Privileges Required | v3.0, v3.1, v4.0 |
| `UI` | User Interaction | v3.0, v3.1, v4.0 |
| `S` | Scope | v3.0, v3.1 |
| `C` / `I` / `A` | Confidentiality / Integrity / Availability impact | v2.0, v3.0, v3.1, v4.0* |
| `Au` | Authentication | v2.0 |
| `AT` | Attack Requirements | v4.0 |

\* CVSS v4.0 uses `VC`/`VI`/`VA` for the vulnerable system and `SC`/`SI`/`SA` for subsequent systems.


## CVSS v4.0

CVSS v4.0 splits impact between the vulnerable system and subsequent systems, and adds Threat, Environmental, and Supplemental groups.

### Base Metrics

- **`AV` Attack Vector**: Reflects the context by which vulnerability exploitation is possible. Allowed values: `N` Network, `A` Adjacent, `L` Local, `P` Physical.
- **`AC` Attack Complexity**: Captures measurable actions that must be taken by the attacker to actively evade or circumvent built-in security-enhancing conditions. Allowed values: `L` Low, `H` High.
- **`AT` Attack Requirements**: Captures the prerequisite deployment and execution conditions or variables of the vulnerable system that enable the attack. Allowed values: `N` None, `P` Present.
- **`PR` Privileges Required**: Describes the level of privileges an attacker must possess prior to successfully exploiting the vulnerability. Allowed values: `N` None, `L` Low, `H` High.
- **`UI` User Interaction**: Captures the requirement for a human user, other than the attacker, to participate in the successful compromise. Allowed values: `N` None, `P` Passive, `A` Active.
- **`VC` Vulnerable System Confidentiality**: Measures the impact to the confidentiality of information managed by the Vulnerable System. Allowed values: `H` High, `L` Low, `N` None.
- **`VI` Vulnerable System Integrity**: Measures the impact to integrity of a successfully exploited vulnerability on the Vulnerable System. Allowed values: `H` High, `L` Low, `N` None.
- **`VA` Vulnerable System Availability**: Measures the impact to the availability of the Vulnerable System resulting from a successfully exploited vulnerability. Allowed values: `H` High, `L` Low, `N` None.
- **`SC` Subsequent System Confidentiality**: Measures the impact to the confidentiality of information managed by Subsequent Systems. Allowed values: `H` High, `L` Low, `N` None.
- **`SI` Subsequent System Integrity**: Measures the impact to integrity of Subsequent Systems. Allowed values: `H` High, `L` Low, `N` None.
- **`SA` Subsequent System Availability**: Measures the impact to the availability of Subsequent Systems. Allowed values: `H` High, `L` Low, `N` None.

### Threat Metrics

- **`E` Exploit Maturity**: Measures the likelihood of the vulnerability being attacked based on the current state of exploit techniques or active exploitation. Allowed values: `X` Not Defined, `A` Attacked, `P` POC, `U` Unreported.

### Environmental Metrics

- **`CR` Confidentiality Requirements**: Customizes the assessment depending on the importance of Confidentiality to the affected asset. Allowed values: `X` Not Defined, `H` High, `M` Medium, `L` Low.
- **`IR` Integrity Requirements**: Customizes the assessment depending on the importance of Integrity to the affected asset. Allowed values: `X` Not Defined, `H` High, `M` Medium, `L` Low.
- **`AR` Availability Requirements**: Customizes the assessment depending on the importance of Availability to the affected asset. Allowed values: `X` Not Defined, `H` High, `M` Medium, `L` Low.
- **`MAV` Modified Attack Vector**: Overrides the Base Attack Vector based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` Network, `A` Adjacent, `L` Local, `P` Physical.
- **`MAC` Modified Attack Complexity**: Overrides the Base Attack Complexity based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `L` Low, `H` High.
- **`MAT` Modified Attack Requirements**: Overrides the Base Attack Requirements based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `P` Present.
- **`MPR` Modified Privileges Required**: Overrides the Base Privileges Required based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `L` Low, `H` High.
- **`MUI` Modified User Interaction**: Overrides the Base User Interaction based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `P` Passive, `A` Active.
- **`MVC` Modified Vulnerable System Confidentiality**: Overrides the Base Vulnerable System Confidentiality impact. Allowed values: `X` Not Defined, `H` High, `L` Low, `N` None.
- **`MVI` Modified Vulnerable System Integrity**: Overrides the Base Vulnerable System Integrity impact. Allowed values: `X` Not Defined, `H` High, `L` Low, `N` None.
- **`MVA` Modified Vulnerable System Availability**: Overrides the Base Vulnerable System Availability impact. Allowed values: `X` Not Defined, `H` High, `L` Low, `N` None.
- **`MSC` Modified Subsequent System Confidentiality**: Overrides the Base Subsequent System Confidentiality impact. Allowed values: `X` Not Defined, `H` High, `L` Low, `N` Negligible.
- **`MSI` Modified Subsequent System Integrity**: Overrides the Base Subsequent System Integrity impact, including potential human safety impact. Allowed values: `X` Not Defined, `S` Safety, `H` High, `L` Low, `N` Negligible.
- **`MSA` Modified Subsequent System Availability**: Overrides the Base Subsequent System Availability impact, including potential human safety impact. Allowed values: `X` Not Defined, `S` Safety, `H` High, `L` Low, `N` Negligible.

### Supplemental Metrics

- **`S` Safety**: Indicates whether exploiting the vulnerability may have Safety impact under IEC 61508. Does not change the numeric score. Allowed values: `X` Not Defined, `N` Negligible, `P` Present.
- **`AU` Automatable**: Captures whether an attacker can reliably automate exploitation events across multiple targets (steps 1-4 of kill chain). Does not change the numeric score. Allowed values: `X` Not Defined, `N` No, `Y` Yes.
- **`R` Recovery**: Describes the resilience of a system to recover services in terms of performance and availability after an attack. Does not change the numeric score. Allowed values: `X` Not Defined, `A` Automatic, `U` User, `I` Irrecoverable.
- **`V` Value Density**: Describes the resources that the attacker will gain control over with a single exploitation event. Does not change the numeric score. Allowed values: `X` Not Defined, `D` Diffuse, `C` Concentrated.
- **`RE` Vulnerability Response Effort**: Provides supplemental information on how difficult it is for consumers to provide an initial response. Does not change the numeric score. Allowed values: `X` Not Defined, `L` Low, `M` Moderate, `H` High.
- **`U` Provider Urgency**: Incorporates additional provider-supplied urgency assessment. Does not change the numeric score. Allowed values: `X` Not Defined, `Clear` Clear, `Green` Green, `Amber` Amber, `Red` Red.

## CVSS v3.1 and v3.0

The Base, Temporal, and Environmental metric keys and allowed values are the same for CVSS v3.1 and v3.0. The versions differ in some scoring formulas, rounding, and Environmental ModifiedImpact rules.

### Base Metrics

- **`AV` Attack Vector**: Reflects the context by which vulnerability exploitation is possible. Allowed values: `N` Network, `A` Adjacent, `L` Local, `P` Physical.
- **`AC` Attack Complexity**: Describes the conditions beyond the attacker’s control that must exist in order to exploit the vulnerability. Allowed values: `L` Low, `H` High.
- **`PR` Privileges Required**: Describes the level of privileges an attacker must possess before successfully exploiting the vulnerability. Allowed values: `N` None, `L` Low, `H` High.
- **`UI` User Interaction**: Captures the requirement for a human user, other than the attacker, to participate in the compromise. Allowed values: `N` None, `R` Required.
- **`S` Scope**: Captures whether a vulnerability in one vulnerable component impacts resources in components beyond its security scope. Allowed values: `U` Unchanged, `C` Changed.
- **`C` Confidentiality**: Measures the impact to the confidentiality of the information resources managed by a software component. Allowed values: `N` None, `L` Low, `H` High.
- **`I` Integrity**: Measures the impact to integrity of a successfully exploited vulnerability. Allowed values: `N` None, `L` Low, `H` High.
- **`A` Availability**: Measures the impact to the availability of the impacted component resulting from a successfully exploited vulnerability. Allowed values: `N` None, `L` Low, `H` High.

### Temporal Metrics

- **`E` Exploit Code Maturity**: Measures the likelihood of the vulnerability being attacked based on the current state of exploit techniques. Allowed values: `X` Not Defined, `U` Unproven, `P` Proof-of-Concept, `F` Functional, `H` High.
- **`RL` Remediation Level**: Prioritizes the remediation status of a vulnerability. Allowed values: `X` Not Defined, `O` Official Fix, `T` Temporary Fix, `W` Workaround, `U` Unavailable.
- **`RC` Report Confidence**: Measures the degree of confidence in the existence of the vulnerability and the credibility of the known technical details. Allowed values: `X` Not Defined, `U` Unknown, `R` Reasonable, `C` Confirmed.

### Environmental Metrics

- **`CR` Confidentiality Requirement**: Customizes the score depending on the importance of the affected IT asset to the organization in terms of Confidentiality. Allowed values: `X` Not Defined, `L` Low, `M` Medium, `H` High.
- **`IR` Integrity Requirement**: Customizes the score depending on the importance of the affected IT asset to the organization in terms of Integrity. Allowed values: `X` Not Defined, `L` Low, `M` Medium, `H` High.
- **`AR` Availability Requirement**: Customizes the score depending on the importance of the affected IT asset to the organization in terms of Availability. Allowed values: `X` Not Defined, `L` Low, `M` Medium, `H` High.
- **`MAV` Modified Attack Vector**: Overrides the Base Attack Vector based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` Network, `A` Adjacent, `L` Local, `P` Physical.
- **`MAC` Modified Attack Complexity**: Overrides the Base Attack Complexity based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `L` Low, `H` High.
- **`MPR` Modified Privileges Required**: Overrides the Base Privileges Required based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `L` Low, `H` High.
- **`MUI` Modified User Interaction**: Overrides the Base User Interaction based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `R` Required.
- **`MS` Modified Scope**: Overrides the Base Scope based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `U` Unchanged, `C` Changed.
- **`MC` Modified Confidentiality**: Overrides the Base Confidentiality impact based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `L` Low, `H` High.
- **`MI` Modified Integrity**: Overrides the Base Integrity impact based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `L` Low, `H` High.
- **`MA` Modified Availability**: Overrides the Base Availability impact based on specific characteristics of the user environment. Allowed values: `X` Not Defined, `N` None, `L` Low, `H` High.

## CVSS v2.0

CVSS v2.0 uses the Authentication (`Au`) metric and its own Base, Temporal, and Environmental keys. It has no Critical severity rating.

### Base Metrics

- **`AV` Access Vector**: Reflects how the vulnerability is exploited in terms of network or local proximity. Allowed values: `L` Local, `A` Adjacent Network, `N` Network.
- **`AC` Access Complexity**: Measures the complexity of the attack required to exploit the vulnerability once target access is gained. Allowed values: `H` High, `M` Medium, `L` Low.
- **`Au` Authentication**: Measures the number of times an attacker must authenticate to a target in order to exploit a vulnerability. Allowed values: `M` Multiple, `S` Single, `N` None.
- **`C` Confidentiality Impact**: Measures the impact on confidentiality of a successfully exploited vulnerability. Allowed values: `N` None, `P` Partial, `C` Complete.
- **`I` Integrity Impact**: Measures the impact to integrity of a successfully exploited vulnerability. Allowed values: `N` None, `P` Partial, `C` Complete.
- **`A` Availability Impact**: Measures the impact to availability of a successfully exploited vulnerability. Allowed values: `N` None, `P` Partial, `C` Complete.

### Temporal Metrics

- **`E` Exploitability**: Measures the current state of exploit techniques or code availability. Allowed values: `ND` Not Defined, `U` Unproven, `POC` Proof-of-Concept, `F` Functional, `H` High.
- **`RL` Remediation Level**: Measures the remediation status of a vulnerability. Allowed values: `ND` Not Defined, `OF` Official Fix, `TF` Temporary Fix, `W` Workaround, `U` Unavailable.
- **`RC` Report Confidence**: Measures the degree of confidence in the existence of the vulnerability and the credibility of its technical details. Allowed values: `ND` Not Defined, `UC` Unconfirmed, `UR` Uncorroborated, `C` Confirmed.

### Environmental Metrics

- **`CDP` Collateral Damage Potential**: Measures the potential for loss of life, physical assets, productivity, or revenue. Allowed values: `ND` Not Defined, `N` None, `L` Low, `LM` Low-Medium, `MH` Medium-High, `H` High.
- **`TD` Target Distribution**: Measures the proportion of vulnerable systems in the environment. Allowed values: `ND` Not Defined, `N` None, `L` Low, `M` Medium, `H` High.
- **`CR` Confidentiality Requirement**: Customizes the Environmental score based on the importance of Confidentiality to the affected asset. Allowed values: `ND` Not Defined, `L` Low, `M` Medium, `H` High.
- **`IR` Integrity Requirement**: Customizes the Environmental score based on the importance of Integrity to the affected asset. Allowed values: `ND` Not Defined, `L` Low, `M` Medium, `H` High.
- **`AR` Availability Requirement**: Customizes the Environmental score based on the importance of Availability to the affected asset. Allowed values: `ND` Not Defined, `L` Low, `M` Medium, `H` High.

## Related references

- [Base Metric Keys & Severity Ranges](/docs/cvss/metric-keys/)
- [Vector String Format & Rules](/docs/cvss/vectors/)
- [CVSS Overview](/docs/cvss/overview/)
- [CVSS v4.0 Specification](https://www.first.org/cvss/v4.0/specification-document)
- [CVSS v3.1 Specification](https://www.first.org/cvss/v3-1/specification-document)
- [CVSS v3.0 Specification](https://www.first.org/cvss/v3.0/specification-document)
- [CVSS v2.0 Guide](https://www.first.org/cvss/v2/guide)

````````

## A.04 — `cvss/metric-keys.md`

**Destination:** `content/docs/cvss/metric-keys.md`  
**SHA-256 (UTF-8 original):** `448d32236e82850b2785e6f0b82dd47c6d346becbc07891b65c03e7f2a2ea489`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Base Metric Keys & Severity Ranges
description: Reference tables for CVSS Base metric keys, allowed values, and qualitative severity score ranges across CVSS 4.0, 3.1, 3.0, and 2.0.
---

This page contains the reference tables for the qualitative severity scale and the mandatory Base metric keys for each supported CVSS version.

## Qualitative Severity Ranges

### CVSS v4.0, v3.1, and v3.0

| Rating | Score | Meaning |
| --- | --- | --- |
| None | `0.0` | No security impact on the target system under the tested prerequisites. |
| Low | `0.1-3.9` | Minimal impact, high privileges, or local physical access required. |
| Medium | `4.0-6.9` | Moderate impact, user interaction, or specific conditions needed. |
| High | `7.0-8.9` | Substantial confidentiality, integrity, or availability compromise. |
| Critical | `9.0-10.0` | Remote unauthenticated compromise without user interaction. |

### CVSS v2.0

| Rating | Score | Meaning |
| --- | --- | --- |
| Low | `0.0-3.9` | Minimal impact. Unauthorized information disclosure or local denial of service. |
| Medium | `4.0-6.9` | Moderate impact. Partial disclosure, integrity compromise, or disruption of service. |
| High | `7.0-10.0` | Complete compromise of system integrity, confidentiality, or availability. |

CVSS v2.0 does not define a None or Critical rating.

## Base Metric Keys

Base metric keys and values are case-sensitive. A complete Base vector must include every required metric for the selected CVSS version.

### CVSS v4.0 (11 Base metrics)

| Key | Metric | Allowed values |
| --- | --- | --- |
| `AV` | Attack Vector | `N` Network, `A` Adjacent, `L` Local, `P` Physical |
| `AC` | Attack Complexity | `L` Low, `H` High |
| `AT` | Attack Requirements | `N` None, `P` Present |
| `PR` | Privileges Required | `N` None, `L` Low, `H` High |
| `UI` | User Interaction | `N` None, `P` Passive, `A` Active |
| `VC` | Vulnerable System Confidentiality | `H` High, `L` Low, `N` None |
| `VI` | Vulnerable System Integrity | `H` High, `L` Low, `N` None |
| `VA` | Vulnerable System Availability | `H` High, `L` Low, `N` None |
| `SC` | Subsequent System Confidentiality | `H` High, `L` Low, `N` None |
| `SI` | Subsequent System Integrity | `H` High, `L` Low, `N` None |
| `SA` | Subsequent System Availability | `H` High, `L` Low, `N` None |

### CVSS v3.1 and v3.0 (8 Base metrics)

| Key | Metric | Allowed values |
| --- | --- | --- |
| `AV` | Attack Vector | `N` Network, `A` Adjacent, `L` Local, `P` Physical |
| `AC` | Attack Complexity | `L` Low, `H` High |
| `PR` | Privileges Required | `N` None, `L` Low, `H` High |
| `UI` | User Interaction | `N` None, `R` Required |
| `S` | Scope | `U` Unchanged, `C` Changed |
| `C` | Confidentiality | `N` None, `L` Low, `H` High |
| `I` | Integrity | `N` None, `L` Low, `H` High |
| `A` | Availability | `N` None, `L` Low, `H` High |

### CVSS v2.0 (6 Base metrics)

| Key | Metric | Allowed values |
| --- | --- | --- |
| `AV` | Access Vector | `L` Local, `A` Adjacent Network, `N` Network |
| `AC` | Access Complexity | `H` High, `M` Medium, `L` Low |
| `Au` | Authentication | `M` Multiple, `S` Single, `N` None |
| `C` | Confidentiality Impact | `N` None, `P` Partial, `C` Complete |
| `I` | Integrity Impact | `N` None, `P` Partial, `C` Complete |
| `A` | Availability Impact | `N` None, `P` Partial, `C` Complete |

For full vector syntax and ordering rules, see [Vector String Format & Rules](/docs/cvss/vectors/).
````````

## A.05 — `cvss/overview.md`

**Destination:** `content/docs/cvss/overview.md`  
**SHA-256 (UTF-8 original):** `be478f0dd3fe0821a791201231a50954be6bf14415f41891aff613d6f188545c`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: CVSS Version Comparison
description: Technical comparison of CVSS v4.0, v3.1, v3.0, and v2.0 metric groups, scoring models, and qualitative severity ratings.
---

The Common Vulnerability Scoring System (CVSS) has evolved across four major versions maintained by FIRST.Org. Because each version uses distinct metrics, equations, and rounding rules, scores from one version cannot be directly translated into another via a simple linear formula.

## Side-by-Side Overview

| Feature | CVSS v4.0 (2023) | CVSS v3.1 (2019) | CVSS v3.0 (2015) | CVSS v2.0 (2007) |
| --- | --- | --- | --- | --- |
| **Vector Prefix** | `CVSS:4.0/` | `CVSS:3.1/` | `CVSS:3.0/` | None (e.g. `AV:N/AC:L/...`) |
| **Mandatory Base Metrics** | 11 (`AV, AC, AT, PR, UI, VC, VI, VA, SC, SI, SA`) | 8 (`AV, AC, PR, UI, S, C, I, A`) | 8 (`AV, AC, PR, UI, S, C, I, A`) | 6 (`AV, AC, Au, C, I, A`) |
| **Time / Threat Group** | **Threat** (`E`) | **Temporal** (`E, RL, RC`) | **Temporal** (`E, RL, RC`) | **Temporal** (`E, RL, RC`) |
| **Environmental Group** | `CR, IR, AR` + 11 Modified Base (`MAV..MSA`, including `S` on `MSI`/`MSA`) | `CR, IR, AR` + 8 Modified Base (`MAV..MA`) | `CR, IR, AR` + 8 Modified Base (`MAV..MA`) | `CDP, TD, CR, IR, AR` |
| **Supplemental Group** | Yes (`S, AU, R, V, RE, U`) - does not alter numeric score | No | No | No |
| **Cross-System Impact** | Explicit Vulnerable (`VC/VI/VA`) vs Subsequent (`SC/SI/SA`) metrics | Single binary `Scope` (`S:U` / `S:C`) | Single binary `Scope` (`S:U` / `S:C`) | Not modeled |
| **Prerequisite Conditions** | Dedicated `Attack Requirements` (`AT:N` / `AT:P`) | Folded into `Attack Complexity` (`AC`) | Folded into `Attack Complexity` (`AC`) | Folded into `Access Complexity` (`AC`) |
| **User Interaction** | 3 levels (`None`, `Passive`, `Active`) | 2 levels (`None`, `Required`) | 2 levels (`None`, `Required`) | Folded into `AC` |
| **Scoring Algorithm** | MacroVector equivalence classes (`eq1..eq6`) + lookup + severity distance interpolation | Algebraic Impact & Exploitability sub-formulas with `Roundup` | Algebraic Impact & Exploitability sub-formulas with `Roundup` | Algebraic Base, Temporal, and Environmental equations with standard 1-decimal rounding |
| **Severity Scale** | `None`, `Low`, `Medium`, `High`, `Critical` | `None`, `Low`, `Medium`, `High`, `Critical` | `None`, `Low`, `Medium`, `High`, `Critical` | `Low`, `Medium`, `High` (**no `None` or `Critical`**) |

## Qualitative Severity Rating Scales

### CVSS v4.0, v3.1, and v3.0

| Severity Rating | Score Range |
| --- | --- |
| **None** | `0.0` |
| **Low** | `0.1 - 3.9` |
| **Medium** | `4.0 - 6.9` |
| **High** | `7.0 - 8.9` |
| **Critical** | `9.0 - 10.0` |

### CVSS v2.0

CVSS v2.0 defines only three qualitative severity levels in the official FIRST v2 guide:

| Severity Rating | Score Range |
| --- | --- |
| **Low** | `0.0 - 3.9` |
| **Medium** | `4.0 - 6.9` |
| **High** | `7.0 - 10.0` |

Never assign a `Critical` or `None` severity label to a CVSS v2.0 score.
````````

## A.06 — `cvss/v2.md`

**Destination:** `content/docs/cvss/v2.md`  
**SHA-256 (UTF-8 original):** `3e964263785a2f29f340c5273b7ff064da9af9d542e7c14041141cba781cb63d`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: CVSS v2.0 Reference
description: Complete metric definitions, equations, vector rules, and the 3-level qualitative severity scale for CVSS v2.0.
---

**CVSS v2.0** (published June 2007) remains present in legacy advisories, NVD historical records, and compliance baselines. Unlike v3.x and v4.0, CVSS v2.0 vectors do not use a `CVSS:x.y/` prefix, use `ND` instead of `X` for Not Defined optional metrics, and use a **three-level qualitative severity scale (`Low`, `Medium`, `High`) without `Critical` or `None`**.

Open the [CVSS v2.0 Calculator](https://cvss.my.id/calculator/v2.0).

## Base Metrics (6 Mandatory)

| Metric | Code | Values & Weights |
| --- | --- | --- |
| **Access Vector** | `AV` | `L` (Local = `0.395`), `A` (Adjacent Network = `0.646`), `N` (Network = `1.000`) |
| **Access Complexity** | `AC` | `H` (High = `0.35`), `M` (Medium = `0.61`), `L` (Low = `0.71`) |
| **Authentication** | `Au` | `M` (Multiple = `0.45`), `S` (Single = `0.56`), `N` (None = `0.704`) |
| **Confidentiality Impact** | `C` | `N` (None = `0.000`), `P` (Partial = `0.275`), `C` (Complete = `0.660`) |
| **Integrity Impact** | `I` | `N` (None = `0.000`), `P` (Partial = `0.275`), `C` (Complete = `0.660`) |
| **Availability Impact** | `A` | `N` (None = `0.000`), `P` (Partial = `0.275`), `C` (Complete = `0.660`) |

> [!NOTE]
> The Authentication metric code in CVSS v2.0 is case-sensitive (`Au`, with a lowercase `u`).

## Temporal Metrics (3 Optional)

Default value when omitted is `ND` (`Not Defined` = `1.00`):

- **Exploitability (`E`)**: `U` (Unproven = `0.85`), `POC` (Proof-of-Concept = `0.90`), `F` (Functional = `0.95`), `H` (High = `1.00`), `ND` (Not Defined = `1.00`)
- **Remediation Level (`RL`)**: `OF` (Official Fix = `0.87`), `TF` (Temporary Fix = `0.90`), `W` (Workaround = `0.95`), `U` (Unavailable = `1.00`), `ND` (Not Defined = `1.00`)
- **Report Confidence (`RC`)**: `UC` (Unconfirmed = `0.90`), `UR` (Uncorroborated = `0.95`), `C` (Confirmed = `1.00`), `ND` (Not Defined = `1.00`)

## Environmental Metrics (5 Optional)

- **Collateral Damage Potential (`CDP`)**: `N` (`0.0`), `L` (`0.1`), `LM` (`0.3`), `MH` (`0.4`), `H` (`0.5`), `ND` (`0.0`)
- **Target Distribution (`TD`)**: `N` (`0.00`), `L` (`0.25`), `M` (`0.75`), `H` (`1.00`), `ND` (`1.00`)
- **Security Requirements (`CR, IR, AR`)**: `L` (`0.5`), `M` (`1.0`), `H` (`1.51`), `ND` (`1.0`)

## Official CVSS v2.0 Equations

### Base Score

1. `Impact = 10.41 * (1 - (1 - C) * (1 - I) * (1 - A))`
2. `Exploitability = 20 * AV * AC * Au`
3. `f(Impact) = 0` if `Impact == 0`, else `1.176`
4. `BaseScore = round_to_1_decimal(((0.6 * Impact) + (0.4 * Exploitability) - 1.5) * f(Impact))`

### Temporal Score

`TemporalScore = round_to_1_decimal(BaseScore * E * RL * RC)`

### Environmental Score

1. `AdjustedImpact = min(10, 10.41 * (1 - (1 - C * CR) * (1 - I * IR) * (1 - A * AR)))`
2.Compute `AdjustedBase` using `AdjustedImpact` in place of `Impact` inside the Base equation (rounded to 1 decimal).
3. `AdjustedTemporal = round_to_1_decimal(AdjustedBase * E * RL * RC)`
4. `EnvironmentalScore = round_to_1_decimal((AdjustedTemporal + (10 - AdjustedTemporal) * CDP) * TD)`

## Qualitative Severity Scale

Per Section 5 of the [FIRST CVSS v2.0 Guide](https://www.first.org/cvss/v2/guide):

- **`Low`**: `0.0 - 3.9`
- **`Medium`**: `4.0 - 6.9`
- **`High`**: `7.0 - 10.0`
````````

## A.07 — `cvss/v3.0.md`

**Destination:** `content/docs/cvss/v3.0.md`  
**SHA-256 (UTF-8 original):** `742192d5890cf1e08bd614a24da2067099d3898188f86a7a151312b38c493334`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: CVSS v3.0 Reference
slug: docs/cvss/v3.0
description: Complete metric definitions, formulas, and exact mathematical differences between CVSS v3.0 and CVSS v3.1.
---

**CVSS v3.0** (published June 2015) introduced the `Scope` metric (`S:U` / `S:C`), replaced `Authentication` (`Au`) with `Privileges Required` (`PR`), and added `User Interaction` (`UI`). While CVSS v3.0 and v3.1 share the same metric names and Base/Temporal weights, **their Environmental `ModifiedImpact` formula when `MS:C` differs mathematically**.

Open the [CVSS v3.0 Calculator](https://cvss.my.id/calculator/v3.0).

## Exact Mathematical Difference from CVSS v3.1

In **CVSS v3.0**, when Modified Scope is Changed (`MS:C`), `ModifiedImpact` is defined as:

```text
ModifiedImpact (v3.0) = 7.52 * (MISS - 0.029) - 3.25 * (MISS - 0.02)^15
```

In **CVSS v3.1**, FIRST updated that equation so `ModifiedImpact` scales continuously up to `MISS = 0.915`:

```text
ModifiedImpact (v3.1) = 7.52 * (MISS - 0.029) - 3.25 * (MISS * 0.9731 - 0.02)^13
```

Because of this difference, a vector with `MS:C` and high security requirements (`CR:H/IR:H/AR:H`) can produce different Environmental scores in v3.0 vs v3.1. For example:

```text
CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H/CR:H/IR:H/AR:H/MS:C
```

Both our `packages/cvss-core` library and the browser calculator preserve this version-specific formula rather than aliasing v3.0 to v3.1.

## Vector Prefix & Metric Groups

- **Prefix**: `CVSS:3.0/`
- **Base Group (8 mandatory)**: `AV, AC, PR, UI, S, C, I, A`
- **Temporal Group (3 optional)**: `E, RL, RC`
- **Environmental Group (11 optional)**: `CR, IR, AR, MAV, MAC, MPR, MUI, MS, MC, MI, MA`

See the [CVSS v3.1 Reference](/docs/cvss/v3.1/) for the shared Base and Temporal weight tables.
````````

## A.08 — `cvss/v3.1.md`

**Destination:** `content/docs/cvss/v3.1.md`  
**SHA-256 (UTF-8 original):** `c3a60160df8aebfa22fb592695d3e9bfece81d50508ebff862d42d69ad4b7112`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: CVSS v3.1 Reference
slug: docs/cvss/v3.1
description: Complete metric definitions, Scope rules, Temporal and Environmental formulas, and Appendix A Roundup specification for CVSS v3.1.
---

**CVSS v3.1** (published June 2019) clarifies guidance from v3.0, refines the **Environmental `ModifiedImpact`** formula for Scope Changed (`MS:C`), and defines the exact floating-point safe **`Roundup`** function in Appendix A (`roundup(x) = ceil(round_to_nearest_int(x * 100000) / 10000) / 10`).

Open the [CVSS v3.1 Calculator](https://cvss.my.id/calculator/v3.1).

## Base Metrics (8 Mandatory)

| Metric | Code | Values |
| --- | --- | --- |
| **Attack Vector** | `AV` | `N` (Network = `0.85`), `A` (Adjacent = `0.62`), `L` (Local = `0.55`), `P` (Physical = `0.20`) |
| **Attack Complexity** | `AC` | `L` (Low = `0.77`), `H` (High = `0.44`) |
| **Privileges Required** | `PR` | `N` (None = `0.85`), `L` (Low = `0.62` when `S:U`, `0.68` when `S:C`), `H` (High = `0.27` when `S:U`, `0.50` when `S:C`) |
| **User Interaction** | `UI` | `N` (None = `0.85`), `R` (Required = `0.62`) |
| **Scope** | `S` | `U` (Unchanged), `C` (Changed) |
| **Confidentiality** | `C` | `N` (None = `0.00`), `L` (Low = `0.22`), `H` (High = `0.56`) |
| **Integrity** | `I` | `N` (None = `0.00`), `L` (Low = `0.22`), `H` (High = `0.56`) |
| **Availability** | `A` | `N` (None = `0.00`), `L` (Low = `0.22`), `H` (High = `0.56`) |

## Temporal Metrics (3 Optional)

When omitted or set to `X` (`Not Defined`), each Temporal metric defaults to a weight of `1.00`:

- **Exploit Code Maturity (`E`)**: `X` (`1.00`), `U` (Unproven = `0.91`), `P` (Proof-of-Concept = `0.94`), `F` (Functional = `0.97`), `H` (High = `1.00`)
- **Remediation Level (`RL`)**: `X` (`1.00`), `O` (Official Fix = `0.95`), `T` (Temporary Fix = `0.96`), `W` (Workaround = `0.97`), `U` (Unavailable = `1.00`)
- **Report Confidence (`RC`)**: `X` (`1.00`), `U` (Unknown = `0.92`), `R` (Reasonable = `0.96`), `C` (Confirmed = `1.00`)

## Environmental Metrics (11 Optional)

- **Security Requirements (`CR, IR, AR`)**: `X` (`1.0`), `L` (Low = `0.5`), `M` (Medium = `1.0`), `H` (High = `1.5`).
- **Modified Base Metrics (`MAV, MAC, MPR, MUI, MS, MC, MI, MA`)**: Override the corresponding Base metric when set to any value other than `X` (`Not Defined`).

## Exact Formulas (CVSS v3.1)

### Base Score

1. `ISS = 1 - ((1 - C) * (1 - I) * (1 - A))`
2. If `S == U`: `Impact = 6.42 * ISS`
3. If `S == C`: `Impact = 7.52 * (ISS - 0.029) - 3.25 * (ISS - 0.02)^15`
4. `Exploitability = 8.22 * AV * AC * PR * UI`
5. If `Impact <= 0`, `BaseScore = 0.0`.
6. Otherwise:
   - If `S == U`: `BaseScore = Roundup(min(Impact + Exploitability, 10))`
   - If `S == C`: `BaseScore = Roundup(min(1.08 * (Impact + Exploitability), 10))`

### Temporal Score

`TemporalScore = Roundup(BaseScore * E * RL * RC)`

### Environmental Score (v3.1 Specific)

1. `MISS = min(1 - (1 - MC * CR) * (1 - MI * IR) * (1 - MA * AR), 0.915)`
2. If `MS == U`: `ModifiedImpact = 6.42 * MISS`
3. If `MS == C`: `ModifiedImpact = 7.52 * (MISS - 0.029) - 3.25 * (MISS * 0.9731 - 0.02)^13`
4. `ModifiedExploitability = 8.22 * MAV * MAC * MPR * MUI`
5. If `ModifiedImpact <= 0`, `EnvironmentalScore = 0.0`.
6. Otherwise:
   - If `MS == U`: `EnvironmentalScore = Roundup(Roundup(min(ModifiedImpact + ModifiedExploitability, 10)) * E * RL * RC)`
   - If `MS == C`: `EnvironmentalScore = Roundup(Roundup(min(1.08 * (ModifiedImpact + ModifiedExploitability), 10)) * E * RL * RC)`
````````

## A.09 — `cvss/v4.md`

**Destination:** `content/docs/cvss/v4.md`  
**SHA-256 (UTF-8 original):** `b334b525feb03aeb55a750c08b4fd4a9672fe989735949943668786fc9fa8bd3`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: CVSS v4.0 Reference
description: Complete metric reference, nomenclature rules, and MacroVector scoring architecture for CVSS v4.0.
---

**CVSS v4.0** is the current standard published by FIRST.Org. It replaces the binary `Scope` switch from v3.x with explicit impact metrics for both the **Vulnerable System** (`VC, VI, VA`) and **Subsequent Systems** (`SC, SI, SA`), separates prerequisite conditions into **Attack Requirements (`AT`)**, refines **User Interaction (`UI`)** into three levels, and introduces **Supplemental** metrics alongside official score nomenclature (`CVSS-B`, `CVSS-BT`, `CVSS-BE`, `CVSS-BTE`).

Open the [CVSS v4.0 Calculator](https://cvss.my.id/calculator/v4.0).

## Official Nomenclature

CVSS v4.0 requires indicating which metric groups contributed to a numeric score:

- **`CVSS-B`**: Base metrics only.
- **`CVSS-BT`**: Base + Threat metrics.
- **`CVSS-BE`**: Base + Environmental metrics.
- **`CVSS-BTE`**: Base + Threat + Environmental metrics.

**Supplemental metrics (`S, AU, R, V, RE, U`) never change the numeric score or the nomenclature label.** They are recorded in the canonical vector string to provide structured context for incident responders.

## Base Metrics (11 Mandatory)

All 11 Base metrics must be specified before a CVSS v4.0 vector is valid.

### Exploitability Metrics

| Metric | Code | Values | Description |
| --- | --- | --- | --- |
| **Attack Vector** | `AV` | `N` (Network), `A` (Adjacent), `L` (Local), `P` (Physical) | Context by which vulnerability exploitation is possible. |
| **Attack Complexity** | `AC` | `L` (Low), `H` (High) | Effort required by the attacker to evade or circumvent defensive or security-enhancing technologies. |
| **Attack Requirements** | `AT` | `N` (None), `P` (Present) | Deployment and execution conditions or variables of the vulnerable system that enable the attack. |
| **Privileges Required** | `PR` | `N` (None), `L` (Low), `H` (High) | Level of privileges an attacker must possess prior to successfully exploiting the vulnerability. |
| **User Interaction** | `UI` | `N` (None), `P` (Passive), `A` (Active) | Whether exploitation requires participation from a human user other than the attacker. |

### Vulnerable System Impact Metrics

| Metric | Code | Values |
| --- | --- | --- |
| **Vulnerable System Confidentiality** | `VC` | `H` (High), `L` (Low), `N` (None) |
| **Vulnerable System Integrity** | `VI` | `H` (High), `L` (Low), `N` (None) |
| **Vulnerable System Availability** | `VA` | `H` (High), `L` (Low), `N` (None) |

### Subsequent System Impact Metrics

| Metric | Code | Values |
| --- | --- | --- |
| **Subsequent System Confidentiality** | `SC` | `H` (High), `L` (Low), `N` (None) |
| **Subsequent System Integrity** | `SI` | `H` (High), `L` (Low), `N` (None) |
| **Subsequent System Availability** | `SA` | `H` (High), `L` (Low), `N` (None) |

## Threat Metric (1 Optional)

| Metric | Code | Values | Default (`X`) |
| --- | --- | --- | --- |
| **Exploit Maturity** | `E` | `X` (Not Defined), `A` (Attacked), `P` (Proof-of-Concept), `U` (Unreported) | When `E:X` is selected (or omitted), the calculator assumes `A` (Attacked) as the worst-case default. |

## Environmental Metrics (14 Optional)

- **Security Requirements (`CR, IR, AR`)**: `X` (Not Defined, treated as `H`), `H` (High), `M` (Medium), `L` (Low).
- **Modified Base Metrics**: `MAV, MAC, MAT, MPR, MUI, MVC, MVI, MVA, MSC, MSI, MSA`.
- **Safety Impact Values on Subsequent Systems**: `MSI` (Modified Subsequent System Integrity) and `MSA` (Modified Subsequent System Availability) include an additional **`S` (Safety)** option above `H` when exploitation can cause physical human safety harm.

## Supplemental Metrics (6 Optional)

Supplemental metrics are purely informational and **never alter the numeric CVSS v4.0 score**:

- **Safety (`S`)**: `X` (Not Defined), `N` (Negligible), `P` (Present)
- **Automatable (`AU`)**: `X` (Not Defined), `N` (No), `Y` (Yes)
- **Recovery (`R`)**: `X` (Not Defined), `A` (Automatic), `U` (User), `I` (Irrecoverable)
- **Value Density (`V`)**: `X` (Not Defined), `D` (Diffuse), `C` (Concentrated)
- **Vulnerability Response Effort (`RE`)**: `X` (Not Defined), `L` (Low), `M` (Medium), `H` (High)
- **Provider Urgency (`U`)**: `X` (Not Defined), `Clear`, `Green`, `Amber`, `Red`

## How CVSS v4.0 MacroVector Scoring Works

Unlike v3.x algebraic formulas, CVSS v4.0 maps the effective metrics (where Modified Environmental metrics override Base metrics when defined) into a 6-digit **MacroVector** (`eq1` through `eq6`):

1. **`eq1`** (0-2): Exploitability prerequisites (`AV, PR, UI`).
2. **`eq2`** (0-1): Complexity and requirements (`AC, AT`).
3. **`eq3`** (0-2): Vulnerable system impact (`VC, VI, VA`).
4. **`eq4`** (0-2): Subsequent system impact (`SC, SI, SA`, including `MSI:S` and `MSA:S`).
5. **`eq5`** (0-2): Threat Exploit Maturity (`E`: `A=0`, `P=1`, `U=2`).
6. **`eq6`** (0-1): Combined Vulnerable Impact and Security Requirements (`VC/VI/VA` with `CR/IR/AR`).

If all six effective impact metrics (`VC, VI, VA, SC, SI, SA`) are `N`, the score is `0.0 (None)`. Otherwise, the engine looks up the baseline score of the active MacroVector in the official FIRST 270-entry lookup table, computes the severity distance from the highest-severity vectors in the same MacroVector, and interpolates against the next lower MacroVector across `eq1..eq6`, rounding to one decimal place.
````````

## A.10 — `cvss/vectors.md`

**Destination:** `content/docs/cvss/vectors.md`  
**SHA-256 (UTF-8 original):** `f9056f6dc6600ce5b01ea0bb0ce85f50b10ab1e96448ee80443ae8057693d796`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Vector String Format
description: Syntax, canonical metric ordering, prefix rules, and validation checks for CVSS v4.0, v3.1, v3.0, and v2.0 vector strings.
---

A CVSS vector string is a compact, machine-readable text representation of the metric values selected for a vulnerability. Both the [full browser calculator](https://cvss.my.id/calculator/) and the MCP server (`cvss-calculator-mcp`) validate vectors against the official FIRST specifications.

## Structure by Version

### CVSS v4.0

- **Prefix**: `CVSS:4.0/`
- **Mandatory Base order** (11 metrics): `AV, AC, AT, PR, UI, VC, VI, VA, SC, SI, SA`
- **Optional Threat order**: `E`
- **Optional Environmental order**: `CR, IR, AR, MAV, MAC, MAT, MPR, MUI, MVC, MVI, MVA, MSC, MSI, MSA`
- **Optional Supplemental order**: `S, AU, R, V, RE, U`
- **Canonical Serialization Rule**: Mandatory Base metrics are always present in exact order. Optional metrics set to `X` (`Not Defined`) are omitted from the serialized canonical vector string.

Example canonical v4.0 vector:

```text
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N/E:P/CR:H/MAV:A/S:P/AU:Y/U:Red
```

### CVSS v3.1 and v3.0

- **Prefix**: `CVSS:3.1/` or `CVSS:3.0/`
- **Mandatory Base order** (8 metrics): `AV, AC, PR, UI, S, C, I, A`
- **Optional Temporal order**: `E, RL, RC`
- **Optional Environmental order**: `CR, IR, AR, MAV, MAC, MPR, MUI, MS, MC, MI, MA`
- **Canonical Serialization Rule**: Mandatory Base metrics are always emitted; optional metrics with value `X` are omitted.

Example canonical v3.1 vector:

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H/E:F/RL:O/RC:C
```

### CVSS v2.0

- **Prefix**: None. (Parentheses around the vector such as `(AV:N/AC:L/Au:N/C:P/I:P/A:P)` are accepted by the parser and normalized to bare `AV:N/...` format.)
- **Mandatory Base order** (6 metrics): `AV, AC, Au, C, I, A`
- **Optional Temporal order**: `E, RL, RC`
- **Optional Environmental order**: `CDP, TD, CR, IR, AR`
- **Canonical Serialization Rule**: Mandatory Base metrics are always emitted; optional metrics with value `ND` are omitted.

Example canonical v2.0 vector:

```text
AV:N/AC:L/Au:N/C:C/I:C/A:C/E:F/RL:OF/RC:C
```

## Validation Errors Caught by the Parser

Our parser rejects malformed vectors with explicit, actionable error messages:

1. **Unrecognized or missing version prefix**: e.g., `CVSS:5.0/...` or missing `AV:` in v2.0.
2. **Duplicate metric keys**: e.g., specifying `AV:N` and `AV:L` in the same string.
3. **Unknown metric abbreviation**: e.g., using `Au` inside a v3.1 vector or `S` inside a v2.0 vector.
4. **Invalid metric value**: e.g., `AV:X` on a mandatory Base metric or `UI:R` in a v4.0 vector (where v4.0 uses `UI:P` or `UI:A`).
5. **Incomplete Base group**: Any vector missing one or more mandatory Base metrics for its version.
````````

## A.11 — `getting-started.md`

**Destination:** `content/docs/getting-started.md`  
**SHA-256 (UTF-8 original):** `798ccd2016a2ace74bd8ed390252b0371b1dad4f398522d9a52a3639165ae8e5`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Getting Started
description: Quick start guide for scoring vulnerabilities in the browser calculator or via the CVSS MCP server.
---

CVSS.my.id provides two ways to calculate Common Vulnerability Scoring System (CVSS) scores:

1. **Interactive Browser Calculator** for full multi-group scoring (Base, Threat/Temporal, Environmental, and Supplemental).
2. **MCP Server (`cvss-calculator-mcp`)** for automated **Base Score** calculation inside AI coding agents and security triage pipelines.

## Which Tool Should I Use?

| Requirement | Recommended Tool | Supported Metric Groups |
| --- | --- | --- |
| Interactive vulnerability triage with Threat, Temporal, Environmental, or Supplemental metrics | [Full Browser Calculator](/docs/calculator/) | Base, Threat, Temporal, Environmental, Supplemental (v4.0, v3.1, v3.0, v2.0) |
| Exporting canonical vector strings or formatted score summaries | [Full Browser Calculator](/docs/calculator/) | Full vector serialization across all metric groups |
| Automated Base Score calculation inside Codex, Claude Code, Cursor, or OpenCode | [Local MCP (`stdio`)](/docs/mcp/local/) | Base Score only (v4.0, v3.1, v3.0, v2.0) |
| Zero-install MCP testing from a cloud workspace or browser-hosted agent | [Remote MCP (HTTP)](/docs/mcp/remote/) | Base Score only (5 requests/min/IP limit) |

## Quick Start: Full Browser Calculator

1. Open the calculator for your target CVSS version:
   - [CVSS v4.0 Calculator](https://cvss.my.id/calculator/v4.0)
   - [CVSS v3.1 Calculator](https://cvss.my.id/calculator/v3.1)
   - [CVSS v3.0 Calculator](https://cvss.my.id/calculator/v3.0)
   - [CVSS v2.0 Calculator](https://cvss.my.id/calculator/v2.0)
2. Select all required **Base** metrics. Until every mandatory Base metric is chosen, the score displays `Incomplete` rather than `0.0`.
3. Optionally switch tabs to add **Threat** (v4.0), **Temporal** (v3.x/v2.0), **Environmental**, or **Supplemental** (v4.0) metrics.
4. Or paste an existing CVSS vector directly into the **Inline Vector String** input at the top of the calculator to populate the form and compute the score immediately.
5. Use **Copy Vector** or **Copy Result** once all required Base metrics are valid.

## Quick Start: Local MCP Server (Recommended for Agents)

Install the standalone Go binary on Linux or macOS:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

Verify the binary is on your `PATH`:

```bash
cvss-calculator-mcp --version
```

Then register `cvss-calculator-mcp` in your AI client using our [Client Setup Guides](/docs/mcp/clients/).
````````

## A.12 — `index.md`

**Destination:** `content/docs/index.md`  
**SHA-256 (UTF-8 original):** `163099fbea4a1418cb075b3d6eb57c7f20f1e6f3e1ffa32cbf0a408cc274b450`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Documentation Overview
description: Technical reference for CVSS 2.0, 3.0, 3.1, and 4.0 scoring, vector strings, the full browser calculator, and the Base-only MCP server.
---

Welcome to the **CVSS.my.id** documentation. This documentation covers two static products and one open-source Model Context Protocol (MCP) server:

1. [**CVSS Calculator**](https://cvss.my.id/calculator/): Score CVSS 4.0, 3.1, 3.0, and 2.0 vulnerabilities across Base, Threat/Temporal, Environmental, and Supplemental metric groups.
2. **Documentation & Product Hub**: Reference guides for CVSS version differences, vector syntax, scoring formulas, and AI agent integrations.
3. **CVSS Calculator MCP (`cvss-calculator-mcp`)**: Open-source Go MCP server for **Base Score only** across CVSS 2.0, 3.0, 3.1, and 4.0. Runs locally over `stdio` or via the free public Streamable HTTP endpoint.

## Documentation Sections

### Getting Started & Calculator

- [Getting Started](/docs/getting-started/): Choose the right CVSS version and tool for your workflow.
- [Full Browser Calculator Guide](/docs/calculator/): Metric group tabs, inline vector editing, keyboard navigation, and result export.

### CVSS Reference

- [Version Comparison](/docs/cvss/overview/): Side-by-side comparison of CVSS 4.0, 3.1, 3.0, and 2.0.
- [CVSS v4.0 Reference](/docs/cvss/v4/): Base, Threat, Environmental, Supplemental groups, MacroVector lookup, and `CVSS-B` / `CVSS-BT` / `CVSS-BE` / `CVSS-BTE` nomenclature.
- [CVSS v3.1 Reference](/docs/cvss/v3.1/): Base, Temporal, and Environmental formulas, Scope rules, and `Roundup` behavior.
- [CVSS v3.0 Reference](/docs/cvss/v3.0/): Historical 3.0 formulas and exact differences from 3.1.
- [CVSS v2.0 Reference](/docs/cvss/v2/): Access Vector, Authentication (`Au`), Temporal, Environmental, and the 3-tier severity scale (`Low`, `Medium`, `High`).
- [Base Metric Keys & Severity Ranges](/docs/cvss/metric-keys/): Key shorthand, allowed Base values, and qualitative score bands for each version.
- [CVSS Metric Glossary](/docs/cvss/metric-glossary/): Definitions and allowed values for every metric key across CVSS 4.0, 3.1, 3.0, and 2.0.
- [Vector String Format](/docs/cvss/vectors/): Prefix rules, metric ordering, mandatory vs optional metrics, and validation rules.
- [Worked Examples](/docs/cvss/examples/): Real-world vulnerability vectors across all four CVSS versions.

### MCP Server (`cvss-calculator-mcp`)

- [MCP Overview](/docs/mcp/): Local `stdio` vs Remote Streamable HTTP comparison and Base-only scope.
- [Local MCP Setup (stdio)](/docs/mcp/local/): Recommended zero-rate-limit installation for Linux and macOS.
- [Remote MCP Setup (HTTP)](/docs/mcp/remote/): Connect to `https://cvss.my.id/mcp` and understand the `5 requests per minute per IP` policy.
- [MCP Tool Reference](/docs/mcp/tools/): Input schema, output fields, and example calls for `calculate_cvss`.
- [Security, Privacy & Rate Limits](/docs/mcp/security/): Offline guarantees, transport notes, and source code verification.
- [Client Setup Guides](/docs/mcp/clients/): Setup instructions for a selection of MCP clients and editors.

## Standards Attribution

CVSS (Common Vulnerability Scoring System) is a free and open industry standard owned and managed by **FIRST.Org, Inc.** (Forum of Incident Response and Security Teams). Official specifications are published at:

- [FIRST CVSS v4.0 Specification](https://www.first.org/cvss/v4.0/specification-document)
- [FIRST CVSS v3.1 Specification](https://www.first.org/cvss/v3-1/specification-document)
- [FIRST CVSS v3.0 Specification](https://www.first.org/cvss/v3.0/specification-document)
- [FIRST CVSS v2.0 Complete Guide](https://www.first.org/cvss/v2/guide)
````````

## A.13 — `mcp/clients/claude-code.md`

**Destination:** `content/docs/mcp/clients/claude-code.md`  
**SHA-256 (UTF-8 original):** `56eea16522e422f6f2156d2744dccb05587a2a36c999836ffda1370eb62e549f`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Claude Code MCP Setup
description: Configure cvss-calculator-mcp in Anthropic Claude Code using Local stdio or Remote Streamable HTTP.
---

This guide shows how to configure `cvss-calculator-mcp` in **Claude Code** for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs directly on your machine with zero rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Add the Server in Claude Code

Register at user scope via the `claude` CLI:

```bash
claude mcp add --scope user cvss-calculator -- cvss-calculator-mcp
```

Or add it to your project `.mcp.json` (or user `~/.claude.json`):

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "type": "stdio",
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to the hosted endpoint at `https://cvss.my.id/mcp` without installing a binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

### Register via Claude CLI

```bash
claude mcp add --scope user --transport http cvss-calculator https://cvss.my.id/mcp
```

Or in `.mcp.json` / `~/.claude.json`:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "type": "http",
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

## Verify the Connection

Check registered MCP servers:

```bash
claude mcp list
```

Inside an interactive Claude Code session, run `/mcp` to inspect server status, or prompt:

```text
Calculate the CVSS base score for CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H using calculate_cvss.
```

Expected Base Score result: `10.0 (CRITICAL)`.

## Troubleshooting

- **Server fails to spawn in Local mode**: Run `which cvss-calculator-mcp` in your terminal and provide the absolute binary path in `"command"` if your GUI/IDE environment does not inherit your shell `PATH`.
- **Rate limit errors (`429`) in Remote mode**: The public endpoint allows `5 requests per minute per IP`. Switch to Option 1 (Local `stdio`) for unrestricted usage.

## Official Reference

- [Claude Code MCP Documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)
````````

## A.14 — `mcp/clients/claude-desktop.md`

**Destination:** `content/docs/mcp/clients/claude-desktop.md`  
**SHA-256 (UTF-8 original):** `52ab67e201372900e8160cc9529b7bf9361f3b5fb7eff5e1d6cdf210b4f5461d`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Claude Desktop MCP Setup
description: Connect the CVSS MCP server to Claude Desktop.
---

## Local MCP (`stdio`) - Recommended

On macOS, install the binary:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

Open Claude Desktop settings, find the developer or local MCP server settings, and edit the configuration file. Merge this entry into `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "command": "/absolute/path/to/cvss-calculator-mcp",
      "args": []
    }
  }
}
```

Replace the command with the path printed by `which cvss-calculator-mcp`, then restart Claude Desktop. The installer command above supports Linux and macOS; for other platforms, see the [local installation guide](/docs/mcp/local/).

## Remote MCP (Streamable HTTP)

In Claude settings, add a custom connector with the URL `https://cvss.my.id/mcp`. Availability may depend on account and organization settings. Remote connectors connect from Anthropic's cloud infrastructure; vectors sent through them leave your device.

**Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).** The relevant IP for a remote connector is the client service's egress IP, which may be shared.

## Verify

In a conversation, open the connector picker and check that the CVSS server and `calculate_cvss` tool are available. Ask Claude to score `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`; the expected Base Score is **5.4 (Medium)**.

## Troubleshooting

- If the local server fails to start, check its logs in Claude Desktop settings, the absolute binary path, and file permissions.
- If the remote connector is unavailable, check account and organization connector settings and the endpoint's reachability.

## Official Reference

- [Claude Desktop local MCP servers](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)
- [Claude custom remote connectors](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
````````

## A.15 — `mcp/clients/cline.md`

**Destination:** `content/docs/mcp/clients/cline.md`  
**SHA-256 (UTF-8 original):** `2ec7b2665e081e551998906b31d53b29a63fd6f106a90f7cbbb6f87cebf12654`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Cline MCP Setup
description: Configure cvss-calculator-mcp in Cline using Local stdio or Remote Streamable HTTP.
---

This guide shows how to connect **Cline** (VS Code extension and Cline CLI) to `cvss-calculator-mcp` for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs directly on your machine with no rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Add to `cline_mcp_settings.json`

In Cline, click **MCP Servers** in the top navigation bar, select the **Configure** tab, and click **Configure MCP Servers** to open `cline_mcp_settings.json` (or edit `~/.cline/data/settings/cline_mcp_settings.json` directly):

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "command": "cvss-calculator-mcp",
      "args": [],
      "disabled": false
    }
  }
}
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to `https://cvss.my.id/mcp` without installing a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

Add the remote server to `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "url": "https://cvss.my.id/mcp",
      "type": "streamableHttp",
      "disabled": false
    }
  }
}
```

## Verify the Connection

1. Check the **MCP Servers** panel in Cline to confirm `cvss-calculator` shows a green status dot and lists the `calculate_cvss` tool.
2. Prompt Cline:

```text
Calculate the CVSS base score for AV:N/AC:L/Au:N/C:P/I:P/A:P using calculate_cvss.
```

Expected Base Score result: `7.5 (HIGH)`.

## Troubleshooting

- **Server fails to start in VS Code**: Replace `"command": "cvss-calculator-mcp"` with the full path from `which cvss-calculator-mcp` (e.g. `"/usr/local/bin/cvss-calculator-mcp"`).
- **Rate limit (`HTTP 429`) on Remote MCP**: The public endpoint allows `5 requests per minute per IP`. Switch to Option 1 (Local `stdio`).

## Official Reference

- [Cline MCP Overview Documentation](https://docs.cline.bot/mcp/mcp-overview)
````````

## A.16 — `mcp/clients/codex.md`

**Destination:** `content/docs/mcp/clients/codex.md`  
**SHA-256 (UTF-8 original):** `f8330bad31536cd87955b2bbe4e5b54f4368e69e032ff55e124f690641377e71`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Codex MCP Setup
description: Configure cvss-calculator-mcp in OpenAI Codex CLI and IDE extension using Local stdio or Remote Streamable HTTP.
---

This guide shows how to connect **OpenAI Codex** to `cvss-calculator-mcp` for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs on your machine with no rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Register with Codex CLI

Run the `codex mcp add` command:

```bash
codex mcp add cvss-calculator -- cvss-calculator-mcp
```

Or add the following block directly to `~/.codex/config.toml` (global) or `.codex/config.toml` (project):

```toml
[mcp_servers.cvss-calculator]
command = "cvss-calculator-mcp"
args = []
```

## Option 2: Remote MCP (Streamable HTTP)

Use the hosted endpoint at `https://cvss.my.id/mcp` when you do not want to install a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

### Register via Codex CLI

```bash
codex mcp add cvss-calculator --url https://cvss.my.id/mcp
```

Or add the following block to `~/.codex/config.toml`:

```toml
[mcp_servers.cvss-calculator]
url = "https://cvss.my.id/mcp"
```

## Verify the Connection

List configured MCP servers in Codex:

```bash
codex mcp list
```

Then ask Codex in a session:

```text
Use calculate_cvss to score CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N
```

Expected Base Score result: `9.3 (CRITICAL)`.

## Troubleshooting

- **`cvss-calculator-mcp: command not found`**: Check `which cvss-calculator-mcp`. If installed in `~/.local/bin`, either add `~/.local/bin` to your `PATH` or specify the full path (e.g. `command = "/home/youruser/.local/bin/cvss-calculator-mcp"`) in `~/.codex/config.toml`.
- **`HTTP 429 Too Many Requests` on Remote MCP**: You exceeded `5 requests per minute per IP` on the public endpoint. Switch to Option 1 (Local `stdio`).

## Official Reference

- [OpenAI Codex Model Context Protocol Documentation](https://developers.openai.com/codex/mcp)
````````

## A.17 — `mcp/clients/cursor.md`

**Destination:** `content/docs/mcp/clients/cursor.md`  
**SHA-256 (UTF-8 original):** `4f84f3b54587223a9e3cd5bcca67e20f97edaf8f90a39e222c0e24ad0133e793`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Cursor MCP Setup
description: Configure cvss-calculator-mcp in Cursor using Local stdio or Remote Streamable HTTP.
---

This guide shows how to connect **Cursor** to `cvss-calculator-mcp` for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` executes on your workstation with zero rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Add to `.cursor/mcp.json`

Create or edit `.cursor/mcp.json` in your project root (or `~/.cursor/mcp.json` for global configuration):

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to `https://cvss.my.id/mcp` over Streamable HTTP without installing a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

Add the remote server to `.cursor/mcp.json` or `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

## Verify the Connection

1. Open **Cursor Settings > MCP** and verify that `cvss-calculator` shows a green active indicator and lists the `calculate_cvss` tool.
2. In Cursor Composer / Agent chat, ask:

```text
Use calculate_cvss to score CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N
```

Expected Base Score result: `9.3 (CRITICAL)`.

## Troubleshooting

- **Red status indicator in Cursor Settings > MCP**: Cursor GUI applications on macOS/Linux often do not load `~/.bashrc` or `~/.zshrc` paths. Run `which cvss-calculator-mcp` in your terminal and set `"command"` to the full path (e.g. `"/usr/local/bin/cvss-calculator-mcp"`).
- **Rate limit (`HTTP 429`) on Remote MCP**: The public endpoint allows `5 requests per minute per IP`. Switch to Option 1 (Local `stdio`).

## Official Reference

- [Cursor Model Context Protocol (MCP) Documentation](https://docs.cursor.com/context/model-context-protocol)
````````

## A.18 — `mcp/clients/gemini-cli.md`

**Destination:** `content/docs/mcp/clients/gemini-cli.md`  
**SHA-256 (UTF-8 original):** `0ed50ff20d901f5d9c807b64f010c74e602f7e139f7ad83be290efa3e0155b8d`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Gemini CLI MCP Setup
description: Connect the CVSS MCP server to Gemini CLI.
---

## Local MCP (`stdio`) - Recommended

Install the binary on Linux or macOS:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

Merge this entry into `~/.gemini/settings.json` (user) or `.gemini/settings.json` (project):

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Remote MCP (Streamable HTTP)

Replace the local entry with:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "httpUrl": "https://cvss.my.id/mcp"
    }
  }
}
```

**Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

## Verify

Start Gemini CLI and run `/mcp` to check the server connection and available tools. Ask it to calculate the Base Score for `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`; the expected result is **5.4 (Medium)**.

## Troubleshooting

- If the command is not found, use the absolute path from `which cvss-calculator-mcp`.
- For an HTTP 429 response, wait before retrying or use the local server.

## Official Reference

- [Gemini CLI MCP server configuration](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/mcp-server.md)
````````

## A.19 — `mcp/clients/github-copilot.md`

**Destination:** `content/docs/mcp/clients/github-copilot.md`  
**SHA-256 (UTF-8 original):** `cd808d9488f07f4bae016e61aa5157f7ec75d2bf30a3a483e5275eb902132e72`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: GitHub Copilot MCP Setup
description: Configure cvss-calculator-mcp in GitHub Copilot Agent Mode (VS Code and Copilot CLI) using Local stdio or Remote Streamable HTTP.
---

This guide shows how to configure `cvss-calculator-mcp` in **GitHub Copilot** (VS Code Agent Mode and GitHub Copilot Coding Agent / CLI) for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs directly on your machine with no rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Add to `.vscode/mcp.json` (VS Code Agent Mode)

Create or update `.vscode/mcp.json` in your workspace (note that VS Code uses the top-level `"servers"` key):

```json
{
  "servers": {
    "cvss-calculator": {
      "type": "stdio",
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

If you are using the standalone **GitHub Copilot CLI** (`~/.copilot/mcp-config.json`), use the `"mcpServers"` key:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "type": "local",
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to `https://cvss.my.id/mcp` without installing a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

In `.vscode/mcp.json`:

```json
{
  "servers": {
    "cvss-calculator": {
      "type": "http",
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

## Verify the Connection

1. Open VS Code Chat, switch to **Agent** mode, and click the **Tools** icon to confirm `calculate_cvss` is listed and enabled.
2. Prompt Copilot Agent Mode:

```text
Calculate the CVSS base score for CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N
```

Expected Base Score result: `5.4 (MEDIUM)`.

## Troubleshooting

- **Server fails to start in VS Code**: Click **Start** code lens above `"cvss-calculator"` in `.vscode/mcp.json` and check the Output panel (`MCP: List Servers`). Use the full path from `which cvss-calculator-mcp` if VS Code does not see `~/.local/bin`.
- **`HTTP 429` in Remote mode**: The free endpoint is limited to `5 requests per minute per IP`. Switch to Option 1 (`"type": "stdio"`).

## Official Reference

- [VS Code & GitHub Copilot MCP Configuration Reference](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
````````

## A.20 — `mcp/clients/hermes-agent.md`

**Destination:** `content/docs/mcp/clients/hermes-agent.md`  
**SHA-256 (UTF-8 original):** `b6cfc11e29dd4683515b8fad51e93ac7e2a135fd1ca2841b150c61b6f7b7de77`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Hermes Agent MCP Setup
description: Configure cvss-calculator-mcp in Nous Research Hermes Agent using Local stdio or Remote Streamable HTTP.
---

This guide shows how to connect **Hermes Agent** to `cvss-calculator-mcp` for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs directly on your machine with no rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Add to `~/.hermes/config.yaml`

Run `hermes mcp add` or add `cvss_calculator` under the `mcp_servers:` key in `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  cvss_calculator:
    command: "cvss-calculator-mcp"
    args: []
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to `https://cvss.my.id/mcp` over Streamable HTTP without installing a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

Add the remote server to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  cvss_calculator:
    url: "https://cvss.my.id/mcp"
```

## Verify the Connection

Inside a Hermes Agent session (or via CLI), check active MCP servers:

```bash
hermes mcp list
```

Or run `/mcp` inside the interactive session, then ask Hermes Agent:

```text
Use calculate_cvss to compute the base score for CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N
```

Expected Base Score result: `9.3 (CRITICAL)`.

## Troubleshooting

- **Binary not found when starting Hermes Agent**: Run `which cvss-calculator-mcp` and set `command:` to the absolute path (for example, `command: "/usr/local/bin/cvss-calculator-mcp"`).
- **Rate limit (`HTTP 429`) on Remote MCP**: The public endpoint allows `5 requests per minute per IP`. Use Option 1 (Local `stdio`) for higher throughput.

## Official Reference

- [Hermes Agent MCP Documentation](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp)
````````

## A.21 — `mcp/clients/index.md`

**Destination:** `content/docs/mcp/clients/index.md`  
**SHA-256 (UTF-8 original):** `ebe55552a7e9bc05909c3af9c5630faccd9748d58b753a14136700b72a46d875`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: MCP Client Setup Guides
description: Setup guides for selected MCP clients and editors.
---

These guides cover a selection of AI agents and editors; other MCP clients may also work if they support stdio or Streamable HTTP. Follow your client’s documentation for its configuration format. Every guide covers **Local `stdio` setup first (recommended, no rate limit)** followed by **Remote Streamable HTTP**.

## Selected Client Guides

1. [**Codex**](/docs/mcp/clients/codex/): Configure via `codex mcp add` or `~/.codex/config.toml`.
2. [**Claude Code**](/docs/mcp/clients/claude-code/): Configure via `claude mcp add` or `.mcp.json` / `~/.claude.json`.
3. [**OpenCode v2**](/docs/mcp/clients/opencode/): Configure via `opencode mcp add` or `opencode.jsonc`.
4. [**GitHub Copilot**](/docs/mcp/clients/github-copilot/): Configure in VS Code Agent Mode (`.vscode/mcp.json`) or Copilot CLI (`~/.copilot/mcp-config.json`).
5. [**Cursor**](/docs/mcp/clients/cursor/): Configure via `.cursor/mcp.json` or `~/.cursor/mcp.json`.
6. [**VS Code**](/docs/mcp/clients/vscode/): Configure the workspace MCP servers in `.vscode/mcp.json`.
7. [**Zed**](/docs/mcp/clients/zed/): Configure `context_servers` in Zed settings.
8. [**Kilo Code**](/docs/mcp/clients/kilo-code/): Configure via `~/.config/kilo/kilo.jsonc` or `.kilo/kilo.jsonc`.
9. [**Cline**](/docs/mcp/clients/cline/): Configure via `cline_mcp_settings.json` in VS Code or Cline CLI.
10. [**Hermes Agent**](/docs/mcp/clients/hermes-agent/): Configure under `mcp_servers` in `~/.hermes/config.yaml`.
11. [**Gemini CLI**](/docs/mcp/clients/gemini-cli/): Configure `mcpServers` in Gemini settings.
12. [**JetBrains**](/docs/mcp/clients/jetbrains/): Add a server through the MCP settings.
13. [**Claude Desktop**](/docs/mcp/clients/claude-desktop/): Configure a local server or a remote connector.

## Prerequisites for Local `stdio`

Before using the **Local `stdio`** configuration in any client below, install `cvss-calculator-mcp` on Linux or macOS:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

Verify the binary is available on your `PATH`:

```bash
which cvss-calculator-mcp
```
````````

## A.22 — `mcp/clients/jetbrains.md`

**Destination:** `content/docs/mcp/clients/jetbrains.md`  
**SHA-256 (UTF-8 original):** `295e44395f6215f8805206b90492fb46d0f74cb6fb91f8e560cd587faa531635`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: JetBrains MCP Setup
description: Connect the CVSS MCP server to JetBrains AI Assistant.
---

These instructions configure **JetBrains AI Assistant as an MCP client**. They are separate from the MCP server built into JetBrains IDEs.

## Local MCP (`stdio`) - Recommended

Install the binary on Linux or macOS:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

Open **Settings → Tools → AI Assistant → Model Context Protocol (MCP)**, add a server, select STDIO, and provide:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Remote MCP (Streamable HTTP)

Add a server using HTTP and provide:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

**Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

## Verify

Check the server status in the MCP settings. In AI Chat, ask the agent to use `calculate_cvss` for `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`; the expected Base Score is **5.4 (Medium)**. Agent availability may depend on your IDE and configuration.

## Troubleshooting

- If STDIO fails to start, use the absolute path from `which cvss-calculator-mcp` and check the server status.
- If adding servers is unavailable, check whether your organization manages MCP settings centrally.

## Official Reference

- [JetBrains AI Assistant MCP documentation](https://www.jetbrains.com/help/ai-assistant/mcp.html)
````````

## A.23 — `mcp/clients/kilo-code.md`

**Destination:** `content/docs/mcp/clients/kilo-code.md`  
**SHA-256 (UTF-8 original):** `abfc89b055d0734a9e7642d60809a5bc9b0637ff8803e4f55442ad8391e2cf37`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Kilo Code MCP Setup
description: Configure cvss-calculator-mcp in Kilo Code using Local stdio or Remote Streamable HTTP.
---

This guide shows how to configure `cvss-calculator-mcp` in **Kilo Code** (CLI and VS Code extension) for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs directly on your machine with zero rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Configure Kilo Code

For **Kilo Code CLI / v1.0+**, add `cvss-calculator` under `"mcp"` in `~/.config/kilo/kilo.jsonc` (global) or `kilo.jsonc` (project):

```json
{
  "$schema": "https://kilo.ai/config.json",
  "mcp": {
    "cvss-calculator": {
      "type": "local",
      "command": ["cvss-calculator-mcp"],
      "enabled": true
    }
  }
}
```

For the **Kilo Code VS Code Extension** (`.kilocode/mcp.json` or Global `mcp_settings.json`), add under `"mcpServers"`:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to `https://cvss.my.id/mcp` without installing a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

In `kilo.jsonc` (`~/.config/kilo/kilo.jsonc`):

```json
{
  "$schema": "https://kilo.ai/config.json",
  "mcp": {
    "cvss-calculator": {
      "type": "remote",
      "url": "https://cvss.my.id/mcp",
      "enabled": true
    }
  }
}
```

Or in `.kilocode/mcp.json`:

```json
{
  "mcpServers": {
    "cvss-calculator": {
      "type": "streamable-http",
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

## Verify the Connection

1. Open Kilo Code **MCP Servers** settings and confirm `cvss-calculator` is connected and exposes `calculate_cvss`.
2. Ask Kilo Code:

```text
Use calculate_cvss to score CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H
```

Expected Base Score result: `10.0 (CRITICAL)`.

## Troubleshooting

- **Command not found**: Replace `"cvss-calculator-mcp"` with the full path from `which cvss-calculator-mcp`.
- **`HTTP 429` on Remote MCP**: The public endpoint allows `5 requests per minute per IP`. Use Option 1 (Local `stdio`).

## Official Reference

- [Using MCP in Kilo Code](https://kilo.ai/docs/automate/mcp/using-in-kilo-code)
````````

## A.24 — `mcp/clients/opencode.md`

**Destination:** `content/docs/mcp/clients/opencode.md`  
**SHA-256 (UTF-8 original):** `1bb1d2ec9ac274d5b4f670701f66e0a10f551c1dd7cd6c9843148bca82f76745`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: OpenCode v2 MCP Setup
description: Configure cvss-calculator-mcp in OpenCode v2 using Local stdio or Remote Streamable HTTP.
---

This guide shows how to configure `cvss-calculator-mcp` in **OpenCode v2** for CVSS 2.0, 3.0, 3.1, and 4.0 Base Score calculations.

## Option 1: Local MCP (`stdio`) - Recommended

Local `stdio` runs directly on your machine with no rate limits.

### Step 1: Install the Binary (Linux / macOS)

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

### Step 2: Configure OpenCode v2

OpenCode v2 places server entries under `mcp.servers`, not directly under `mcp`. Merge the following into `~/.config/opencode/opencode.json` (global) or `opencode.json` in your project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "cvss-calculator": {
        "type": "local",
        "command": ["cvss-calculator-mcp"]
      }
    }
  }
}
```

## Option 2: Remote MCP (Streamable HTTP)

Connect to `https://cvss.my.id/mcp` over Streamable HTTP without installing a local binary.

> **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

Replace the local server entry with the remote configuration in `~/.config/opencode/opencode.json` or `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "cvss-calculator": {
        "type": "remote",
        "url": "https://cvss.my.id/mcp"
      }
    }
  }
}
```

## Verify the Connection

Check registered MCP tools in OpenCode:

```bash
opencode mcp list
```

Then ask OpenCode in a session:

```text
Use calculate_cvss to compute the score for CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N
```

Expected Base Score result: `9.3 (CRITICAL)`.

## Troubleshooting

- **Binary not found (`ENOENT`)**: Replace `["cvss-calculator-mcp"]` with the absolute path from `which cvss-calculator-mcp` (for example, `["/usr/local/bin/cvss-calculator-mcp"]`).
- **Rate limit (`HTTP 429`) on Remote MCP**: The public endpoint allows `5 requests per minute per IP`. Use Option 1 (`"type": "local"`) for unlimited scoring.

## Official Reference

- [OpenCode v2 MCP Servers Documentation](https://github.com/anomalyco/opencode/blob/v2.1.2/services/www/src/docs/content/mcp-servers.mdx)
````````

## A.25 — `mcp/clients/vscode.md`

**Destination:** `content/docs/mcp/clients/vscode.md`  
**SHA-256 (UTF-8 original):** `89df1dc946d9b314c1ba2c25e7a0a967f5a9624e55d2dff17f5efb0e9a9325d3`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: VS Code MCP Setup
description: Configure the CVSS MCP server in VS Code with local stdio or remote HTTP.
---

Use VS Code's MCP configuration with a supported agent, such as GitHub Copilot. These examples configure one workspace; merge the entry into an existing `.vscode/mcp.json` if present.

## Local MCP (`stdio`) - Recommended

Install the binary on Linux or macOS:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

Add the server to `.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "cvss-calculator": {
      "type": "stdio",
      "command": "cvss-calculator-mcp",
      "args": []
    }
  }
}
```

## Remote MCP (Streamable HTTP)

To use the public endpoint instead, replace the local entry with:

```json
{
  "servers": {
    "cvss-calculator": {
      "type": "http",
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

**Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

## Verify

Start the server from `.vscode/mcp.json` or use **MCP: List Servers** in the Command Palette. In Chat, select an agent that supports MCP and check that its tools include `calculate_cvss`. Ask it to score `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`; the expected Base Score is **5.4 (Medium)**.

## Troubleshooting

- If the local server does not start, check the server output and use the absolute path returned by `which cvss-calculator-mcp`. For remote workspaces, the binary must be available in the environment where the server runs.
- If remote requests receive HTTP 429, use Local MCP or wait before retrying.

## Official Reference

- [VS Code MCP configuration reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration)
- [Manage MCP servers in VS Code](https://code.visualstudio.com/docs/agents/run/manage-mcp-servers)
````````

## A.26 — `mcp/clients/zed.md`

**Destination:** `content/docs/mcp/clients/zed.md`  
**SHA-256 (UTF-8 original):** `74d54aecd3c8707c85da6d6729ac069b9c3f46e6aa3ec40f5373e27e868ec2cc`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Zed MCP Setup
description: Configure the CVSS MCP server in Zed using local stdio or remote HTTP.
---

These instructions configure a custom MCP server for the **Zed Agent**. External agents or terminal-based agents may use their own MCP configuration; see [Zed's MCP documentation](https://zed.dev/docs/ai/mcp).

## Local MCP (`stdio`) - Recommended

Install the binary on Linux or macOS:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

In Zed, open **Settings → AI → MCP Servers**, choose **Add Server → Add Local Server**, and set the command to `cvss-calculator-mcp`. Alternatively, merge this entry into the settings file opened with **zed: open settings file**:

```json
{
  "context_servers": {
    "cvss-calculator": {
      "command": "cvss-calculator-mcp",
      "args": [],
      "env": {}
    }
  }
}
```

## Remote MCP (Streamable HTTP)

Choose **Add Server → Add Remote Server**, or replace the local entry in settings with:

```json
{
  "context_servers": {
    "cvss-calculator": {
      "url": "https://cvss.my.id/mcp"
    }
  }
}
```

**Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).** Zed's documentation notes that a remote server without an Authorization header can trigger its MCP OAuth flow; check the server status if prompted.

## Verify

Check the MCP server status in **Settings → AI → MCP Servers**. In a Zed Agent thread, check that the server's tools are available and ask it to score `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`. The expected Base Score is **5.4 (Medium)**.

## Troubleshooting

- If the local server fails to start, replace the command with the absolute path from `which cvss-calculator-mcp`; ensure the binary is installed in the environment running the server.
- If the remote server returns HTTP 429, use Local MCP or wait before retrying.

## Official Reference

- [Zed Model Context Protocol documentation](https://zed.dev/docs/ai/mcp)
````````

## A.27 — `mcp/index.md`

**Destination:** `content/docs/mcp/index.md`  
**SHA-256 (UTF-8 original):** `63970ed0bdf27477c976464bb57b69511292fda2c7bc8d2455dfd673640d2df8`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: MCP Overview
description: Run the CVSS Calculator MCP server locally over stdio or connect to the free public Streamable HTTP endpoint.
---

**`cvss-calculator-mcp`** is a fast, lightweight, open-source Model Context Protocol (MCP) server written in Go ([`alfa-reza/cvss-calculator-mcp`](https://github.com/alfa-reza/cvss-calculator-mcp)). It enables AI coding agents and security tools to validate CVSS vectors and compute **Base Scores** across **CVSS v4.0, v3.1, v3.0, and v2.0**.

> [!IMPORTANT]
> **Scope Comparison: Base-Only MCP vs Full Browser Calculator**
> The Go MCP server (`cvss-calculator-mcp`) computes **Base Scores only** via the `calculate_cvss` tool. When you need **Threat, Temporal, Environmental, or Supplemental** metric group scoring, use the [Full Browser Calculator](https://cvss.my.id/calculator/).

## Set Up with an AI Agent

Copy this prompt to your AI agent:

```text
Read https://get.cvss.my.id/LLM.txt and follow its instructions to set up CVSS Calculator MCP and offer to install its vulnerability assessment skill.
```

The agent will ask you to choose and approve local or remote MCP before making changes. It will ask for separate approval before installing the optional skill.

## Two Ways to Connect

### 1. Local MCP (`stdio`) - Recommended

Install the standalone single binary onto your workstation and let your AI client launch `cvss-calculator-mcp` directly over standard input/output (`stdio`).

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

- **Supported OS**: Linux, macOS, and Windows (`amd64` and `arm64`). See the local setup guide for the Windows installer.
- **Transport**: `stdio`
- **Rate limit**: None (100% local execution, works offline).
- **Guide**: [Local MCP Setup (stdio)](/docs/mcp/local/)

### 2. Remote MCP (HTTP) - Free Public Endpoint

Connect any MCP client that supports **Streamable HTTP** directly to the hosted endpoint without installing a local binary:

```text
https://cvss.my.id/mcp
```

- **Transport**: Streamable HTTP
- **Rate limit**: **5 requests per minute per IP. For higher usage, run Local MCP (stdio).**
- **Note**: The endpoint `https://cvss.my.id/mcp` is a protocol endpoint for MCP clients, not a human HTML page. For the human overview page, visit [MCP Tools Overview](/tools/mcp).
- **Guide**: [Remote MCP Setup (HTTP)](/docs/mcp/remote/)

## Vulnerability Assessment Skill

The optional skill guides an agent through assessing vulnerability evidence, selecting CVSS metrics, and calculating a Base Score using the connected MCP server. Install it with the [Skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add alfa-reza/cvss-calculator-mcp --skill vulnerability-cvss-assessment
```

Choose your agent when prompted. Add `--global` to make the skill available across projects. Connect the MCP server using one of the options above. Review the [skill instructions](https://github.com/alfa-reza/cvss-calculator-mcp/blob/main/skills/vulnerability-cvss-assessment/SKILL.md) and [tool reference](https://github.com/alfa-reza/cvss-calculator-mcp/blob/main/skills/vulnerability-cvss-assessment/references/tool-reference.md).

## Client Configuration Guides

Choose your AI agent or editor for copy-paste configuration:

- [Codex](/docs/mcp/clients/codex/)
- [Claude Code](/docs/mcp/clients/claude-code/)
- [OpenCode v2](/docs/mcp/clients/opencode/)
- [GitHub Copilot](/docs/mcp/clients/github-copilot/)
- [Cursor](/docs/mcp/clients/cursor/)
- [VS Code](/docs/mcp/clients/vscode/)
- [Zed](/docs/mcp/clients/zed/)
- [Gemini CLI](/docs/mcp/clients/gemini-cli/)
- [JetBrains](/docs/mcp/clients/jetbrains/)
- [Claude Desktop](/docs/mcp/clients/claude-desktop/)
- [Kilo Code](/docs/mcp/clients/kilo-code/)
- [Cline](/docs/mcp/clients/cline/)
- [Hermes Agent](/docs/mcp/clients/hermes-agent/)
````````

## A.28 — `mcp/local.md`

**Destination:** `content/docs/mcp/local.md`  
**SHA-256 (UTF-8 original):** `ebd3894301352d4a2b350dd528e747230d25f3c245f807d9df2775996509cd13`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Local MCP Setup (stdio)
description: Install and run cvss-calculator-mcp locally over stdio on Linux, macOS, and Windows with zero rate limits.
---

Running `cvss-calculator-mcp` locally over `stdio` is the **recommended** setup for daily security engineering, automated code review, and batch vulnerability triage. All calculations execute on your machine with zero network latency and no public endpoint rate limits.

## 1. Install with the Official Installer (Linux & macOS)

Run the installer script:

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```

The installer detects your operating system (`linux` or `darwin`) and CPU architecture (`amd64` or `arm64`), downloads the matching release archive from [`alfa-reza/cvss-calculator-mcp`](https://github.com/alfa-reza/cvss-calculator-mcp/releases), and installs `cvss-calculator-mcp` into `/usr/local/bin` (or `~/.local/bin` if `/usr/local/bin` is not writable without `sudo`).

### Windows (PowerShell, x64 and ARM64)

Run in PowerShell:

```powershell
irm https://raw.githubusercontent.com/alfa-reza/cvss-calculator-mcp/main/install.ps1 | iex
```

The installer verifies the release checksum and installs to `%LOCALAPPDATA%\Programs\cvss-calculator-mcp` by default. Add that directory to your PATH, or use the full binary path in your MCP client.

## 2. Alternative Installation Methods

### Install from Source with Go

If you have Go 1.25+ installed:

```bash
go install github.com/alfa-reza/cvss-calculator-mcp/cmd/cvss-calculator-mcp@latest
```

Ensure `$(go env GOPATH)/bin` is in your shell `PATH`.

### Manual Release Binary Download

Download the precompiled binary archive directly from the [GitHub Releases page](https://github.com/alfa-reza/cvss-calculator-mcp/releases), extract `cvss-calculator-mcp`, mark it executable (`chmod +x cvss-calculator-mcp`), and move it into a directory on your `PATH`.

## 3. Verify Installation

Confirm the binary responds on your `PATH`:

```bash
which cvss-calculator-mcp
```

Test a JSON-RPC `initialize` request over `stdio`:

```bash
printf '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"smoke-test","version":"1.0.0"}}}\n' | cvss-calculator-mcp
```

You should receive a single-line JSON-RPC response containing `serverInfo` and `capabilities.tools`.

## 4. Connect Your MCP Client

Proceed to the [Client Setup Guides](/docs/mcp/clients/) for your specific agent or IDE.
````````

## A.29 — `mcp/remote.md`

**Destination:** `content/docs/mcp/remote.md`  
**SHA-256 (UTF-8 original):** `e118049ec8459338c5147f9c7635ca7e4e6dc0db117c7cd7230cbd95eea7f5ba`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Remote MCP Setup (HTTP)
description: Connect MCP clients to the remote endpoint over Streamable HTTP.
---

If you are working in a browser-based environment, ephemeral container, or want to test `cvss-calculator-mcp` without installing a local binary, connect your MCP client to our free public endpoint.

## Endpoint Details

- **Endpoint URL**: `https://cvss.my.id/mcp`
- **Transport Protocol**: MCP **Streamable HTTP** (`POST` / `GET` JSON-RPC over HTTPS)
- **Authentication**: None required
- **Supported Scoring**: CVSS v4.0, v3.1, v3.0, and v2.0 **Base Score** (`calculate_cvss`)
- **Published Usage Policy**: **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**

> [!NOTE]
> Opening `https://cvss.my.id/mcp` directly in a web browser address bar will not render a website page because it is an automated MCP protocol endpoint. For human documentation and configuration instructions, visit the [MCP Tools Overview](/tools/mcp) or [MCP Documentation](/docs/mcp/).

## Testing the Remote Endpoint with `curl`

You can verify that the remote MCP endpoint is reachable from your network by sending an MCP `initialize` handshake:

```bash
curl -i -X POST https://cvss.my.id/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "curl-check",
        "version": "1.0.0"
      }
    }
  }'
```

## Avoiding Rate Limits (`HTTP 429`)

The public endpoint enforces a fair-use policy of **5 requests per minute per IP** to keep the hosted service available at zero cost. If your agent performs batch vulnerability reviews or automated CI runs, switch to [Local MCP (`stdio`)](/docs/mcp/local/):

```bash
curl -fsSL https://get.cvss.my.id/install.sh | bash
```
````````

## A.30 — `mcp/security.md`

**Destination:** `content/docs/mcp/security.md`  
**SHA-256 (UTF-8 original):** `eb1b930ccc72b24597df498243091041e8e9e6ec3348bc8ee7da7e8112bdfe0e`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: Security, Privacy & Rate Limits
description: Privacy model, rate limits, offline guarantees, and source verification for CVSS.my.id and cvss-calculator-mcp.
---

Security engineers working on unreleased vulnerabilities or internal audit findings need clear visibility into where their data goes.

## 1. Full Browser Calculator

- **100% Local Execution**: Every CVSS 2.0, 3.0, 3.1, and 4.0 calculation runs in your browser tab using static JavaScript (`@cvss/core`).
- **Zero Scoring Network Calls**: Editing metrics, pasting vectors, or copying results never sends your CVSS vector to `cvss.my.id/mcp` or any external server.
- **No Analytics or Telemetry**: Neither the website nor the browser calculator embeds third-party analytics, tracking pixels, session replay scripts, or CDN fonts.
- **Ephemeral Storage**: Draft vectors and selected metrics are stored only in browser session storage so refreshing the page preserves in-progress work. Closing the browser tab clears session data.

## 2. Local MCP Server (`cvss-calculator-mcp` via `stdio`)

- **Offline Operation**: Once installed, `cvss-calculator-mcp` communicates exclusively with your local MCP host over `stdio` pipes. It makes zero outbound network connections.
- **Auditable Source**: Inspect the complete Go implementation and release workflow at [`github.com/alfa-reza/cvss-calculator-mcp`](https://github.com/alfa-reza/cvss-calculator-mcp).
- **Rate Limits**: None.

## 3. Remote MCP Endpoint (Streamable HTTP)

- **Published Fair-Use Policy**: **Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).**
- **Transport**: HTTPS (`Streamable HTTP`) terminated on Cloudflare Workers.
- **Confidential Advisory Recommendation**: While `calculate_cvss` only receives bare CVSS vector strings (such as `CVSS:4.0/AV:N/...`) without vulnerability titles or asset names, teams working under strict air-gapped or zero-external-traffic policies should use **Local MCP (`stdio`)**.
````````

## A.31 — `mcp/tools.md`

**Destination:** `content/docs/mcp/tools.md`  
**SHA-256 (UTF-8 original):** `6dcbd6a49fccd31dc33030b052a3adaf8971766dc1b1cd2c0e8bb8d9c9d8fe6d`  
**Content begins exactly after the opening fence:**

````````markdown
---
title: MCP Tool Reference
description: Input parameters, response format, and example calls for the calculate_cvss MCP tool.
---

Both the Local (`stdio`) and Remote (Streamable HTTP) servers expose the **`calculate_cvss`** tool for computing **Base Scores** across CVSS v4.0, v3.1, v3.0, and v2.0.

## Tool Name: `calculate_cvss`

Calculates the CVSS Base Score and qualitative severity rating for one vector. Specify its CVSS version explicitly.

### Input Schema

`calculate_cvss` requires two fields: the CVSS specification `version` and a complete, case-sensitive **Base-only** `vector` string:

```json
{
  "type": "object",
  "properties": {
    "version": {
      "type": "string",
      "enum": ["2.0", "3.0", "3.1", "4.0"],
      "description": "CVSS specification version (2.0, 3.0, 3.1, or 4.0)."
    },
    "vector": {
      "type": "string",
      "description": "A complete, case-sensitive CVSS Base vector string."
    }
  },
  "required": ["version", "vector"]
}
```

### Example Call

**Tool Arguments:**

```json
{
  "version": "3.1",
  "vector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H"
}
```

**Tool Response:**

```json
{
  "version": "3.1",
  "vector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
  "score": 9.8,
  "severity": "Critical"
}
```

### Example Across CVSS Versions

#### CVSS v4.0

**Tool Arguments:**

```json
{
  "version": "4.0",
  "vector": "CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N"
}
```

**Tool Response:**

```json
{
  "version": "4.0",
  "vector": "CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N",
  "score": 9.3,
  "severity": "Critical"
}
```

#### CVSS v2.0

**Tool Arguments:**

```json
{
  "version": "2.0",
  "vector": "AV:N/AC:L/Au:N/C:P/I:P/A:P"
}
```

**Tool Response:**

```json
{
  "version": "2.0",
  "vector": "AV:N/AC:L/Au:N/C:P/I:P/A:P",
  "score": 7.5,
  "severity": "High"
}
```

## Important Scope Notes

- **Base Score Only**: `calculate_cvss` evaluates the Base metric group. If you need Threat (`E`), Temporal (`E, RL, RC`), Environmental (`CR, IR, AR`, Modified Base), or Supplemental metrics, use the [Full Browser Calculator](https://cvss.my.id/calculator/).
- **No CVE Database Lookup**: `calculate_cvss` computes mathematical scores from CVSS vector strings. It does not query NVD or external CVE feeds.
````````

---

# APPENDIX B — COMPLETE ORIGINAL ENGLISH USER-FACING TEXT DICTIONARY

The exact English copy for homepage, MCP workspace, navigation, labels, status indicators and footer was defined in the source at `apps/website/src/i18n/ui.ts` under `ui.en`. The following is **ONLY the full English dictionary**, excluding the old Indonesian dictionary entirely. Apply it to the new Next pages, reorganizing layout while preserving factual meaning, all commands, metadata, and useful links. It is NOT necessary to recreate the bilingual i18n system.

````````typescript
  en: {
    meta: {
      homeTitle: 'CVSS Calculator & MCP Server',
      homeDescription: 'Calculate CVSS scores in your browser or connect an AI agent to the MCP server.',
      mcpTitle: 'CVSS Calculator MCP Server for AI Agents',
      mcpDescription: 'Calculate CVSS Base Scores from an MCP client. Supports CVSS 2.0, 3.0, 3.1, and 4.0.',
    },
    common: {
      skip: 'Skip to main content',
      primaryNav: 'Primary navigation',
      language: 'Language',
      brandHome: 'CVSS.my.id home',
      themeToggle: 'Toggle color theme',
      themeLight: 'Light',
      themeDark: 'Dark',
      navigate: 'Navigate',
      copyStatus: 'Copied to clipboard.',
      copyBlocked: 'Clipboard blocked; select and copy the text above.',
      copyManual: 'Select and copy the text above.',
    },
    nav: {
      calculator: 'Calculator',
      mcp: 'MCP',
      docs: 'Docs',
      github: 'GitHub',
      mcpGithub: 'MCP GitHub',
    },
    home: {
      heroTitle: 'CVSS Calculator & MCP Server',
      heroLead: 'Calculate CVSS scores in your browser or connect an AI agent to the MCP server.',
      openCalculator: 'Open Calculator',
      configureMcp: 'Configure MCP Server',
      documentation: 'Documentation',
      calculatorHeading: 'Calculator',
      calculatorLead: 'Open the full browser calculator. Select the CVSS version inside.',
      primaryCardTitle: 'CVSS v4.0 Calculator',
      primaryCardTag: 'Current',
      primaryCardMeta: 'Base, Threat, Environmental, and Supplemental metrics.',
      otherVersions: 'Other versions:',
      mcpHeading: 'CVSS for AI Agents (MCP)',
      mcpLead: 'Connect an AI agent to calculate CVSS Base Scores using the calculate_cvss tool.',
      localTitle: '1. Local MCP (Recommended)',
      localDesc: 'Run the server on your machine. The public endpoint rate limit does not apply.',
      copyCommand: 'Copy Command',
      copyEndpoint: 'Copy Endpoint',
      localGuide: 'Local installation guide',
      remoteTitle: '2. Remote MCP (Free)',
      remoteDesc: 'Connect with Streamable HTTP. No local installation or API key required.',
      ratePolicy: 'Published usage policy:',
      rateLimit: 'Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).',
      remoteGuide: 'Remote connection guide',
      docsHeading: 'Documentation',
      docsLead: 'Learn about CVSS, the calculator, and MCP setup.',
      docsCvssTitle: 'CVSS Reference Guides',
      docsCvssDesc: 'Learn about CVSS metrics and scores.',
      docsVectorTitle: 'Vector String Syntax & Ordering Rules',
      docsVectorDesc: 'Learn how to read and write CVSS vectors.',
      docsClientsTitle: 'MCP Client Setup Guides',
      docsClientsDesc: 'Connect an MCP client to the server.',
      docsCalculatorTitle: 'Using the Browser Calculator',
      docsCalculatorDesc: 'Learn how to select metrics, edit vectors, and copy results.',
      ossHeading: 'Open source',
      ossLead: 'Explore the code on GitHub.',
      websiteRepoTitle: 'Website and calculator',
      websiteRepoDesc: 'The website, documentation, and browser calculator.',
      websiteRepoLink: 'Website repository on GitHub',
      mcpRepoTitle: 'CVSS Calculator MCP',
      mcpRepoDesc: 'An MCP server for CVSS Base Scores.',
      mcpRepoLink: 'MCP repository on GitHub',
      copyStatus: 'Copied to clipboard.',
      copyBlocked: 'Clipboard blocked; select and copy the text above.',
      copyManual: 'Select and copy the text above.',
    },
    mcp: {
      heroTitle: 'CVSS Calculator MCP Server',
      heroLead: 'Give your AI agent a tool to calculate CVSS Base Scores. Connect an MCP client locally or through the remote endpoint.',
      clientGuides: 'Client setup guides',
      toolReference: 'Tool reference',
      mcpRepo: 'MCP repository',
      registry: 'Official MCP Registry',
      setupHeading: 'Connect to the server',
      setupLead: 'Run the server locally or connect to the public endpoint.',
      localTitle: '1. Local MCP (Recommended)',
      localDesc: 'Run the server on your machine. The public endpoint rate limit does not apply.',
      copyCommand: 'Copy Command',
      localGuide: 'Local installation guide',
      remoteTitle: '2. Remote MCP (Free)',
      remoteDesc: 'Connect your MCP client without installing the server. The endpoint is not a web page.',
      copyEndpoint: 'Copy Endpoint',
      ratePolicy: 'Published usage policy:',
      rateLimit: 'Rate limit: 5 requests per minute per IP. For higher usage, run Local MCP (stdio).',
      remoteGuide: 'Remote connection guide',
      calculateHeading: 'Calculate a score',
      skillHeading: 'Vulnerability assessment skill',
      agentSetupHeading: 'Set up with an AI agent',
      agentSetupLead: 'Copy this prompt to your AI agent. It will ask you to choose and approve local or remote MCP before making changes, and ask separately before installing the optional skill.',
      copyPrompt: 'Copy prompt',
      skillLead: 'Guide your agent to assess vulnerability evidence, select CVSS metrics, and calculate Base Scores with the MCP server.',
      skillNote: 'Choose your agent when prompted. Add --global to install across projects.',
      skillGuide: 'Skill setup guide',
      calculateLead: 'The calculate_cvss tool accepts a complete Base vector and returns its score and severity.',
      input: 'Input',
      inputDesc: 'Provide a CVSS version and a complete Base vector. Vector values are case-sensitive.',
      output: 'Output',
      outputDesc: 'The response includes the version, vector, score, and severity.',
      copyJson: 'Copy JSON',
      docsHeading: 'Documentation',
      docsLead: 'Set up a client or learn more about the server.',
      clientsTitle: 'Client setup guides',
      clientsDesc: 'Connect an MCP client to the server.',
      toolsTitle: 'Tool reference',
      toolsDesc: 'See input formats and errors for calculate_cvss.',
      securityTitle: 'Security and privacy',
      securityDesc: 'Learn about checksums, network access, and data handling.',
      copyStatus: 'Copied to clipboard.',
      copyBlocked: 'Clipboard blocked; select and copy the text above.',
      copyManual: 'Select and copy the text above.',
    },
    status: {
      button: 'Check MCP status',
      notChecked: 'Not checked',
      checking: 'Checking...',
      available: 'Available from your connection.',
      unavailable: 'Unable to check from your connection.',
      rateLimited: 'Rate limited from your connection. Try again later.',
      checkedAt: 'Checked at',
      note: 'Checks from your connection and counts as a request.',
    },
    footer: {
      homeLinks: [
        { label: 'Calculator', href: '/calculator/v4.0' },
        { label: 'MCP Overview', href: '/tools/mcp' },
        { label: 'Documentation', href: '/docs/' },
        { label: 'MCP Client Guides', href: '/docs/mcp/clients/' },
        { label: 'Website & Calculator GitHub', href: 'https://github.com/alfa-reza/cvss.my.id', external: true },
        { label: 'MCP server on GitHub', href: 'https://github.com/alfa-reza/cvss-calculator-mcp', external: true },
      ],
      mcpLinks: [
        { label: 'Home', href: '/' },
        { label: 'Calculator', href: '/calculator/v4.0' },
        { label: 'MCP Docs', href: '/docs/mcp/' },
        { label: 'Client Setup Guides', href: '/docs/mcp/clients/' },
        { label: 'Website & Calculator GitHub', href: 'https://github.com/alfa-reza/cvss.my.id', external: true },
        { label: 'MCP server on GitHub', href: 'https://github.com/alfa-reza/cvss-calculator-mcp', external: true },
      ],
      attribution: 'CVSS.my.id is an independent open-source project and is not affiliated with FIRST. CVSS (Common Vulnerability Scoring System) is a standard maintained by ',
      attributionLink: 'FIRST.org, Inc.',
    },
  }
````````

---

# APPENDIX C — COMPLETE ORIGINAL ENGLISH WEBSITE PAGE & CALCULATOR REFERENCE SOURCES

These are the original full page templates and browser calculator interaction source, included to solve the Arena fetch/clone context limitation. **These are REFERENCE SOURCE, not a directive to keep Astro/Vite or old UI**. The new destination routes/components are given by Section 3 and Section 5. HTML/SVG icons included in these source examples are old implementation details; the new project must not create new images or hero sections. For each entry preserve factual text/functional semantics, not its old styling. Original site used some `lang === 'id'` branching in component logic; do NOT implement Indonesian UI, and extract only English text from Appendix B.

## C.01 — Homepage source template

**Original file:** `apps/website/src/components/HomeView.astro`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `8d9c182fcb65497e65efb4fa4ece3398c283f26ac9ca4173b46b2dfc5c2ae60e`

````````astro
---
import { ui, type Lang } from '../i18n/ui';
import McpStatus from './McpStatus.astro';

interface Props {
  lang: Lang;
}

const { lang } = Astro.props;
const t = ui[lang];
const calcBaseUrl = '/calculator';
const localInstallCmd = 'curl -fsSL https://get.cvss.my.id/install.sh | bash';
const remoteMcpUrl = 'https://cvss.my.id/mcp';
const docsBase = lang === 'id' ? '/id/docs' : '/docs';
const mcpPage = lang === 'id' ? '/id/tools/mcp' : '/tools/mcp';
---

<main id="main-content" class="container">
  <section class="hero-section" aria-labelledby="hero-heading">
    <h1 id="hero-heading" class="hero-title">{t.home.heroTitle}</h1>
    <p class="hero-lead">{t.home.heroLead}</p>
    <div class="cta-row">
      <a href={`${calcBaseUrl}/v4.0`} class="btn btn-primary">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="4" y="2" width="16" height="20" rx="2"></rect>
          <line x1="8" y1="6" x2="16" y2="6"></line>
          <line x1="16" y1="14" x2="16" y2="18"></line>
          <path d="M16 10h.01"></path>
          <path d="M12 10h.01"></path>
          <path d="M8 10h.01"></path>
          <path d="M12 14h.01"></path>
          <path d="M8 14h.01"></path>
          <path d="M12 18h.01"></path>
          <path d="M8 18h.01"></path>
        </svg>
        {t.home.openCalculator}
      </a>
      <a href={mcpPage} class="btn">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
          <line x1="8" y1="21" x2="16" y2="21"></line>
          <line x1="12" y1="17" x2="12" y2="21"></line>
        </svg>
        {t.home.configureMcp}
      </a>
      <a href={`${docsBase}/`} class="btn">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"></path>
          <path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"></path>
        </svg>
        {t.home.documentation}
      </a>
    </div>
  </section>

  <section class="section-block" aria-labelledby="versions-heading">
    <div class="section-header">
      <h2 id="versions-heading">{t.home.calculatorHeading}</h2>
      <p>{t.home.calculatorLead}</p>
    </div>
    <a href={`${calcBaseUrl}/v4.0`} class="version-card version-card-primary">
      <div>
        <div class="version-card-top">
          <span class="version-name">{t.home.primaryCardTitle}</span>
          <span class="version-tag">{t.home.primaryCardTag}</span>
        </div>
        <p class="version-meta">{t.home.primaryCardMeta}</p>
      </div>
      <span class="version-action">
        <span>{t.home.openCalculator}</span>
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <line x1="5" y1="12" x2="19" y2="12"></line>
          <polyline points="12 5 19 12 12 19"></polyline>
        </svg>
      </span>
    </a>
    <p class="version-alt">
      <span>{t.home.otherVersions}</span>
      <a href={`${calcBaseUrl}/v3.1`}>v3.1</a>
      <a href={`${calcBaseUrl}/v3.0`}>v3.0</a>
      <a href={`${calcBaseUrl}/v2.0`}>v2.0</a>
    </p>
  </section>

  <section class="section-block" aria-labelledby="mcp-heading">
    <div class="section-header">
      <h2 id="mcp-heading">{t.home.mcpHeading}</h2>
      <p set:html={t.home.mcpLead.replace('calculate_cvss', '<code>calculate_cvss</code>')} />
    </div>

    <div class="mcp-stack">
      <div class="mcp-panel is-recommended">
        <div class="mcp-panel-header">
          <div class="mcp-title-row">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <polyline points="4 17 10 11 4 5"></polyline>
              <line x1="12" y1="19" x2="20" y2="19"></line>
            </svg>
            <h3>{t.home.localTitle}</h3>
            <span class="mono-tag">stdio</span>
            <span class="mono-tag">Linux / macOS</span>
          </div>
        </div>
        <p class="mcp-desc">{t.home.localDesc}</p>
        <div class="command-row">
          <pre class="command-code"><code id="local-mcp-cmd">{localInstallCmd}</code></pre>
          <button
            type="button"
            class="btn copy-cmd-btn"
            data-copy-target="local-mcp-cmd"
            data-status-target="local-copy-status"
            aria-label={t.home.copyCommand}
          >
            {t.home.copyCommand}
          </button>
        </div>
        <div class="mcp-footer-row">
          <a href={`${docsBase}/mcp/local/`} class="text-link">{t.home.localGuide}</a>
          <span id="local-copy-status" class="copy-status" role="status" aria-live="polite"></span>
        </div>
      </div>

      <div class="mcp-panel">
        <div class="mcp-panel-header">
          <div class="mcp-title-row">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <circle cx="12" cy="12" r="10"></circle>
              <line x1="2" y1="12" x2="22" y2="12"></line>
              <path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"></path>
            </svg>
            <h3>{t.home.remoteTitle}</h3>
            <span class="mono-tag">Streamable HTTP</span>
          </div>
        </div>
        <p class="mcp-desc">{t.home.remoteDesc}</p>
        <div class="command-row">
          <pre class="command-code"><code id="remote-mcp-url">{remoteMcpUrl}</code></pre>
          <button
            type="button"
            class="btn copy-cmd-btn"
            data-copy-target="remote-mcp-url"
            data-status-target="remote-copy-status"
            aria-label={t.home.copyEndpoint}
          >
            {t.home.copyEndpoint}
          </button>
        </div>
        <p class="mcp-limit-note">
          <strong>{t.home.ratePolicy}</strong> {t.home.rateLimit}
        </p>
        <McpStatus lang={lang} />
        <div class="mcp-footer-row">
          <a href={`${docsBase}/mcp/remote/`} class="text-link">{t.home.remoteGuide}</a>
          <span id="remote-copy-status" class="copy-status" role="status" aria-live="polite"></span>
        </div>
      </div>
    </div>
  </section>

  <section class="section-block" aria-labelledby="docs-heading">
    <div class="section-header">
      <h2 id="docs-heading">{t.home.docsHeading}</h2>
      <p>{t.home.docsLead}</p>
    </div>
    <ul class="docs-index-list">
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M4 19.5v-15A2.5 2.5 0 0 1 6.5 2H20v20H6.5a2.5 2.5 0 0 1-2.5-2.5Z"></path>
          </svg>
          <a href={`${docsBase}/cvss/overview/`}>{t.home.docsCvssTitle}</a>
        </div>
        <p>{t.home.docsCvssDesc}</p>
      </li>
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <polyline points="16 18 22 12 16 6"></polyline>
            <polyline points="8 6 2 12 8 18"></polyline>
          </svg>
          <a href={`${docsBase}/cvss/vectors/`}>{t.home.docsVectorTitle}</a>
        </div>
        <p>{t.home.docsVectorDesc}</p>
      </li>
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
            <line x1="8" y1="21" x2="16" y2="21"></line>
            <line x1="12" y1="17" x2="12" y2="21"></line>
          </svg>
          <a href={`${docsBase}/mcp/clients/`}>{t.home.docsClientsTitle}</a>
        </div>
        <p>{t.home.docsClientsDesc}</p>
      </li>
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="4" y="2" width="16" height="20" rx="2"></rect>
            <line x1="8" y1="6" x2="16" y2="6"></line>
          </svg>
          <a href={`${docsBase}/calculator/`}>{t.home.docsCalculatorTitle}</a>
        </div>
        <p>{t.home.docsCalculatorDesc}</p>
      </li>
    </ul>
  </section>

  <section id="open-source" class="section-block" aria-labelledby="oss-heading">
    <div class="section-header">
      <h2 id="oss-heading">{t.home.ossHeading}</h2>
      <p>{t.home.ossLead}</p>
    </div>
    <div class="repo-grid">
      <div class="repo-box">
        <div>
          <div class="repo-header-top">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path>
            </svg>
            <h3>{t.home.websiteRepoTitle}</h3>
          </div>
          <p>{t.home.websiteRepoDesc}</p>
        </div>
        <a href="https://github.com/alfa-reza/cvss.my.id" class="repo-url" rel="noopener noreferrer" aria-label={t.home.websiteRepoLink}>
          <span>{t.home.websiteRepoLink}</span>
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <line x1="7" y1="17" x2="17" y2="7"></line>
            <polyline points="7 7 17 7 17 17"></polyline>
          </svg>
        </a>
      </div>

      <div class="repo-box">
        <div>
          <div class="repo-header-top">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path>
            </svg>
            <h3>{t.home.mcpRepoTitle}</h3>
          </div>
          <p>{t.home.mcpRepoDesc}</p>
        </div>
        <a href="https://github.com/alfa-reza/cvss-calculator-mcp" class="repo-url" rel="noopener noreferrer" aria-label={t.home.mcpRepoLink}>
          <span>{t.home.mcpRepoLink}</span>
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <line x1="7" y1="17" x2="17" y2="7"></line>
            <polyline points="7 7 17 7 17 17"></polyline>
          </svg>
        </a>
      </div>
    </div>
  </section>
</main>
````````

## C.02 — MCP setup page source template

**Original file:** `apps/website/src/components/McpView.astro`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `eed9ee24a1f4e76f2bfa7035751439c51895a8750a2fb4f0b9657cf9b747f13e`

````````astro
---
import { ui, type Lang } from '../i18n/ui';
import McpStatus from './McpStatus.astro';

interface Props {
  lang: Lang;
}

const { lang } = Astro.props;
const t = ui[lang];
const calcBaseUrl = '/calculator';
const localInstallCmd = 'curl -fsSL https://get.cvss.my.id/install.sh | bash';
const remoteMcpUrl = 'https://cvss.my.id/mcp';
const skillInstallCmd = 'npx skills add alfa-reza/cvss-calculator-mcp --skill vulnerability-cvss-assessment';
const agentSetupPrompt = 'Read https://get.cvss.my.id/LLM.txt and follow its instructions to set up CVSS Calculator MCP and offer to install its vulnerability assessment skill.';
const docsBase = lang === 'id' ? '/id/docs' : '/docs';
---

<main id="main-content" class="container">
  <section class="hero-section" aria-labelledby="mcp-page-heading">
    <h1 id="mcp-page-heading" class="hero-title">{t.mcp.heroTitle}</h1>
    <p class="hero-lead">{t.mcp.heroLead}</p>
    <div class="cta-row">
      <a href={`${docsBase}/mcp/clients/`} class="btn btn-primary">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
          <line x1="8" y1="21" x2="16" y2="21"></line>
          <line x1="12" y1="17" x2="12" y2="21"></line>
        </svg>
        {t.mcp.clientGuides}
      </a>
      <a href={`${docsBase}/mcp/tools/`} class="btn">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
          <polyline points="14 2 14 8 20 8"></polyline>
          <line x1="16" y1="13" x2="8" y2="13"></line>
          <line x1="16" y1="17" x2="8" y2="17"></line>
          <polyline points="10 9 9 9 8 9"></polyline>
        </svg>
        {t.mcp.toolReference}
      </a>
      <a href="https://github.com/alfa-reza/cvss-calculator-mcp" class="btn" rel="noopener noreferrer">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path>
        </svg>
        {t.mcp.mcpRepo}
      </a>
      <a href="https://registry.modelcontextprotocol.io/?q=CVSS&all=1" class="btn" rel="noopener noreferrer">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="2" y="2" width="20" height="8" rx="2"></rect>
          <rect x="2" y="14" width="20" height="8" rx="2"></rect>
          <line x1="6" y1="6" x2="6.01" y2="6"></line>
          <line x1="6" y1="18" x2="6.01" y2="18"></line>
        </svg>
        {t.mcp.registry}
      </a>
    </div>
  </section>

  <section class="section-block" aria-labelledby="agent-setup-heading">
    <div class="section-header">
      <h2 id="agent-setup-heading">{t.mcp.agentSetupHeading}</h2>
      <p>{t.mcp.agentSetupLead}</p>
    </div>
    <div class="command-row">
      <pre class="command-code"><code id="mcp-agent-prompt">{agentSetupPrompt}</code></pre>
      <button type="button" class="btn copy-cmd-btn" data-copy-target="mcp-agent-prompt" data-status-target="mcp-agent-status">{t.mcp.copyPrompt}</button>
    </div>
    <span id="mcp-agent-status" class="copy-status" role="status" aria-live="polite"></span>
  </section>

  <section class="section-block" aria-labelledby="setup-heading">
    <div class="section-header">
      <h2 id="setup-heading">{t.mcp.setupHeading}</h2>
      <p>{t.mcp.setupLead}</p>
    </div>

    <div class="mcp-stack">
      <div class="mcp-panel is-recommended">
        <div class="mcp-panel-header">
          <div class="mcp-title-row">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <polyline points="4 17 10 11 4 5"></polyline>
              <line x1="12" y1="19" x2="20" y2="19"></line>
            </svg>
            <h3>{t.mcp.localTitle}</h3>
            <span class="mono-tag">stdio</span>
            <span class="mono-tag">Linux / macOS</span>
          </div>
        </div>
        <p class="mcp-desc">{t.mcp.localDesc}</p>
        <div class="command-row">
          <pre class="command-code"><code id="mcp-local-cmd">{localInstallCmd}</code></pre>
          <button
            type="button"
            class="btn copy-cmd-btn"
            data-copy-target="mcp-local-cmd"
            data-status-target="mcp-local-status"
          >
            {t.mcp.copyCommand}
          </button>
        </div>
        <div class="mcp-footer-row">
          <a href={`${docsBase}/mcp/local/`} class="text-link">{t.mcp.localGuide}</a>
          <span id="mcp-local-status" class="copy-status" role="status" aria-live="polite"></span>
        </div>
      </div>

      <div class="mcp-panel">
        <div class="mcp-panel-header">
          <div class="mcp-title-row">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <circle cx="12" cy="12" r="10"></circle>
              <line x1="2" y1="12" x2="22" y2="12"></line>
              <path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"></path>
            </svg>
            <h3>{t.mcp.remoteTitle}</h3>
            <span class="mono-tag">Streamable HTTP</span>
          </div>
        </div>
        <p class="mcp-desc">{t.mcp.remoteDesc}</p>
        <div class="command-row">
          <pre class="command-code"><code id="mcp-remote-url">{remoteMcpUrl}</code></pre>
          <button
            type="button"
            class="btn copy-cmd-btn"
            data-copy-target="mcp-remote-url"
            data-status-target="mcp-remote-status"
          >
            {t.mcp.copyEndpoint}
          </button>
        </div>
        <p class="mcp-limit-note">
          <strong>{t.mcp.ratePolicy}</strong> {t.mcp.rateLimit}
        </p>
        <McpStatus lang={lang} />
        <div class="mcp-footer-row">
          <a href={`${docsBase}/mcp/remote/`} class="text-link">{t.mcp.remoteGuide}</a>
          <span id="mcp-remote-status" class="copy-status" role="status" aria-live="polite"></span>
        </div>
      </div>
    </div>
  </section>

  <section class="section-block" aria-labelledby="skill-heading">
    <div class="section-header">
      <h2 id="skill-heading">{t.mcp.skillHeading}</h2>
      <p>{t.mcp.skillLead}</p>
    </div>
    <div class="command-row">
      <pre class="command-code"><code id="mcp-skill-cmd">{skillInstallCmd}</code></pre>
      <button type="button" class="btn copy-cmd-btn" data-copy-target="mcp-skill-cmd" data-status-target="mcp-skill-status">{t.mcp.copyCommand}</button>
    </div>
    <p>{t.mcp.skillNote} <a href={`${docsBase}/mcp/#${lang === 'id' ? 'skill-penilaian-kerentanan' : 'vulnerability-assessment-skill'}`} class="text-link">{t.mcp.skillGuide}</a></p>
    <span id="mcp-skill-status" class="copy-status" role="status" aria-live="polite"></span>
  </section>

  <section class="section-block" aria-labelledby="contract-heading">
    <div class="section-header">
      <h2 id="contract-heading">{t.mcp.calculateHeading}</h2>
      <p set:html={t.mcp.calculateLead.replace('calculate_cvss', '<code>calculate_cvss</code>')} />
    </div>
    <div class="contract-grid">
      <div class="contract-card">
        <div class="contract-card-intro">
          <div class="contract-card-header">
            <div class="contract-card-title-row">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <polyline points="16 18 22 12 16 6"></polyline>
                <polyline points="8 6 2 12 8 18"></polyline>
              </svg>
              <h3>{t.mcp.input}</h3>
            </div>
            <span class="mono-badge">JSON</span>
          </div>
          <p class="contract-card-desc">{t.mcp.inputDesc}</p>
        </div>
        <div class="payload-box">
          <pre class="code-payload-block"><code id="mcp-input-payload">{`{
  "version": "3.1",
  "vector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H"
}`}</code></pre>
          <div class="payload-action-row">
            <button
              type="button"
              class="btn copy-cmd-btn"
              data-copy-target="mcp-input-payload"
              data-status-target="mcp-input-status"
              aria-label={t.mcp.copyJson}
            >
              {t.mcp.copyJson}
            </button>
            <span id="mcp-input-status" class="copy-status" role="status" aria-live="polite"></span>
          </div>
        </div>
      </div>
      <div class="contract-card">
        <div class="contract-card-intro">
          <div class="contract-card-header">
            <div class="contract-card-title-row">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                <polyline points="14 2 14 8 20 8"></polyline>
                <line x1="16" y1="13" x2="8" y2="13"></line>
                <line x1="16" y1="17" x2="8" y2="17"></line>
                <polyline points="10 9 9 9 8 9"></polyline>
              </svg>
              <h3>{t.mcp.output}</h3>
            </div>
            <span class="mono-badge">JSON</span>
          </div>
          <p class="contract-card-desc">{t.mcp.outputDesc}</p>
        </div>
        <div class="payload-box">
          <pre class="code-payload-block"><code id="mcp-output-payload">{`{
  "version": "3.1",
  "vector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
  "score": 9.8,
  "severity": "Critical"
}`}</code></pre>
          <div class="payload-action-row">
            <button
              type="button"
              class="btn copy-cmd-btn"
              data-copy-target="mcp-output-payload"
              data-status-target="mcp-output-status"
              aria-label={t.mcp.copyJson}
            >
              {t.mcp.copyJson}
            </button>
            <span id="mcp-output-status" class="copy-status" role="status" aria-live="polite"></span>
          </div>
        </div>
      </div>
    </div>
  </section>

  <section class="section-block" aria-labelledby="next-steps-heading">
    <div class="section-header">
      <h2 id="next-steps-heading">{t.mcp.docsHeading}</h2>
      <p>{t.mcp.docsLead}</p>
    </div>
    <ul class="docs-index-list">
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
            <line x1="8" y1="21" x2="16" y2="21"></line>
            <line x1="12" y1="17" x2="12" y2="21"></line>
          </svg>
          <a href={`${docsBase}/mcp/clients/`}>{t.mcp.clientsTitle}</a>
        </div>
        <p>{t.mcp.clientsDesc}</p>
      </li>
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
            <polyline points="14 2 14 8 20 8"></polyline>
            <line x1="16" y1="13" x2="8" y2="13"></line>
            <line x1="16" y1="17" x2="8" y2="17"></line>
          </svg>
          <a href={`${docsBase}/mcp/tools/`}>{t.mcp.toolsTitle}</a>
        </div>
        <p set:html={t.mcp.toolsDesc.replace('calculate_cvss', '<code>calculate_cvss</code>')} />
      </li>
      <li class="docs-index-item">
        <div class="docs-item-title-row">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
            <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
          </svg>
          <a href={`${docsBase}/mcp/security/`}>{t.mcp.securityTitle}</a>
        </div>
        <p>{t.mcp.securityDesc}</p>
      </li>
    </ul>
  </section>
</main>
````````

## C.03 — MCP status original source template

**Original file:** `apps/website/src/components/McpStatus.astro`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `d29b1e5b7c4285b35b800f3260bd34b8060322e026ce05059feb0c593e8838e4`

````````astro
---
import { ui, type Lang } from '../i18n/ui';

interface Props {
  lang: Lang;
}

const { lang } = Astro.props;
const t = ui[lang].status;
---

<div
  class="mcp-status-check"
  data-locale={lang === 'id' ? 'id-ID' : 'en-US'}
  data-checking={t.checking}
  data-available={t.available}
  data-unavailable={t.unavailable}
  data-rate-limited={t.rateLimited}
  data-checked-at={t.checkedAt}
>
  <button class="btn" type="button" data-mcp-check>{t.button}</button>
  <span class="mcp-status-line" data-mcp-status-line>
    <span class="mcp-status-dot" aria-hidden="true"></span>
    <span data-mcp-status role="status" aria-live="polite">{t.notChecked}</span>
  </span>
  <p>{t.note}</p>
</div>

<script>
  document.querySelectorAll<HTMLElement>('.mcp-status-check').forEach((container) => {
    const button = container.querySelector<HTMLButtonElement>('[data-mcp-check]');
    const status = container.querySelector<HTMLElement>('[data-mcp-status]');
    const line = container.querySelector<HTMLElement>('[data-mcp-status-line]');
    if (!button || !status) return;

    const locale = container.dataset.locale || 'en-US';
    const checking = container.dataset.checking || 'Checking...';
    const available = container.dataset.available || 'Available from your connection.';
    const unavailable = container.dataset.unavailable || 'Unable to check from your connection.';
    const rateLimited = container.dataset.rateLimited || 'Rate limited from your connection.';
    const checkedAt = container.dataset.checkedAt || 'Checked at';

    const time = (): string => new Date().toLocaleTimeString(locale);

    button.addEventListener('click', async () => {
      button.disabled = true;
      status.textContent = checking;
      line?.classList.remove('is-available', 'is-error');
      line?.classList.add('is-checking');
      const controller = new AbortController();
      const timeout = window.setTimeout(() => controller.abort(), 10000);
      try {
        const response = await fetch('/mcp', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json', 'Accept': 'application/json, text/event-stream' },
          body: JSON.stringify({
            jsonrpc: '2.0', id: 1, method: 'initialize',
            params: { protocolVersion: '2024-11-05', capabilities: {}, clientInfo: { name: 'cvss-website-status', version: '1.0.0' } },
          }),
          cache: 'no-store', signal: controller.signal,
        });
        let message = unavailable;
        let isAvailable = false;
        if (response.status === 429) {
          message = rateLimited;
        } else if (response.ok && response.body) {
          const reader = response.body.getReader();
          const decoder = new TextDecoder();
          let buffer = '';
          let valid = false;
          const isInitializeResponse = (text: string): boolean => {
            try {
              const data: unknown = JSON.parse(text);
              if (!data || typeof data !== 'object') return false;
              const result = data as { jsonrpc?: unknown; id?: unknown; result?: { protocolVersion?: unknown; serverInfo?: { name?: unknown } } };
              return result.jsonrpc === '2.0' && result.id === 1 && typeof result.result?.protocolVersion === 'string' && typeof result.result?.serverInfo?.name === 'string';
            } catch { return false; }
          };
          while (buffer.length < 65536) {
            const { value, done } = await reader.read();
            buffer += decoder.decode(value, { stream: !done });
            const contentType = response.headers.get('content-type') || '';
            if (contentType.includes('text/event-stream')) {
              valid = buffer.split(/\r?\n/).some((entry) => entry.startsWith('data:') && isInitializeResponse(entry.slice(5).trim()));
            } else if (contentType.includes('application/json')) {
              valid = isInitializeResponse(buffer);
            }
            if (valid || done) break;
          }
          void reader.cancel().catch(() => {});
          if (valid) {
            isAvailable = true;
            message = available;
          }
        }
        line?.classList.toggle('is-available', isAvailable);
        line?.classList.toggle('is-error', !isAvailable);
        status.textContent = `${message} ${checkedAt} ${time()}`;
      } catch {
        line?.classList.remove('is-available');
        line?.classList.add('is-error');
        status.textContent = `${unavailable} ${checkedAt} ${time()}`;
      } finally {
        line?.classList.remove('is-checking');
        window.clearTimeout(timeout);
        button.disabled = false;
      }
    });
  });
</script>
````````

## C.04 — Browser calculator shared interaction source

**Original file:** `apps/calculator/src/app.ts`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `6d93357a07f499e1225891ef24db0140dddd4c0c4a72c2e24e53215f5d60cc12`

````````typescript
import {
  calculateScore,
  createInitialMetrics,
  formatResultSummary,
  getMissingBaseMetrics,
  getVersionSchema,
  parseVector,
  serializeVector,
  type CvssVersion,
  type MetricDefinition,
  type ScoreResult,
  type SeverityLevel,
  type VersionSchema,
} from '@cvss/core';

const SCHEMA_VERSION = 1;

interface StoredDraft {
  schemaVersion: number;
  version: CvssVersion;
  metrics: Record<string, string>;
  activeTab: string;
  rawEditorText: string;
  lastValidVector: string;
  isEditorDirtyInvalid: boolean;
}

interface AppState {
  version: CvssVersion;
  schema: VersionSchema;
  metrics: Record<string, string>;
  activeTab: string;
  rawEditorText: string;
  lastValidVector: string;
  editorError: string | null;
  detectedOtherVersion: CvssVersion | null;
  normalizedOtherVector: string | null;
  hasUrlVectorOnLoad: boolean;
  urlError: string | null;
  feedbackMessage: string;
  fallbackCopyValue: string | null;
  showExplanations: boolean;
  lastGaugeValue: number;
}

function getStorageKey(version: CvssVersion): string {
  return `cvss_draft_v${SCHEMA_VERSION}_${version}`;
}

function loadDraftFromSession(version: CvssVersion): StoredDraft | null {
  try {
    const raw = window.sessionStorage.getItem(getStorageKey(version));
    if (!raw) return null;
    const parsed = JSON.parse(raw) as Partial<StoredDraft>;
    if (
      !parsed ||
      parsed.schemaVersion !== SCHEMA_VERSION ||
      parsed.version !== version ||
      typeof parsed.metrics !== 'object' ||
      parsed.metrics === null
    ) {
      window.sessionStorage.removeItem(getStorageKey(version));
      return null;
    }
    const cleanMetrics = createInitialMetrics(version);
    const schema = getVersionSchema(version);
    for (const def of schema.allMetrics) {
      const candidate = parsed.metrics[def.key];
      if (typeof candidate === 'string' && def.options.some((o) => o.value === candidate)) {
        cleanMetrics[def.key] = candidate;
      }
    }
    const validTab = schema.groups.some((g) => g.id === parsed.activeTab)
      ? (parsed.activeTab as string)
      : schema.groups[0].id;
    return {
      schemaVersion: SCHEMA_VERSION,
      version,
      metrics: cleanMetrics,
      activeTab: validTab,
      rawEditorText: typeof parsed.rawEditorText === 'string' ? parsed.rawEditorText : '',
      lastValidVector: typeof parsed.lastValidVector === 'string' ? parsed.lastValidVector : '',
      isEditorDirtyInvalid: Boolean(parsed.isEditorDirtyInvalid),
    };
  } catch {
    return null;
  }
}

function saveDraftToSession(state: AppState): void {
  try {
    const payload: StoredDraft = {
      schemaVersion: SCHEMA_VERSION,
      version: state.version,
      metrics: state.metrics,
      activeTab: state.activeTab,
      rawEditorText: state.rawEditorText,
      lastValidVector: state.lastValidVector,
      isEditorDirtyInvalid: state.editorError !== null,
    };
    window.sessionStorage.setItem(getStorageKey(state.version), JSON.stringify(payload));
  } catch {
    // sessionStorage may be blocked in strict privacy modes; fail gracefully
  }
}

function clearDraftFromSession(version: CvssVersion): void {
  try {
    window.sessionStorage.removeItem(getStorageKey(version));
  } catch {
    // ignore
  }
}

function clearStaleUrlVectorParam(state: AppState): void {
  if (!state.hasUrlVectorOnLoad) return;
  try {
    const url = new URL(window.location.href);
    if (url.searchParams.has('vector') || /^#(?:vector=|CVSS:|AV:)/.test(url.hash)) {
      url.searchParams.delete('vector');
      if (/^#(?:vector=|CVSS:|AV:)/.test(url.hash)) url.hash = '';
      const nextUrl = url.pathname + (url.searchParams.toString() ? `?${url.searchParams.toString()}` : '') + url.hash;
      window.history.replaceState(null, '', nextUrl);
    }
    state.hasUrlVectorOnLoad = false;
  } catch {
    state.hasUrlVectorOnLoad = false;
  }
}

function getSeverityClass(sev: SeverityLevel): string {
  switch (sev) {
    case 'None':
      return 'sev-none';
    case 'Low':
      return 'sev-low';
    case 'Medium':
      return 'sev-medium';
    case 'High':
      return 'sev-high';
    case 'Critical':
      return 'sev-critical';
  }
}

function escapeHtml(str: string): string {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

function countDefinedInGroup(version: CvssVersion, groupId: string, metrics: Record<string, string>): string {
  const schema = getVersionSchema(version);
  const group = schema.groups.find((g) => g.id === groupId);
  if (!group) return '0';
  const total = group.metrics.length;
  let defined = 0;
  for (const m of group.metrics) {
    const val = metrics[m.key];
    if (m.required) {
      if (val && val !== '') defined += 1;
    } else {
      if (val && val !== m.defaultValue) defined += 1;
    }
  }
  return mGroupSummary(group.id, defined, total);
}

function mGroupSummary(groupId: string, defined: number, total: number): string {
  if (groupId === 'base') {
    return `${defined}/${total}`;
  }
  return defined > 0 ? `+${defined}` : '0';
}

function shouldShowIndonesianNotice(): boolean {
  try {
    const override = new URLSearchParams(window.location.search).get('lang');
    if (override === 'en') return false;
    if (override === 'id') return true;
  } catch {
    // Ignore malformed URLs and fall back to the saved or browser preference.
  }

  let stored: string | null = null;
  try {
    stored = localStorage.getItem('cvss-lang');
  } catch {
    // Storage can be unavailable in strict privacy modes.
  }
  if (stored === 'en') return false;
  if (stored === 'id') return true;

  const languages = navigator.languages && navigator.languages.length
    ? navigator.languages
    : [navigator.language || ''];
  return languages.some((value) => {
    const tag = String(value).toLowerCase();
    return tag === 'id' || tag.indexOf('id-') === 0;
  });
}

function renderLanguageNotice(): void {
  if (!shouldShowIndonesianNotice()) return;
  const app = document.getElementById('calculator-app');
  if (!app || document.querySelector('.calc-lang-note')) return;
  const note = document.createElement('p');
  note.className = 'calc-lang-note';
  note.lang = 'id';
  note.innerHTML =
    'Kalkulator ini memakai istilah CVSS standar dalam bahasa Inggris. ' +
    '<a href="/id/docs/calculator/">Baca panduan kalkulator berbahasa Indonesia</a>.';
  app.parentElement?.insertBefore(note, app);
}

function initCalculator(): void {
  const root = document.getElementById('calculator-app');
  const urlBanner = document.getElementById('url-error-banner');
  const bodyVersion = document.body.getAttribute('data-cvss-version') as CvssVersion | null;
  if (!root || !bodyVersion) return;

  renderLanguageNotice();

  const version: CvssVersion = ['4.0', '3.1', '3.0', '2.0'].includes(bodyVersion) ? bodyVersion : '4.0';
  const schema = getVersionSchema(version);

  const state: AppState = {
    version,
    schema,
    metrics: createInitialMetrics(version),
    activeTab: schema.groups[0].id,
    rawEditorText: '',
    lastValidVector: '',
    editorError: null,
    detectedOtherVersion: null,
    normalizedOtherVector: null,
    hasUrlVectorOnLoad: false,
    urlError: null,
    feedbackMessage: '',
    fallbackCopyValue: null,
    showExplanations: true,
    lastGaugeValue: 0,
  };

  // 1. Check URL ?vector=... parameter first (URL takes precedence over sessionStorage)
  const urlParams = new URLSearchParams(window.location.search);
  let rawUrlVector = urlParams.get('vector');
  if (!rawUrlVector && window.location.hash.length > 1) {
    const rawHash = window.location.hash.slice(1);
    if (rawHash.startsWith('vector=')) {
      try { rawUrlVector = decodeURIComponent(rawHash.slice(7)); } catch { rawUrlVector = rawHash.slice(7); }
    } else if (rawHash.startsWith('CVSS:') || rawHash.startsWith('AV:')) {
      try { rawUrlVector = decodeURIComponent(rawHash); } catch { rawUrlVector = rawHash; }
    }
  }

  if (rawUrlVector !== null) {
    state.hasUrlVectorOnLoad = true;
    if (rawUrlVector.length > 512) {
      state.urlError = 'The shared URL vector parameter exceeds the maximum allowed length (512 characters).';
      state.rawEditorText = rawUrlVector.slice(0, 512);
      state.editorError = state.urlError;
    } else {
      const parsed = parseVector(version, rawUrlVector);
      if (parsed.ok) {
        state.metrics = parsed.metrics;
        state.lastValidVector = parsed.canonicalVector;
        state.rawEditorText = parsed.canonicalVector;
        state.editorError = null;
        saveDraftToSession(state);
      } else {
        // Invalid shared URL produces a visible error and never falls back silently to unrelated saved data
        state.urlError = `Invalid shared URL vector: ${parsed.error}`;
        state.rawEditorText = rawUrlVector;
        state.editorError = parsed.error;
        state.detectedOtherVersion = parsed.detectedOtherVersion ?? null;
        state.normalizedOtherVector = parsed.normalizedOtherVector ?? null;
      }
    }
  } else {
    // 2. Otherwise restore from sessionStorage if available
    const saved = loadDraftFromSession(version);
    if (saved) {
      state.metrics = saved.metrics;
      state.activeTab = saved.activeTab;
      state.lastValidVector = serializeVector(version, saved.metrics);
      if (saved.isEditorDirtyInvalid) {
        state.rawEditorText = saved.rawEditorText;
        const validation = parseVector(version, saved.rawEditorText);
        if (!validation.ok) {
          state.editorError = validation.error;
          state.detectedOtherVersion = validation.detectedOtherVersion ?? null;
          state.normalizedOtherVector = validation.normalizedOtherVector ?? null;
        } else {
          state.metrics = validation.metrics;
          state.lastValidVector = validation.canonicalVector;
          state.rawEditorText = validation.canonicalVector;
        }
      } else {
        state.rawEditorText = state.lastValidVector;
      }
    }
  }

  function updateUrlErrorBanner(): void {
    if (!urlBanner) return;
    if (state.urlError) {
      urlBanner.classList.remove('is-hidden');
      urlBanner.innerHTML = `
        <span><strong>Shared URL Error:</strong> ${escapeHtml(state.urlError)}</span>
        <button type="button" class="btn" id="dismiss-url-error-btn">Clear Invalid URL</button>
      `;
      const dismissBtn = document.getElementById('dismiss-url-error-btn');
      dismissBtn?.addEventListener('click', () => {
        state.urlError = null;
        clearStaleUrlVectorParam(state);
        state.rawEditorText = state.lastValidVector;
        state.editorError = null;
        state.detectedOtherVersion = null;
        state.normalizedOtherVector = null;
        saveDraftToSession(state);
        renderAll();
      });
    } else {
      urlBanner.classList.add('is-hidden');
      urlBanner.innerHTML = '';
    }
  }

  function renderMetricControl(metric: MetricDefinition, isDisabled: boolean): string {
    const currentVal = state.metrics[metric.key] ?? metric.defaultValue;
    const isMissingBase = metric.required && (!currentVal || currentVal === '');
    const selectedOption = metric.options.find((o) => o.value === currentVal);
    const stateTag = isMissingBase
      ? 'Required (Unselected)'
      : selectedOption
        ? `Selected: ${selectedOption.label}`
        : '';

    let controlHtml = '';
    if (metric.controlType === 'segmented') {
      const buttonsHtml = metric.options
        .map((opt) => {
          const isSelected = currentVal === opt.value;
          const indicatorHtml = isSelected
            ? `<span class="seg-indicator is-selected" aria-hidden="true"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg></span>`
            : `<span class="seg-indicator is-empty" aria-hidden="true"><span class="seg-circle"></span></span>`;
          return `
            <button
              type="button"
              role="radio"
              tabindex="${isSelected || (!currentVal && opt === metric.options[0]) ? '0' : '-1'}"
              aria-checked="${isSelected ? 'true' : 'false'}"
              class="seg-btn"
              data-metric-key="${escapeHtml(metric.key)}"
              data-metric-val="${escapeHtml(opt.value)}"
              title="${escapeHtml(opt.description)}"
              ${isDisabled ? 'disabled' : ''}
            >
              <span class="seg-label">${escapeHtml(opt.label)}</span>
              ${indicatorHtml}
            </button>
          `;
        })
        .join('');

      controlHtml = `
        <div
          class="segmented-group"
          role="radiogroup"
          aria-label="${escapeHtml(metric.name)} (${escapeHtml(metric.key)})"
        >
          ${buttonsHtml}
        </div>
      `;
    } else {
      const selectId = `select-metric-${metric.key}`;
      const optionsHtml = metric.options
        .map((opt) => {
          const isSelected = currentVal === opt.value;
          return `<option value="${escapeHtml(opt.value)}" ${isSelected ? 'selected' : ''}>${escapeHtml(opt.label)}</option>`;
        })
        .join('');

      controlHtml = `
        <div class="select-wrapper">
          <label for="${escapeHtml(selectId)}" class="metric-state-tag">Choose ${escapeHtml(metric.name)} (${escapeHtml(metric.key)})</label>
          <select
            id="${escapeHtml(selectId)}"
            class="metric-select"
            data-metric-select="${escapeHtml(metric.key)}"
            ${isDisabled ? 'disabled' : ''}
          >
            ${optionsHtml}
          </select>
        </div>
      `;
    }

    const optionHelpHtml = selectedOption
      ? `<p class="selected-option-help"><strong>${escapeHtml(selectedOption.label)}:</strong> ${escapeHtml(selectedOption.description)}</p>`
      : '';

    return `
      <div class="metric-card ${isMissingBase ? 'is-missing' : ''}">
        <div class="metric-top">
          <span class="metric-label">
            ${escapeHtml(metric.name)}
            <span class="metric-abbr">${escapeHtml(metric.key)}</span>
          </span>
          <span class="metric-state-tag">${escapeHtml(stateTag)}</span>
        </div>
        <p class="metric-desc">${escapeHtml(metric.description)}</p>
        ${controlHtml}
        ${optionHelpHtml}
      </div>
    `;
  }

  function renderMetricsColumn(): string {
    const activeGroup =
      state.schema.groups.find((g) => g.id === state.activeTab) ?? state.schema.groups[0];
    const isControlsDisabled = state.editorError !== null;

    const tabsHtml = state.schema.groups
      .map((group) => {
        const isSelected = group.id === activeGroup.id;
        const countBadge = countDefinedInGroup(state.version, group.id, state.metrics);
        return `
          <button
            type="button"
            role="tab"
            id="tab-${escapeHtml(group.id)}"
            aria-selected="${isSelected ? 'true' : 'false'}"
            aria-controls="panel-${escapeHtml(group.id)}"
            tabindex="${isSelected ? '0' : '-1'}"
            class="tab-btn"
            data-tab-id="${escapeHtml(group.id)}"
          >
            <span>${escapeHtml(group.label)}</span>
            <span class="tab-count">${escapeHtml(countBadge)}</span>
          </button>
        `;
      })
      .join('');

    // Group metrics by subGroup while preserving order
    const subGroups: Array<{ title: string; items: MetricDefinition[] }> = [];
    for (const m of activeGroup.metrics) {
      const existing = subGroups.find((sg) => sg.title === m.subGroup);
      if (existing) {
        existing.items.push(m);
      } else {
        subGroups.push({ title: m.subGroup, items: [m] });
      }
    }

    const subGroupsHtml = subGroups
      .map((sg) => {
        const cards = sg.items.map((m) => renderMetricControl(m, isControlsDisabled)).join('');
        return `
          <section class="sub-group-section" aria-label="${escapeHtml(sg.title)}">
            <h3 class="sub-group-title">${escapeHtml(sg.title)}</h3>
            <div class="metrics-list">
              ${cards}
            </div>
          </section>
        `;
      })
      .join('');

    return `
      <div class="metrics-column">
        <div class="metrics-toolbar">
          <label class="explain-toggle" for="explain-toggle">
            <input type="checkbox" id="explain-toggle" ${state.showExplanations ? 'checked' : ''} />
            <span>Show explanations</span>
          </label>
        </div>
        <div class="tabs-bar" role="tablist" aria-label="CVSS v${escapeHtml(state.version)} Metric Groups">
          ${tabsHtml}
        </div>
        <div
          id="panel-${escapeHtml(activeGroup.id)}"
          role="tabpanel"
          aria-labelledby="tab-${escapeHtml(activeGroup.id)}"
        >
          <div class="group-header">
            <div class="group-header-text">
              <div class="group-header-top">
                <h2>${escapeHtml(activeGroup.label)} Metrics</h2>
                <span class="group-badge">${activeGroup.id === 'base' ? 'Required' : 'Optional'}</span>
              </div>
              <p>${escapeHtml(activeGroup.description)}</p>
            </div>
          </div>
          ${subGroupsHtml}
        </div>
      </div>
    `;
  }

  function renderScorePanel(scoreResult: ScoreResult): string {
    const isOutOfSync = state.editorError !== null;
    const isActionable = scoreResult.isComplete && !isOutOfSync;
    const activeGroup =
      state.schema.groups.find((g) => g.id === state.activeTab) ?? state.schema.groups[0];

    let syncBadgeHtml = `<span class="sync-badge">Live</span>`;
    if (isOutOfSync) {
      syncBadgeHtml = `<span class="sync-badge is-warning">Out of sync (invalid vector draft)</span>`;
    } else if (!scoreResult.isComplete) {
      syncBadgeHtml = `<span class="sync-badge">Incomplete</span>`;
    }

    const sevClass = scoreResult.isComplete ? getSeverityClass(scoreResult.primarySeverity) : 'sev-none';
    const scoreColor = `var(--${sevClass})`;
    const targetScore = scoreResult.isComplete ? scoreResult.primaryScore : 0;
    const displayScoreText = scoreResult.isComplete ? scoreResult.primaryScore.toFixed(1) : '--';
    const severityText = scoreResult.isComplete ? scoreResult.primarySeverity : 'Incomplete';
    const scoreLabel = scoreResult.isComplete
      ? (!scoreResult.hasTemporalOrThreat && !scoreResult.hasEnvironmental ? 'Base' : 'Overall')
      : 'Score';
    const startScore = Math.max(0, Math.min(10, state.lastGaugeValue));
    const startFilled = (254.47 * (startScore / 10)).toFixed(2);
    const startPct = Math.max(0, Math.min(100, startScore * 10));

    let autoNote = 'Updates automatically as you select metrics.';
    if (isOutOfSync) {
      autoNote = 'Showing the last valid score. Fix the vector input to resume.';
    } else if (!scoreResult.isComplete) {
      autoNote = 'Awaiting required Base metrics.';
    }

    const subcard = (label: string, value: number | undefined): string => {
      const pct = value === undefined ? 0 : Math.max(0, Math.min(100, value * 10));
      const text = value === undefined ? '--' : value.toFixed(1);
      return `
        <div class="score-subcard">
          <div class="score-subcard-top">
            <span>${label}</span>
            <span class="score-subcard-value">${text}</span>
          </div>
          <div class="score-subbar" aria-hidden="true">
            <div class="score-subbar-fill" style="width:${pct}%"></div>
          </div>
        </div>
      `;
    };

    let subscoreCardsHtml = '';
    if (scoreResult.isComplete && scoreResult.impactScore !== undefined && scoreResult.exploitabilityScore !== undefined) {
      subscoreCardsHtml = `
        <div class="score-subgrid">
          ${subcard('Impact', scoreResult.impactScore)}
          ${subcard('Exploitability', scoreResult.exploitabilityScore)}
        </div>
      `;
    } else if (scoreResult.isComplete && scoreResult.version === '4.0') {
      subscoreCardsHtml = `
        <div class="score-subgrid">
          ${subcard('Base', scoreResult.baseScore)}
          <div class="score-subcard is-text">
            <div class="score-subcard-top">
              <span>Nomenclature</span>
              <span class="score-subcard-value">${escapeHtml(scoreResult.nomenclature ?? 'CVSS-B')}</span>
            </div>
          </div>
        </div>
      `;
    } else {
      subscoreCardsHtml = `
        <div class="score-subgrid">
          ${subcard(state.version === '4.0' ? 'Base' : 'Impact', undefined)}
          <div class="score-subcard is-text">
            <div class="score-subcard-top">
              <span>${state.version === '4.0' ? 'Nomenclature' : 'Exploitability'}</span>
              <span class="score-subcard-value">--</span>
            </div>
          </div>
        </div>
      `;
    }

    let breakdownHtml = '';
    if (scoreResult.isComplete) {
      if (scoreResult.version === '4.0') {
        const items: string[] = [];
        if (scoreResult.hasTemporalOrThreat || scoreResult.hasEnvironmental) {
          items.push(`
            <div class="subscore-item">
              <span class="subscore-label">Base Reference (CVSS-B)</span>
              <span class="subscore-val">${scoreResult.baseScore.toFixed(1)} (${escapeHtml(scoreResult.baseSeverity)})</span>
            </div>
          `);
        }
        if (scoreResult.macroVector) {
          items.push(`
            <div class="subscore-item">
              <span class="subscore-label">MacroVector</span>
              <span class="subscore-val">${escapeHtml(scoreResult.macroVector)}</span>
            </div>
          `);
        }
        breakdownHtml = items.length ? `<div class="score-breakdown">${items.join('')}</div>` : '';
      } else {
        const items: string[] = [];
        if (scoreResult.hasTemporalOrThreat || scoreResult.hasEnvironmental) {
          items.push(`
            <div class="subscore-item">
              <span class="subscore-label">Base Score</span>
              <span class="subscore-val">${scoreResult.baseScore.toFixed(1)} (${escapeHtml(scoreResult.baseSeverity)})</span>
            </div>
          `);
        }
        if (scoreResult.hasTemporalOrThreat && scoreResult.temporalScore !== undefined && scoreResult.temporalSeverity) {
          items.push(`
            <div class="subscore-item">
              <span class="subscore-label">Temporal Score</span>
              <span class="subscore-val">${scoreResult.temporalScore.toFixed(1)} (${escapeHtml(scoreResult.temporalSeverity)})</span>
            </div>
          `);
        }
        if (scoreResult.hasEnvironmental && scoreResult.environmentalScore !== undefined && scoreResult.environmentalSeverity) {
          items.push(`
            <div class="subscore-item">
              <span class="subscore-label">Environmental</span>
              <span class="subscore-val">${scoreResult.environmentalScore.toFixed(1)} (${escapeHtml(scoreResult.environmentalSeverity)})</span>
            </div>
          `);
        }
        breakdownHtml = items.length ? `<div class="score-breakdown">${items.join('')}</div>` : '';
      }
    }

    const missingHtml = !scoreResult.isComplete
      ? `<p class="missing-hint">Select required Base metrics: ${scoreResult.missingBaseMetrics.map((k) => `<code>${escapeHtml(k)}</code>`).join(', ')}</p>`
      : '';

    let vectorStatusText = 'Awaiting complete Base metrics';
    let vectorStatusClass = '';
    if (isOutOfSync) {
      vectorStatusText = 'Invalid vector syntax';
      vectorStatusClass = 'is-invalid';
    } else if (scoreResult.isComplete) {
      vectorStatusText = 'Valid canonical vector';
      vectorStatusClass = 'is-valid';
    }

    const placeholderVec =
      state.version === '4.0'
        ? 'CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N'
        : state.version === '2.0'
          ? 'AV:N/AC:L/Au:N/C:P/I:P/A:P'
          : `CVSS:${state.version}/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`;

    let errorBoxHtml = '';
    if (state.editorError) {
      const crossVersionBtn =
        state.detectedOtherVersion && state.normalizedOtherVector
          ? `<a class="btn btn-primary" href="/calculator/v${escapeHtml(state.detectedOtherVersion)}?vector=${encodeURIComponent(state.normalizedOtherVector)}">Open in CVSS v${escapeHtml(state.detectedOtherVersion)} Calculator</a>`
          : '';
      errorBoxHtml = `
        <div class="vector-error-box" id="vector-error-box" role="alert">
          <div><strong>Validation Error:</strong> ${escapeHtml(state.editorError)}</div>
          <div class="vector-error-actions">
            ${crossVersionBtn}
            <button type="button" class="btn" id="revert-vector-btn">Revert to last valid vector</button>
          </div>
        </div>
      `;
    }

    const fallbackHtml = state.fallbackCopyValue
      ? `
        <div class="fallback-copy-box">
          <label for="fallback-copy-input">Clipboard access unavailable. Copy manually below:</label>
          <input id="fallback-copy-input" class="fallback-copy-input" type="text" readonly value="${escapeHtml(state.fallbackCopyValue)}" />
        </div>
      `
      : '';

    return `
      <aside class="score-column" aria-label="CVSS Score and Vector Panel">
        <div class="score-panel ${isOutOfSync ? 'is-out-of-sync' : ''}" style="--score-color: ${scoreColor};">
          <div class="score-header">
            <h2 class="score-header-title">CVSS v${escapeHtml(state.version)} Result</h2>
            ${syncBadgeHtml}
          </div>

          <div class="score-hero" aria-live="polite">
            <div class="score-gauge">
              <svg viewBox="0 0 128 128" aria-hidden="true">
                <circle class="score-gauge-track" cx="64" cy="64" r="54" transform="rotate(135 64 64)"></circle>
                <circle class="score-gauge-arc" cx="64" cy="64" r="54" transform="rotate(135 64 64)" data-gauge-target="${targetScore}" style="stroke-dasharray:${startFilled} 339.29;"></circle>
              </svg>
              <div class="score-gauge-center">
                <div class="score-gauge-value ${scoreResult.isComplete ? '' : 'is-incomplete'}" id="score-value-display">${displayScoreText}</div>
                <div class="score-gauge-label">${scoreLabel}</div>
              </div>
            </div>
            <span class="severity-pill ${sevClass}" id="score-severity-display">${escapeHtml(severityText)}</span>
          </div>

          <div class="score-scale" aria-hidden="true">
            <div class="score-scale-track">
              <div class="score-scale-fill" style="width:${startPct}%"></div>
            </div>
            <div class="score-scale-labels"><span>0</span><span>2.5</span><span>5</span><span>7.5</span><span>10</span></div>
          </div>

          ${subscoreCardsHtml}
          ${breakdownHtml}

          <p class="score-auto-note ${isOutOfSync ? 'is-warning' : ''}">
            <span class="score-auto-dot" aria-hidden="true"></span>
            ${escapeHtml(autoNote)}
          </p>
          ${missingHtml}

          <div class="vector-editor-block">
            <div class="vector-label-row">
              <label for="inline-vector-input" class="vector-label">Vector String</label>
              <div class="vector-label-actions">
                <span class="vector-status ${vectorStatusClass}" id="vector-status-label">${escapeHtml(vectorStatusText)}</span>
                <button type="button" class="vector-share-btn" id="share-vector-btn" ${!isActionable ? 'disabled' : ''}>Share</button>
              </div>
            </div>
            <input
              id="inline-vector-input"
              type="text"
              class="vector-input ${isOutOfSync ? 'is-invalid' : ''}"
              value="${escapeHtml(state.rawEditorText)}"
              placeholder="${escapeHtml(placeholderVec)}"
              spellcheck="false"
              autocomplete="off"
              autocapitalize="off"
              aria-invalid="${isOutOfSync ? 'true' : 'false'}"
              aria-describedby="vector-status-label"
            />
            ${errorBoxHtml}
          </div>

          <div class="action-grid">
            <button type="button" class="btn btn-primary" id="copy-vector-btn" ${!isActionable ? 'disabled' : ''}>
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
                <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
              </svg>
              Copy Vector
            </button>
            <button type="button" class="btn" id="copy-result-btn" ${!isActionable ? 'disabled' : ''}>
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                <polyline points="14 2 14 8 20 8"></polyline>
                <line x1="16" y1="13" x2="8" y2="13"></line>
                <line x1="16" y1="17" x2="8" y2="17"></line>
                <polyline points="10 9 9 9 8 9"></polyline>
              </svg>
              Copy Result
            </button>
          </div>

          <div class="copy-feedback" id="copy-feedback-status" role="status" aria-live="polite">${escapeHtml(state.feedbackMessage)}</div>
          ${fallbackHtml}

          <div class="reset-row">
            <button type="button" class="btn" id="score-reset-group-btn" ${isOutOfSync ? 'disabled' : ''}>
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"></path>
                <path d="M3 3v5h5"></path>
              </svg>
              Reset ${escapeHtml(activeGroup.label)}
            </button>
            <button type="button" class="btn" id="reset-all-btn">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="15" y1="9" x2="9" y2="15"></line>
                <line x1="9" y1="9" x2="15" y2="15"></line>
              </svg>
              Reset All
            </button>
          </div>
        </div>
      </aside>
    `;
  }

  async function copyTextWithFallback(text: string, successLabel: string, buttonId?: string): Promise<void> {
    state.fallbackCopyValue = null;
    try {
      if (navigator.clipboard && typeof navigator.clipboard.writeText === 'function') {
        await navigator.clipboard.writeText(text);
        state.feedbackMessage = successLabel;
        renderAll();
        flashCopiedButton(buttonId);
        return;
      }
      throw new Error('Clipboard API unavailable');
    } catch {
      state.feedbackMessage = 'Clipboard blocked by browser. Select and copy the text below.';
      state.fallbackCopyValue = text;
      renderAll();
      flashCopiedButton(buttonId);
      const fallbackInput = document.getElementById('fallback-copy-input') as HTMLInputElement | null;
      fallbackInput?.focus();
      fallbackInput?.select();
    }
  }

  function flashCopiedButton(id: string | undefined): void {
    if (!id) return;
    const button = document.getElementById(id);
    if (!button) return;
    const original = button.innerHTML;
    button.classList.add('is-copied');
    button.innerHTML = `<svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><polyline points="20 6 9 17 4 12"></polyline></svg>Copied`;
    window.setTimeout(() => {
      if (!button.isConnected) return;
      button.classList.remove('is-copied');
      button.innerHTML = original;
    }, 1200);
  }

  function handleMetricChange(key: string, val: string): void {
    if (state.editorError !== null) return;
    clearStaleUrlVectorParam(state);
    state.urlError = null;
    state.feedbackMessage = '';
    state.fallbackCopyValue = null;
    state.metrics[key] = val;

    const missing = getMissingBaseMetrics(state.version, state.metrics);
    if (missing.length === 0) {
      const canonical = serializeVector(state.version, state.metrics);
      state.lastValidVector = canonical;
      state.rawEditorText = canonical;
    } else {
      state.lastValidVector = '';
      state.rawEditorText = '';
    }
    saveDraftToSession(state);
    renderAll();
  }

  function handleResetGroup(): void {
    if (state.editorError !== null) return;
    clearStaleUrlVectorParam(state);
    state.urlError = null;
    state.feedbackMessage = `Reset ${state.activeTab} metrics.`;
    state.fallbackCopyValue = null;

    const group = state.schema.groups.find((g) => g.id === state.activeTab);
    if (!group) return;
    for (const m of group.metrics) {
      state.metrics[m.key] = m.defaultValue;
    }

    const missing = getMissingBaseMetrics(state.version, state.metrics);
    if (missing.length === 0) {
      const canonical = serializeVector(state.version, state.metrics);
      state.lastValidVector = canonical;
      state.rawEditorText = canonical;
    } else {
      state.lastValidVector = '';
      state.rawEditorText = '';
    }
    saveDraftToSession(state);
    renderAll();
  }

  function handleResetAll(): void {
    clearStaleUrlVectorParam(state);
    clearDraftFromSession(state.version);
    state.metrics = createInitialMetrics(state.version);
    state.rawEditorText = '';
    state.lastValidVector = '';
    state.editorError = null;
    state.detectedOtherVersion = null;
    state.normalizedOtherVector = null;
    state.urlError = null;
    state.feedbackMessage = `Cleared CVSS v${state.version} draft.`;
    state.fallbackCopyValue = null;
    renderAll();
  }

  function handleVectorInput(newText: string): void {
    clearStaleUrlVectorParam(state);
    state.urlError = null;
    state.feedbackMessage = '';
    state.fallbackCopyValue = null;
    state.rawEditorText = newText;

    const trimmed = newText.trim();
    if (!trimmed) {
      // Empty vector input clears error if no valid vector was committed, or marks incomplete
      state.editorError = 'Vector field is empty. Type or paste a valid vector, or click Revert.';
      state.detectedOtherVersion = null;
      state.normalizedOtherVector = null;
      saveDraftToSession(state);
      renderAllPreserveInputFocus();
      return;
    }

    const parsed = parseVector(state.version, trimmed);
    if (parsed.ok) {
      state.metrics = parsed.metrics;
      state.lastValidVector = parsed.canonicalVector;
      state.editorError = null;
      state.detectedOtherVersion = null;
      state.normalizedOtherVector = null;
      saveDraftToSession(state);
      renderAllPreserveInputFocus();
    } else {
      state.editorError = parsed.error;
      state.detectedOtherVersion = parsed.detectedOtherVersion ?? null;
      state.normalizedOtherVector = parsed.normalizedOtherVector ?? null;
      saveDraftToSession(state);
      renderAllPreserveInputFocus();
    }
  }

  function renderAllPreserveInputFocus(): void {
    const inputEl = document.getElementById('inline-vector-input') as HTMLInputElement | null;
    const selStart = inputEl?.selectionStart ?? null;
    const selEnd = inputEl?.selectionEnd ?? null;
    const wasFocused = document.activeElement === inputEl;

    renderAll();

    if (wasFocused) {
      const nextInput = document.getElementById('inline-vector-input') as HTMLInputElement | null;
      if (nextInput) {
        nextInput.focus();
        if (selStart !== null && selEnd !== null) {
          try {
            nextInput.setSelectionRange(selStart, selEnd);
          } catch {
            // ignore
          }
        }
      }
    }
  }

  function attachEventListeners(scoreResult: ScoreResult): void {
    if (!root) return;
    // Tab switching + keyboard arrow navigation
    const tabBtns = Array.from(root.querySelectorAll<HTMLButtonElement>('[role="tab"]'));
    tabBtns.forEach((btn, idx) => {
      btn.addEventListener('click', () => {
        const tabId = btn.getAttribute('data-tab-id');
        if (tabId) {
          state.activeTab = tabId;
          saveDraftToSession(state);
          renderAll();
        }
      });

      btn.addEventListener('keydown', (e: KeyboardEvent) => {
        let nextIdx: number | null = null;
        if (e.key === 'ArrowRight') nextIdx = (idx + 1) % tabBtns.length;
        else if (e.key === 'ArrowLeft') nextIdx = (idx - 1 + tabBtns.length) % tabBtns.length;
        else if (e.key === 'Home') nextIdx = 0;
        else if (e.key === 'End') nextIdx = tabBtns.length - 1;

        if (nextIdx !== null) {
          e.preventDefault();
          const targetTab = tabBtns[nextIdx];
          const tabId = targetTab.getAttribute('data-tab-id');
          if (tabId) {
            state.activeTab = tabId;
            saveDraftToSession(state);
            renderAll();
            const newlyRenderedTab = document.getElementById(`tab-${tabId}`);
            newlyRenderedTab?.focus();
          }
        }
      });
    });

    // Segmented buttons
    const segBtns = root.querySelectorAll<HTMLButtonElement>('.seg-btn');
    segBtns.forEach((btn) => {
      btn.addEventListener('keydown', (event) => {
        if (!['ArrowRight', 'ArrowDown', 'ArrowLeft', 'ArrowUp'].includes(event.key)) return;
        event.preventDefault();
        const group = btn.closest('[role="radiogroup"]');
        const buttons = Array.from(group?.querySelectorAll<HTMLButtonElement>('.seg-btn') ?? []);
        const delta = event.key === 'ArrowRight' || event.key === 'ArrowDown' ? 1 : -1;
        buttons[(buttons.indexOf(btn) + delta + buttons.length) % buttons.length]?.click();
      });
      btn.addEventListener('click', () => {
        const key = btn.getAttribute('data-metric-key');
        const val = btn.getAttribute('data-metric-val');
        if (key && val !== null) {
          handleMetricChange(key, val);
          root.querySelector<HTMLButtonElement>(`[data-metric-key="${key}"][data-metric-val="${val}"]`)?.focus();
        }
      });
    });

    // Select dropdowns
    const selects = root.querySelectorAll<HTMLSelectElement>('[data-metric-select]');
    selects.forEach((sel) => {
      sel.addEventListener('change', () => {
        const key = sel.getAttribute('data-metric-select');
        if (key) {
          handleMetricChange(key, sel.value);
          root.querySelector<HTMLSelectElement>(`[data-metric-select="${key}"]`)?.focus();
        }
      });
    });

    // Inline vector input
    const vectorInput = document.getElementById('inline-vector-input') as HTMLInputElement | null;
    vectorInput?.addEventListener('input', () => {
      handleVectorInput(vectorInput.value);
    });
    vectorInput?.addEventListener('blur', () => {
      // On blur, if valid, normalize editor text to canonical vector
      if (state.editorError === null && state.lastValidVector) {
        state.rawEditorText = state.lastValidVector;
        saveDraftToSession(state);
        vectorInput.value = state.lastValidVector;
      }
    });

    // Revert invalid vector draft
    const revertBtn = document.getElementById('revert-vector-btn');
    revertBtn?.addEventListener('click', () => {
      state.rawEditorText = state.lastValidVector;
      state.editorError = null;
      state.detectedOtherVersion = null;
      state.normalizedOtherVector = null;
      state.urlError = null;
      saveDraftToSession(state);
      renderAll();
    });

    // Copy / Reset buttons
    const copyVectorBtn = document.getElementById('copy-vector-btn');
    copyVectorBtn?.addEventListener('click', () => {
      if (scoreResult.isComplete && state.editorError === null) {
        void copyTextWithFallback(scoreResult.canonicalVector, 'Copied canonical vector to clipboard.', 'copy-vector-btn');
      }
    });

    const copyResultBtn = document.getElementById('copy-result-btn');
    copyResultBtn?.addEventListener('click', () => {
      if (scoreResult.isComplete && state.editorError === null) {
        const summary = formatResultSummary(scoreResult);
        void copyTextWithFallback(summary, 'Copied score result to clipboard.', 'copy-result-btn');
      }
    });

    const scoreResetGroupBtn = document.getElementById('score-reset-group-btn');
    scoreResetGroupBtn?.addEventListener('click', handleResetGroup);

    const resetAllBtn = document.getElementById('reset-all-btn');
    resetAllBtn?.addEventListener('click', handleResetAll);

    const shareVectorBtn = document.getElementById('share-vector-btn');
    shareVectorBtn?.addEventListener('click', () => {
      if (!scoreResult.isComplete || state.editorError !== null) return;
      const url = new URL(window.location.href);
      url.search = '';
      url.hash = '';
      url.searchParams.set('vector', scoreResult.canonicalVector);
      void copyTextWithFallback(url.toString(), 'Shareable link copied to clipboard.', 'share-vector-btn');
    });

    const explainToggle = document.getElementById('explain-toggle') as HTMLInputElement | null;
    explainToggle?.addEventListener('change', () => {
      state.showExplanations = explainToggle.checked;
      root?.classList.toggle('hide-explanations', !state.showExplanations);
    });
  }

  function animateScoreGauge(): void {
    const arc = document.querySelector<SVGCircleElement>('.score-gauge-arc');
    if (!arc) return;
    const target = Math.max(0, Math.min(10, Number(arc.dataset.gaugeTarget ?? '0')));
    const filled = (254.47 * (target / 10)).toFixed(2);
    const pct = Math.max(0, Math.min(100, target * 10));
    requestAnimationFrame(() => {
      requestAnimationFrame(() => {
        arc.style.strokeDasharray = `${filled} 339.29`;
        const fill = document.querySelector<HTMLElement>('.score-scale-fill');
        if (fill) fill.style.width = `${pct}%`;
      });
    });
    state.lastGaugeValue = target;
  }

  function renderSeverityLevels(version: CvssVersion): string {
    const ranges: Array<{ sev: SeverityLevel; range: string; desc: string }> =
      version === '2.0'
        ? [
            { sev: 'Low', range: '0.0 - 3.9', desc: 'Minimal impact. Unauthorized information disclosure or local denial of service.' },
            { sev: 'Medium', range: '4.0 - 6.9', desc: 'Moderate impact. Partial disclosure, integrity compromise, or disruption of service.' },
            { sev: 'High', range: '7.0 - 10.0', desc: 'Complete compromise of system integrity, confidentiality, or availability.' },
          ]
        : [
            { sev: 'None', range: '0.0', desc: 'No security impact on the target system under the tested prerequisites.' },
            { sev: 'Low', range: '0.1 - 3.9', desc: 'Minimal impact, high privileges, or local physical access required.' },
            { sev: 'Medium', range: '4.0 - 6.9', desc: 'Moderate impact, user interaction, or specific conditions needed.' },
            { sev: 'High', range: '7.0 - 8.9', desc: 'Substantial confidentiality, integrity, or availability compromise.' },
            { sev: 'Critical', range: '9.0 - 10.0', desc: 'Remote unauthenticated compromise without user interaction.' },
          ];

    const cards = ranges
      .map(({ sev, range, desc }) => `
        <div class="severity-card">
          <div class="severity-card-top">
            <span class="severity-pill ${getSeverityClass(sev)}">${escapeHtml(sev)}</span>
            <span class="severity-range">${escapeHtml(range)}</span>
          </div>
          <p>${escapeHtml(desc)}</p>
        </div>
      `)
      .join('');

    return `
      <section class="severity-reference-section" aria-labelledby="severity-heading">
        <div class="severity-reference-header">
          <h2 id="severity-heading">CVSS Severity Levels</h2>
          <p>Qualitative rating ranges for CVSS v${escapeHtml(version)}.</p>
        </div>
        <div class="severity-grid ${version === '2.0' ? 'is-v2' : ''}">
          ${cards}
        </div>
      </section>
    `;
  }

  function renderAll(): void {
    if (!root) return;
    root.classList.toggle('hide-explanations', !state.showExplanations);
    updateUrlErrorBanner();
    const scoreResult = calculateScore(state.version, state.metrics);
    root.innerHTML = `
      <div class="calc-grid">
        ${renderMetricsColumn()}
        ${renderScorePanel(scoreResult)}
      </div>
      ${renderSeverityLevels(state.version)}
    `;
    attachEventListeners(scoreResult);
    animateScoreGauge();
  }

  renderAll();
  initThemeToggle();
}

function initThemeToggle(): void {
  const button = document.getElementById('theme-toggle');
  if (!button) return;
  const media = window.matchMedia('(prefers-color-scheme: light)');
  const currentTheme = (): string => document.documentElement.getAttribute('data-theme') || (media.matches ? 'light' : 'dark');
  const updateButton = (): void => {
    const theme = currentTheme();
    const next = theme === 'light' ? 'dark' : 'light';
    button.setAttribute('aria-label', `Color theme: ${theme}. Switch to ${next}`);
    button.setAttribute('title', `Switch to ${next} theme`);
    const label = button.querySelector('.theme-toggle-text');
    if (label) label.textContent = theme === 'light' ? 'Light' : 'Dark';
  };
  button.addEventListener('click', () => {
    const next = currentTheme() === 'light' ? 'dark' : 'light';
    document.documentElement.setAttribute('data-theme', next);
    try {
      localStorage.setItem('theme', next);
      localStorage.setItem('starlight-theme', next);
    } catch (_) { /* Theme remains active for this page. */ }
    updateButton();
  });
  media.addEventListener('change', updateButton);
  updateButton();
}

initCalculator();
````````

## C.05 — Browser calculator version chooser source

**Original file:** `apps/calculator/index.html`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `907a34f05e1ad0a03ec65068c39bba48c6c75269478020b6244ee7a86f9c7109`

````````html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CVSS Calculator</title>
    <meta name="description" content="Score vulnerabilities across CVSS 4.0, 3.1, 3.0, and 2.0 with instant metric evaluation." />
    <meta http-equiv="refresh" content="0; url=/calculator/v4.0" />
    <link rel="canonical" href="https://cvss.my.id/calculator/v4.0" />
    <link rel="icon" href="/favicon.ico" sizes="any" />
    <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    <link rel="stylesheet" href="/src/styles.css" />
    <script>
      (function() {
        try {
          const t = localStorage.getItem('starlight-theme') || localStorage.getItem('theme');
          if (t === 'light' || t === 'dark') {
            document.documentElement.setAttribute('data-theme', t);
          } else {
            document.documentElement.removeAttribute('data-theme');
          }
        } catch (_) {}
      })();
      window.location.replace('/calculator/v4.0' + window.location.search + window.location.hash);
    </script>
  </head>
  <body>
    <header class="site-header">
      <div class="container header-inner">
        <a href="/" class="brand" aria-label="CVSS.my.id home">
          <svg class="brand-logo-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M12 2v3"></path>
            <path d="M3.34 19a10 10 0 1 1 17.32 0"></path>
            <path d="m12 14 4.5-4.5"></path>
            <circle cx="12" cy="14" r="1.5" fill="currentColor"></circle>
          </svg>
          <span class="brand-text"><span class="brand-name">CVSS</span><span class="brand-tld">.my.id</span></span>
        </a>
        <div class="header-right">
          <nav aria-label="Primary navigation" class="site-nav">
            <a href="/calculator/v4.0" aria-current="page">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="4" y="2" width="16" height="20" rx="2"></rect><line x1="8" y1="6" x2="16" y2="6"></line><line x1="16" y1="14" x2="16" y2="18"></line><path d="M16 10h.01"></path><path d="M12 10h.01"></path><path d="M8 10h.01"></path><path d="M12 14h.01"></path><path d="M8 14h.01"></path><path d="M12 18h.01"></path><path d="M8 18h.01"></path></svg>
              <span>Calculator</span>
            </a>
            <a href="/tools/mcp">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg>
              <span>MCP</span>
            </a>
            <a href="/docs/">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg>
              <span>Docs</span>
            </a>
            <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path></svg>
              <span>GitHub</span>
            </a>
          </nav>
          <button type="button" id="theme-toggle" class="theme-toggle-btn" aria-label="Toggle color theme" title="Toggle theme">
            <svg class="theme-icon theme-icon-sun" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
            <svg class="theme-icon theme-icon-moon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
            <span class="theme-toggle-text"></span>
          </button>
        </div>
      </div>
    </header>
    <main class="container main-redirect">
      <h1>CVSS Calculator</h1>
      <p>Redirecting to the <a href="/calculator/v4.0">CVSS v4.0 Calculator</a>. Select a version directly if you are not redirected:</p>
      <ul class="redirect-links">
        <li><a href="/calculator/v4.0">CVSS v4.0 Calculator</a></li>
        <li><a href="/calculator/v3.1">CVSS v3.1 Calculator</a></li>
        <li><a href="/calculator/v3.0">CVSS v3.0 Calculator</a></li>
        <li><a href="/calculator/v2.0">CVSS v2.0 Calculator</a></li>
      </ul>
    </main>
    <footer class="site-footer">
      <div class="container footer-inner">
        <p>Independent open-source project. CVSS is maintained by <a href="https://www.first.org/cvss/" rel="noopener noreferrer">FIRST.org, Inc.</a></p>
      </div>
    </footer>
  </body>
</html>
````````

## C.06 — Browser calculator 4.0 shell

**Original file:** `apps/calculator/v4.0.html`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `3940bf91a5b83e6d4e734774310f93408f017654ad3981ecefc123fa5d5c7839`

````````html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CVSS v4.0 Calculator</title>
    <meta name="description" content="Score CVSS v4.0 Base, Threat, Environmental, and Supplemental metrics with official MacroVector interpolation." />
    <link rel="canonical" href="https://cvss.my.id/calculator/v4.0" />
    <link rel="icon" href="/favicon.ico" sizes="any" />
    <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    <link rel="stylesheet" href="/src/styles.css" />
    <script>
      (function() {
        try {
          const t = localStorage.getItem('starlight-theme') || localStorage.getItem('theme');
          if (t === 'light' || t === 'dark') {
            document.documentElement.setAttribute('data-theme', t);
          } else {
            document.documentElement.removeAttribute('data-theme');
          }
        } catch (_) {}
      })();
    </script>
  </head>
  <body data-cvss-version="4.0">
    <a href="#calculator-main" class="skip-link">Skip to calculator</a>
    <header class="site-header">
      <div class="container header-inner">
          <a href="/" class="brand" aria-label="CVSS.my.id home">
            <svg class="brand-logo-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M12 2v3"></path>
              <path d="M3.34 19a10 10 0 1 1 17.32 0"></path>
              <path d="m12 14 4.5-4.5"></path>
              <circle cx="12" cy="14" r="1.5" fill="currentColor"></circle>
            </svg>
            <span class="brand-text"><span class="brand-name">CVSS</span><span class="brand-tld">.my.id</span></span>
          </a>
        <div class="header-right">
          <nav aria-label="Primary navigation" class="site-nav">
            <a href="/calculator/v4.0" aria-current="page">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="4" y="2" width="16" height="20" rx="2"></rect><line x1="8" y1="6" x2="16" y2="6"></line><line x1="16" y1="14" x2="16" y2="18"></line><path d="M16 10h.01"></path><path d="M12 10h.01"></path><path d="M8 10h.01"></path><path d="M12 14h.01"></path><path d="M8 14h.01"></path><path d="M12 18h.01"></path><path d="M8 18h.01"></path></svg>
              <span>Calculator</span>
            </a>
            <a href="/tools/mcp">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg>
              <span>MCP</span>
            </a>
            <a href="/docs/">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg>
              <span>Docs</span>
            </a>
            <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path></svg>
              <span>GitHub</span>
            </a>
          </nav>
          <button type="button" id="theme-toggle" class="theme-toggle-btn" aria-label="Toggle color theme" title="Toggle theme">
            <svg class="theme-icon theme-icon-sun" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
            <svg class="theme-icon theme-icon-moon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
            <span class="theme-toggle-text"></span>
          </button>
        </div>
      </div>
    </header>

    <main id="calculator-main" class="container calc-main">
      <div class="calc-top-bar">
        <div class="calc-title-block">
          <h1>CVSS v4.0 Calculator</h1>
          <p class="calc-subtitle">Score Base (11), Threat (1), Environmental (14), and Supplemental (6) metrics with MacroVector lookup.</p>
        </div>
        <nav aria-label="CVSS version selector" class="version-switcher">
          <a href="/calculator/v4.0" class="version-link is-active" aria-current="page">v4.0</a>
          <a href="/calculator/v3.1" class="version-link">v3.1</a>
          <a href="/calculator/v3.0" class="version-link">v3.0</a>
          <a href="/calculator/v2.0" class="version-link">v2.0</a>
        </nav>
      </div>

      <div id="url-error-banner" class="alert-banner is-hidden" role="alert" aria-live="assertive"></div>

      <div id="calculator-app" class="calc-workspace" data-state="ready">
        <noscript>
          <div class="alert-banner">
            JavaScript is required to compute interactive CVSS scores in the browser. Read the static <a href="/docs/cvss/v4/">CVSS v4.0 Reference Guide</a> for specification details and vector formulas.
          </div>
        </noscript>
      </div>
    </main>

    <footer class="site-footer">
      <div class="container footer-inner">
        <div class="footer-links">
          <a href="/">Home</a>
          <a href="/docs/cvss/v4/">CVSS v4.0 Docs</a>
          <a href="/tools/mcp">MCP Server</a>
          <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">Website &amp; Calculator GitHub</a>
          <a href="https://github.com/alfa-reza/cvss-calculator-mcp" rel="noopener noreferrer">Go MCP GitHub</a>
        </div>
        <p class="footer-attribution">
          Independent open-source project. CVSS is maintained by <a href="https://www.first.org/cvss/v4.0/specification-document" rel="noopener noreferrer">FIRST.org, Inc.</a>
        </p>
      </div>
    </footer>

    <script type="module" src="/src/app.ts"></script>
  </body>
</html>
````````

## C.07 — Browser calculator 3.1 shell

**Original file:** `apps/calculator/v3.1.html`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `76225799a686904739ff72e51c02b1b5a5e2b05e9c1f0bc76d01aba1d13c0e66`

````````html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CVSS v3.1 Calculator</title>
    <meta name="description" content="Score CVSS v3.1 Base, Temporal, and Environmental metrics according to FIRST v3.1." />
    <link rel="canonical" href="https://cvss.my.id/calculator/v3.1" />
    <link rel="icon" href="/favicon.ico" sizes="any" />
    <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    <link rel="stylesheet" href="/src/styles.css" />
    <script>
      (function() {
        try {
          const t = localStorage.getItem('starlight-theme') || localStorage.getItem('theme');
          if (t === 'light' || t === 'dark') {
            document.documentElement.setAttribute('data-theme', t);
          } else {
            document.documentElement.removeAttribute('data-theme');
          }
        } catch (_) {}
      })();
    </script>
  </head>
  <body data-cvss-version="3.1">
    <a href="#calculator-main" class="skip-link">Skip to calculator</a>
    <header class="site-header">
      <div class="container header-inner">
          <a href="/" class="brand" aria-label="CVSS.my.id home">
            <svg class="brand-logo-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M12 2v3"></path>
              <path d="M3.34 19a10 10 0 1 1 17.32 0"></path>
              <path d="m12 14 4.5-4.5"></path>
              <circle cx="12" cy="14" r="1.5" fill="currentColor"></circle>
            </svg>
            <span class="brand-text"><span class="brand-name">CVSS</span><span class="brand-tld">.my.id</span></span>
          </a>
        <div class="header-right">
          <nav aria-label="Primary navigation" class="site-nav">
            <a href="/calculator/v3.1" aria-current="page">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="4" y="2" width="16" height="20" rx="2"></rect><line x1="8" y1="6" x2="16" y2="6"></line><line x1="16" y1="14" x2="16" y2="18"></line><path d="M16 10h.01"></path><path d="M12 10h.01"></path><path d="M8 10h.01"></path><path d="M12 14h.01"></path><path d="M8 14h.01"></path><path d="M12 18h.01"></path><path d="M8 18h.01"></path></svg>
              <span>Calculator</span>
            </a>
            <a href="/tools/mcp">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg>
              <span>MCP</span>
            </a>
            <a href="/docs/">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg>
              <span>Docs</span>
            </a>
            <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path></svg>
              <span>GitHub</span>
            </a>
          </nav>
          <button type="button" id="theme-toggle" class="theme-toggle-btn" aria-label="Toggle color theme" title="Toggle theme">
            <svg class="theme-icon theme-icon-sun" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
            <svg class="theme-icon theme-icon-moon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
            <span class="theme-toggle-text"></span>
          </button>
        </div>
      </div>
    </header>

    <main id="calculator-main" class="container calc-main">
      <div class="calc-top-bar">
        <div class="calc-title-block">
          <h1>CVSS v3.1 Calculator</h1>
          <p class="calc-subtitle">Score Base (8), Temporal (3), and Environmental (11) metrics with official v3.1 formulas.</p>
        </div>
        <nav aria-label="CVSS version selector" class="version-switcher">
          <a href="/calculator/v4.0" class="version-link">v4.0</a>
          <a href="/calculator/v3.1" class="version-link is-active" aria-current="page">v3.1</a>
          <a href="/calculator/v3.0" class="version-link">v3.0</a>
          <a href="/calculator/v2.0" class="version-link">v2.0</a>
        </nav>
      </div>

      <div id="url-error-banner" class="alert-banner is-hidden" role="alert" aria-live="assertive"></div>

      <div id="calculator-app" class="calc-workspace" data-state="ready">
        <noscript>
          <div class="alert-banner">
            JavaScript is required to compute interactive CVSS scores in the browser. Read the static <a href="/docs/cvss/v3.1/">CVSS v3.1 Reference Guide</a> for specification details and vector formulas.
          </div>
        </noscript>
      </div>
    </main>

    <footer class="site-footer">
      <div class="container footer-inner">
        <div class="footer-links">
          <a href="/">Home</a>
          <a href="/docs/cvss/v3.1/">CVSS v3.1 Docs</a>
          <a href="/tools/mcp">MCP Server</a>
          <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">Website &amp; Calculator GitHub</a>
          <a href="https://github.com/alfa-reza/cvss-calculator-mcp" rel="noopener noreferrer">Go MCP GitHub</a>
        </div>
        <p class="footer-attribution">
          Independent open-source project. CVSS is maintained by <a href="https://www.first.org/cvss/v3-1/specification-document" rel="noopener noreferrer">FIRST.org, Inc.</a>
        </p>
      </div>
    </footer>

    <script type="module" src="/src/app.ts"></script>
  </body>
</html>
````````

## C.08 — Browser calculator 3.0 shell

**Original file:** `apps/calculator/v3.0.html`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `ecac115156946905a26011b9a2713be864186c4044c0ec84741ac5629e5d97a8`

````````html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CVSS v3.0 Calculator</title>
    <meta name="description" content="Score CVSS v3.0 Base, Temporal, and Environmental metrics according to FIRST v3.0." />
    <link rel="canonical" href="https://cvss.my.id/calculator/v3.0" />
    <link rel="icon" href="/favicon.ico" sizes="any" />
    <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    <link rel="stylesheet" href="/src/styles.css" />
    <script>
      (function() {
        try {
          const t = localStorage.getItem('starlight-theme') || localStorage.getItem('theme');
          if (t === 'light' || t === 'dark') {
            document.documentElement.setAttribute('data-theme', t);
          } else {
            document.documentElement.removeAttribute('data-theme');
          }
        } catch (_) {}
      })();
    </script>
  </head>
  <body data-cvss-version="3.0">
    <a href="#calculator-main" class="skip-link">Skip to calculator</a>
    <header class="site-header">
      <div class="container header-inner">
          <a href="/" class="brand" aria-label="CVSS.my.id home">
            <svg class="brand-logo-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M12 2v3"></path>
              <path d="M3.34 19a10 10 0 1 1 17.32 0"></path>
              <path d="m12 14 4.5-4.5"></path>
              <circle cx="12" cy="14" r="1.5" fill="currentColor"></circle>
            </svg>
            <span class="brand-text"><span class="brand-name">CVSS</span><span class="brand-tld">.my.id</span></span>
          </a>
        <div class="header-right">
          <nav aria-label="Primary navigation" class="site-nav">
            <a href="/calculator/v3.0" aria-current="page">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="4" y="2" width="16" height="20" rx="2"></rect><line x1="8" y1="6" x2="16" y2="6"></line><line x1="16" y1="14" x2="16" y2="18"></line><path d="M16 10h.01"></path><path d="M12 10h.01"></path><path d="M8 10h.01"></path><path d="M12 14h.01"></path><path d="M8 14h.01"></path><path d="M12 18h.01"></path><path d="M8 18h.01"></path></svg>
              <span>Calculator</span>
            </a>
            <a href="/tools/mcp">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg>
              <span>MCP</span>
            </a>
            <a href="/docs/">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg>
              <span>Docs</span>
            </a>
            <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path></svg>
              <span>GitHub</span>
            </a>
          </nav>
          <button type="button" id="theme-toggle" class="theme-toggle-btn" aria-label="Toggle color theme" title="Toggle theme">
            <svg class="theme-icon theme-icon-sun" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
            <svg class="theme-icon theme-icon-moon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
            <span class="theme-toggle-text"></span>
          </button>
        </div>
      </div>
    </header>

    <main id="calculator-main" class="container calc-main">
      <div class="calc-top-bar">
        <div class="calc-title-block">
          <h1>CVSS v3.0 Calculator</h1>
          <p class="calc-subtitle">Score Base (8), Temporal (3), and Environmental (11) metrics with legacy v3.0 formulas.</p>
        </div>
        <nav aria-label="CVSS version selector" class="version-switcher">
          <a href="/calculator/v4.0" class="version-link">v4.0</a>
          <a href="/calculator/v3.1" class="version-link">v3.1</a>
          <a href="/calculator/v3.0" class="version-link is-active" aria-current="page">v3.0</a>
          <a href="/calculator/v2.0" class="version-link">v2.0</a>
        </nav>
      </div>

      <div id="url-error-banner" class="alert-banner is-hidden" role="alert" aria-live="assertive"></div>

      <div id="calculator-app" class="calc-workspace" data-state="ready">
        <noscript>
          <div class="alert-banner">
            JavaScript is required to compute interactive CVSS scores in the browser. Read the static <a href="/docs/cvss/v3.0/">CVSS v3.0 Reference Guide</a> for specification details and vector formulas.
          </div>
        </noscript>
      </div>
    </main>

    <footer class="site-footer">
      <div class="container footer-inner">
        <div class="footer-links">
          <a href="/">Home</a>
          <a href="/docs/cvss/v3.0/">CVSS v3.0 Docs</a>
          <a href="/tools/mcp">MCP Server</a>
          <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">Website &amp; Calculator GitHub</a>
          <a href="https://github.com/alfa-reza/cvss-calculator-mcp" rel="noopener noreferrer">Go MCP GitHub</a>
        </div>
        <p class="footer-attribution">
          Independent open-source project. CVSS is maintained by <a href="https://www.first.org/cvss/v3.0/specification-document" rel="noopener noreferrer">FIRST.org, Inc.</a>
        </p>
      </div>
    </footer>

    <script type="module" src="/src/app.ts"></script>
  </body>
</html>
````````

## C.09 — Browser calculator 2.0 shell

**Original file:** `apps/calculator/v2.0.html`  
**Destination:** redesigned React components within the corresponding route (`app/page.tsx`, `app/tools/mcp/page.tsx`, `app/calculator/`, etc.)  
**SHA-256:** `1f457776a87b1a01eb6bb2c65e5048f06661e02bb1e27790045c6524292890c1`

````````html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CVSS v2.0 Calculator</title>
    <meta name="description" content="Score CVSS v2.0 Base, Temporal, and Environmental metrics according to FIRST v2.0." />
    <link rel="canonical" href="https://cvss.my.id/calculator/v2.0" />
    <link rel="icon" href="/favicon.ico" sizes="any" />
    <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    <link rel="stylesheet" href="/src/styles.css" />
    <script>
      (function() {
        try {
          const t = localStorage.getItem('starlight-theme') || localStorage.getItem('theme');
          if (t === 'light' || t === 'dark') {
            document.documentElement.setAttribute('data-theme', t);
          } else {
            document.documentElement.removeAttribute('data-theme');
          }
        } catch (_) {}
      })();
    </script>
  </head>
  <body data-cvss-version="2.0">
    <a href="#calculator-main" class="skip-link">Skip to calculator</a>
    <header class="site-header">
      <div class="container header-inner">
          <a href="/" class="brand" aria-label="CVSS.my.id home">
            <svg class="brand-logo-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M12 2v3"></path>
              <path d="M3.34 19a10 10 0 1 1 17.32 0"></path>
              <path d="m12 14 4.5-4.5"></path>
              <circle cx="12" cy="14" r="1.5" fill="currentColor"></circle>
            </svg>
            <span class="brand-text"><span class="brand-name">CVSS</span><span class="brand-tld">.my.id</span></span>
          </a>
        <div class="header-right">
          <nav aria-label="Primary navigation" class="site-nav">
            <a href="/calculator/v2.0" aria-current="page">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="4" y="2" width="16" height="20" rx="2"></rect><line x1="8" y1="6" x2="16" y2="6"></line><line x1="16" y1="14" x2="16" y2="18"></line><path d="M16 10h.01"></path><path d="M12 10h.01"></path><path d="M8 10h.01"></path><path d="M12 14h.01"></path><path d="M8 14h.01"></path><path d="M12 18h.01"></path><path d="M8 18h.01"></path></svg>
              <span>Calculator</span>
            </a>
            <a href="/tools/mcp">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg>
              <span>MCP</span>
            </a>
            <a href="/docs/">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg>
              <span>Docs</span>
            </a>
            <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">
              <svg class="nav-icon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path></svg>
              <span>GitHub</span>
            </a>
          </nav>
          <button type="button" id="theme-toggle" class="theme-toggle-btn" aria-label="Toggle color theme" title="Toggle theme">
            <svg class="theme-icon theme-icon-sun" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
            <svg class="theme-icon theme-icon-moon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
            <span class="theme-toggle-text"></span>
          </button>
        </div>
      </div>
    </header>

    <main id="calculator-main" class="container calc-main">
      <div class="calc-top-bar">
        <div class="calc-title-block">
          <h1>CVSS v2.0 Calculator</h1>
          <p class="calc-subtitle">Score Base (6), Temporal (3), and Environmental (5) metrics on the official v2.0 scale.</p>
        </div>
        <nav aria-label="CVSS version selector" class="version-switcher">
          <a href="/calculator/v4.0" class="version-link">v4.0</a>
          <a href="/calculator/v3.1" class="version-link">v3.1</a>
          <a href="/calculator/v3.0" class="version-link">v3.0</a>
          <a href="/calculator/v2.0" class="version-link is-active" aria-current="page">v2.0</a>
        </nav>
      </div>

      <div id="url-error-banner" class="alert-banner is-hidden" role="alert" aria-live="assertive"></div>

      <div id="calculator-app" class="calc-workspace" data-state="ready">
        <noscript>
          <div class="alert-banner">
            JavaScript is required to compute interactive CVSS scores in the browser. Read the static <a href="/docs/cvss/v2/">CVSS v2.0 Reference Guide</a> for specification details and vector formulas.
          </div>
        </noscript>
      </div>
    </main>

    <footer class="site-footer">
      <div class="container footer-inner">
        <div class="footer-links">
          <a href="/">Home</a>
          <a href="/docs/cvss/v2/">CVSS v2.0 Docs</a>
          <a href="/tools/mcp">MCP Server</a>
          <a href="https://github.com/alfa-reza/cvss.my.id" rel="noopener noreferrer">Website &amp; Calculator GitHub</a>
          <a href="https://github.com/alfa-reza/cvss-calculator-mcp" rel="noopener noreferrer">Go MCP GitHub</a>
        </div>
        <p class="footer-attribution">
          Independent open-source project. CVSS is maintained by <a href="https://www.first.org/cvss/v2/guide" rel="noopener noreferrer">FIRST.org, Inc.</a>
        </p>
      </div>
    </footer>

    <script type="module" src="/src/app.ts"></script>
  </body>
</html>
````````

---

# APPENDIX D — Content integrity and implementation handoff

1. Content in Appendix A has been copied from the uploaded website repository snapshot's **English-only** folder `apps/website/src/content/docs/docs/`. Every destination is explicitly mapped in Section 3.2.
2. Appendix B captures all original English product/UI copy from `apps/website/src/i18n/ui.ts` and excludes the Indonesian translation block.
3. Appendix C contains original full Homepage, MCP, status UI, calculator controller and calculator version HTML source; implement the same features and route purposes in Next + React, not copy/paste original Astro component syntax into a new Next project.
4. The **only repository whose content may be modified by Arena** is `https://github.com/alfa-reza/cvss.my.id`. The Go MCP repository is reference-only.
5. Automated checks should hash each `content/docs/...` source against Appendix A or the original source snapshot; regenerate `public/docs/*.md` and `llms-full.txt` from the same input documents.
6. If project HEAD has newer source than this uploaded snapshot, preserve non-conflicting newer verified facts and report each discrepancy. Do not silently import stale data or replace a newer secure client command with an older example.
7. GitHub clone/network failures are NOT a basis for reducing the whole website to a one-page sample: this PRD includes all 31 docs and full original product page/controller references.
8. No site deployment/production changes, DNS changes, or editing the MCP repository without explicit owner authorization. After implementation, provide a route/feature acceptance table and actual test results.

**End of complete English-only PRD.**
