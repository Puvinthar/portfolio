# Puvinthar S: Portfolio Implementation Plan (Final, free-only)

**Positioning:** Backend & AI Engineer. ETL, APIs, and cloud deployments. **Look:** dark, cinematic, precise. Near-black `#0A0A0B`, azure-to-cyan accent, Geist / Inter / JetBrains Mono. **Approach:** static Git-backed site, plus one edge worker for chat and contact. No servers, no database, no paid services.

> Free-tier limits change. Check each provider's pricing page when you create the account, and treat the numbers below as "free tier exists", not guarantees.

---

## 1. Cost: everything free

| Need | Tool | Cost |
| --- | --- | --- |
| Site hosting | Vercel Hobby (keep `puvinthar-stephen.vercel.app`) | Free |
| Edge worker (chat + contact) | Cloudflare Workers | Free tier |
| Chat LLM + embeddings | Cloudflare Workers AI | Free daily allowance |
| Chat index storage | Worker KV or Vectorize (later) | Free tier |
| Bot protection | Cloudflare Turnstile | Free |
| Contact email | Resend free tier (fallback: `mailto:`) | Free |
| Admin CMS | Keystatic (open source) via GitHub login | Free |
| CI/CD | GitHub Actions (free for public repos) | Free |
| Analytics | Vercel Analytics or Plausible alternative | Free tier |
| Fonts, icons | `next/font` (Geist, Inter, JetBrains Mono), Lucide | Free |
| Domain | Default `*.vercel.app` (custom domain optional, paid) | Free |

Because the repo is public (needed for free Actions minutes), **nothing confidential goes in it**: no SRY Labs code, client names or screenshots.

---

## 2. Architecture

```
  Visitor ──► Vercel: Next.js App Router, static (SSG/ISR)
              GSAP + Lenis + Tailwind
                 │ browser calls (CORS locked to your site)
                 ▼
  Cloudflare Worker
   ├─ POST /chat     → Turnstile → rate limit → context → Workers AI → streamed reply
   └─ POST /contact  → Turnstile → validate → Resend → your inbox
   Secrets: wrangler secrets (never in the repo)

  Content flow (you are the only editor):
  /admin (Keystatic, GitHub login) ─► commit to repo ─► GitHub Actions
      ├─ validate content (Zod + confidentiality blocklist)
      ├─ Python script rebuilds chat index ─► Worker
      └─ Vercel auto-deploys the site (~1 min)
```

---

## 3. Tech stack

- **Web:** Next.js (App Router), TypeScript, Tailwind CSS
- **Animation:** GSAP (ScrollTrigger, SplitText), Lenis smooth scroll synced to the GSAP ticker, Motion for small UI interactions
- **Content:** JSON and MDX in `/content`, validated with Zod
- **Admin:** Keystatic in GitHub mode (fallback: Decap CMS)
- **Worker:** Cloudflare Workers (TypeScript, Wrangler), Workers AI
- **Index builder:** Python script in GitHub Actions
- **Testing:** Playwright (E2E), Vitest (Worker), Lighthouse CI

---

## 4. Content plan

**Hero:** Puvinthar S, Backend & AI Engineer. One-line value proposition, links to GitHub, LinkedIn, resume.

**Experience timeline**

- **SRY Labs, AI Engineer Intern (current).** Start date to confirm; the offer letter is dated 25 Aug 2026.
- **Postiefs, DevOps Intern (Jun to Jul 2025).** GitHub Actions CI/CD, Docker, AWS.
- **Codsoft, Data Science Intern (Jul to Aug 2024).** Pipelines over 1M+ data points, fraud detection, SQL.
- **NullClass, AI Engineer Intern (Jan to Feb 2025).** Shown only if you confirm it.

**Featured projects**

1. **LMS Backend** (employer work, sanitized, no repo). Auth, communication and course endpoints in Nest.js, about 30 tests per module and about 150 in total, deployed on Azure with GitHub Actions CI/CD, plus a frontend built to verify the workflow end to end. Footer note: "Source is private under company confidentiality."
2. **NL2SQL Agent** (`agent-sql`). FastAPI, LangGraph, FAISS, PostgreSQL.
3. **No-Code ETL Framework** (`etl-no-code`). Ingestion from REST, CSV and JSON into PostgreSQL, with outlier detection and scaling.
4. **Jarvis** (`jarvis`). Offline voice assistant, 92% command accuracy, packaged as a 40 MB exe.
5. **Oracle-to-Databricks Pipeline** (no repo). Built in a one-week assessment through the Databricks CLI; shown as a short entry with an architecture diagram and no code link.

**How I build with AI:** your `rules.md` / `skills.md` / plugins workflow, a before/after example, and "about 50 commits in a month".

**Skills matrix**

| Group | Items |
| --- | --- |
| Backend | Python, FastAPI, Nest.js |
| ORM | SQLAlchemy, Prisma |
| Data | PostgreSQL, ETL/ELT pipelines |
| Frontend | Next.js, Tailwind CSS |
| Cloud / DevOps | Azure (Container Apps, managed Postgres, Communication Services), Docker, GitHub Actions |
| Testing | Playwright, Postman |
| AI | RAG, FAISS, LangGraph, NL2SQL, agent workflow (Cursor, Codex, Claude Code) |
| Observability | ELK stack |

**Also:** certifications (IBM AI Engineer, AWS Analytics, Accenture Forage), CSI General Secretary, contact section.

**Metrics policy:** use only numbers you can explain in an interview: about 150 tests, about 50 commits in a month, 92% (Jarvis). Leave out the 4 to 5 s API latency unless you can add context. "5+ microservices" and "40% FAQ retrieval" stay off until you confirm them.

---

## 5. Content files (`/content`)

```
profile.json · skills.json · certifications.json · settings.json
experience/*.json     company, role, dates, bullets, stack, order, visible
projects/*.mdx        slug, title, summary, stack, outcomes, repoUrl|null,
                      demoUrl, featured, order, status, sanitized
ai-workflow.mdx
```

The same files drive the site, the admin forms and the chat index.

---

## 6. Business logic

- **Draft vs. published:** drafts are excluded from builds and the chat index.
- **Confidentiality gate:** `sanitized: true` projects need a ticked checklist in the CMS. A CI step fails the build if any term from `blocklist.txt` (client names, internal product names) appears in content.
- **No-repo projects:** the repo button hides itself when `repoUrl` is null, and a "private / confidential" note shows instead.
- **Chat:**
  - Answers only from published content, cites the section, and says "I don't know" otherwise.
  - Fixed system prompt, short answers, no tools or actions.
  - Starter chips such as "What did you build at SRY Labs?".
  - Starts with the whole profile in the prompt, and moves to a Vectorize index only if content outgrows it.
- **Contact:** validate, check Turnstile, rate limit, send via Resend, and show a clear success or error. `mailto:` fallback if the Worker is down.
- **Kill switch:** `settings.json` can hide the chat widget instantly.

---

## 7. Admin (`/admin`, owner only)

1. GitHub sign-in through Keystatic. Only accounts with **write access to your repo** can save, and that is only you.
2. Enable **2FA or a passkey on GitHub** as the second factor.
3. Every save is a commit, which gives you a free audit trail and one-click rollback.
4. Branch protection on `main`, `noindex`, and excluded from the sitemap.
5. Edit profile, experience, projects (MDX editor), skills, certifications, AI-workflow text, hero copy, toggles and images. Changes go live in about a minute.
6. Fallback: edit the files directly in Cursor and push.

---

## 8. Security

| Area | Control |
| --- | --- |
| Admin | Repo-writer-only GitHub login, GitHub 2FA, `noindex` |
| Worker | CORS locked to your domain, size limits, Zod validation |
| Abuse | Turnstile, per-IP rate limits, global daily cap, kill switch |
| Chat | Fixed prompt, retrieval-only answers, output cap, no private data in the index |
| Secrets | `wrangler secret` and GitHub secrets, secret scanning and push protection on |
| Headers | CSP, HSTS, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy` |
| Content | MDX without raw HTML or scripts |
| Supply chain | Dependabot, `npm audit`, lockfiles |
| Privacy | Cookie-free analytics, no stored contact data |
| Confidentiality | Checklist plus CI blocklist |

---

## 9. CI/CD (GitHub Actions)

- **PR:** lint, typecheck, content validation, build, Playwright, Lighthouse CI, `npm audit`.
- **Merge to `main`:** Vercel deploys.
- **`/content` changes:** Python script chunks content and refreshes the chat index in the Worker.
- **`/worker` changes:** Vitest, then `wrangler deploy`.
- **Weekly:** dependency scan and link checker.

---

## 10. Animation and UX

- **Hero:** SplitText reveal with blur-to-sharp, magnetic CTAs, mini terminal typing a `rules.md` snippet.
- **Global:** Lenis smooth scroll, progress bar, clip-path page wipes, desktop-only custom cursor.
- **Sections:** velocity-reactive stack marquee, pinned experience timeline, spotlight-hover project cards, pinned AI-workflow step-through.
- **Chat widget:** floating "Ask about my work" button, slide-up panel, streaming replies.
- **Budgets:** LCP under 2.5 s, animate transform and opacity only, lazy-load heavy effects, reduced-motion fallbacks, AA contrast, keyboard support.

---

## 11. Repo structure

```
portfolio/
├─ app/ · components/ · lib/ · styles/
├─ content/ · public/
├─ keystatic.config.ts
├─ worker/              # Cloudflare Worker + wrangler.toml
├─ scripts/build_index.py · check_content.ts
├─ e2e/ · docs/ · .github/workflows/
├─ rules.md · skills.md
└─ README.md
```

---

## 12. Phases

| Phase | Scope | Done when |
| --- | --- | --- |
| 1. Foundation | Next.js scaffold, Tailwind tokens, fonts, Lenis + GSAP, layout, nav, page transitions | Smooth-scroll shell runs locally |
| 2. Content layer | Schemas, seed content from your resume and GitHub, Keystatic admin, validation script | You can log in, edit, and see the change |
| 3. Homepage | All sections animated and responsive | Full scroll experience works |
| 4. Case studies | MDX template, diagrams, sanitized LMS page, Databricks entry, `/resume` | 5 project pages live |
| 5. Edge worker | Cloudflare account, contact and chat endpoints, widget UI | Both work on staging |
| 6. Index pipeline | Python index builder in Actions | Editing content updates chat answers |
| 7. Polish and ship | Playwright and Lighthouse pass, SEO and OG images, header check, deploy, update LinkedIn and GitHub README | Live and linked everywhere |

---

## 13. Still to confirm (defaults in brackets)

1. SRY Labs start date. \[Show "Aug 2026 to present" until confirmed.\]
2. NullClass on the timeline? \[Off until confirmed.\]
3. "5+ microservices" and "40% FAQ retrieval" claims. \[Off until confirmed.\]
4. LMS entry. \[Publish the generic, resume-level description only: role, company, Nest.js, tests, Azure CI/CD. No client names, screenshots, code, schema or endpoint names. Optional courtesy heads-up to Yeshwanth; remove immediately if asked.\]
5. Chat model. \[Workers AI default.\]