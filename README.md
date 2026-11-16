# Jym-Alexandre Nkana

**Computer Engineering student** at Brussels School of Engineering (ULB) · **Founder** of [Academic Weapon](https://academicweapon.dev) · **CEO & Co-founder** of [Wavydream](https://wavydream.com)

I design and ship production software end to end: data model, security, AI features, payments, deployment. I measure what I ship, and every bug I fix becomes a test that replays it.

*Open to internships and student positions in AI, Data Science, and Finance.*

---

## Academic Weapon · [academicweapon.dev](https://academicweapon.dev)

**An exam-prep platform for university students.** It turns a course into a study plan that fits around the student's timetable, practice sessions, flashcards and graded mock exams, with an AI tutor that answers only from the course material and cites it. In production at academicweapon.dev.

**Scale (Nov 2026):** 14 university courses · 197 course sheets · 334 planned study sessions · 6,391 hand-written practice questions, each linked to the exact passage that answers it.

**What's technically interesting**

- **Retrieval-augmented tutor.** Hybrid search inside Postgres: `pgvector` (1024-d `mistral-embed`, HNSW index) fused with full-text search (`tsvector`) by Reciprocal Rank Fusion. Answers stream through the Vercel AI SDK (Mistral, Gemini) with citations to the source passage. When the course doesn't cover a question, the tutor says so and lists what the chapter does cover instead of making something up.
- **AI grading, chosen by measurement.** Written answers, typed or photographed handwriting, are graded criterion by criterion against a rubric. The model returns a probability per criterion and the score is their weighted sum. An eval bench with known verdicts compares models and prompts on agreement, score error, ranking, cost and latency. On that benchmark, the selected setup (Gemini 3.5 Flash-Lite with a criteria-only prompt) agrees on 76 of 80 criteria, is off by 0.55/10 on average and costs about $0.00035 per answer, 40% less than a free-form prompt. Ministral 8B takes over typed answers when the primary model is rate-limited.
- **Security enforced by the database, not by app code.** Every user read and write hits Postgres with the user's own JWT under Row-Level Security (23 RLS-protected tables). Tokens (ES256) are verified against the identity provider's public JWKS with WebCrypto, with no shared secret. Sign-up is invite-only, rate limiting runs in Postgres, and LLM spend is capped per student and globally in the database.
- **Offline-first sync without data loss.** App state lives in the browser and merges with the server per entry (last write wins on timestamps), never as a whole document, so two devices can't overwrite each other. Writes wait for the first server read, a guard added after a fresh device could replace an account with its empty local state. Local data is bound to the account that wrote it, so a shared device never carries one student's data into another's account.
- **Scheduling engine.** The study plan is derived, never stored: sessions are placed in the free slots around the student's imported university timetable (ICS, fetched server-side with private-network protection). Only the student's adjustments are persisted, each with its own timestamp.
- **Payments without an SDK.** Stripe Checkout plus a webhook verified by HMAC over the raw body. Accounts are provisioned idempotently, so a replayed event can't send a second invitation.
- **Performance and cost.** Static-first Next.js 16: a page is dynamic only if it must read the session cookie. Math (KaTeX) is rendered on the server, so sheets of up to 250 KB never ship to the client. A CI job warms the CDN on each deploy, capped to stay inside the free tier. The launch animation runs on compositor-only `transform`, measured at 66 frames instead of 53 and a longest freeze of 67 ms instead of 185 ms while React hydrates.

**Quality:** strict TypeScript, ESLint with zero warnings, ~50k lines of TS/TSX and 50 SQL migrations. 40 purpose-built regression scripts replay real failure scenarios (sync ordering, slow or silent servers, multi-account devices, rendering all 197 sheets), and 25 custom linters guard the content and the codebase (math syntax, broken images, question references, migrations, design tokens).

`Next.js 16` `React 19` `TypeScript` `Tailwind v4` `PostgreSQL` `Supabase` `pgvector` `Vercel AI SDK` `Mistral` `Gemini` `Stripe` `Vercel` `GitHub Actions`

*Private codebase. Happy to walk through it in an interview.*

---

## Wavydream · [wavydream.com](https://wavydream.com)

**NFC business cards backed by a B2B lead and analytics platform.** A tap opens the holder's live digital card, where a visitor can save the contact (vCard, Apple Wallet), book a meeting or leave their details. Companies manage their fleet of cards, enforce their branding, and follow scans and leads in a dashboard. Customers include dental practices, insurance brokers and event organizers.

**What's technically interesting**

- **Tap to card, fast.** A tap resolves through `/go/<slug>` to the right state (unclaimed, active or inactive). Card pages are pre-rendered (ISR) from a materialized view and regenerated when a card changes. A weekly CI benchmark fails the build when the page goes over its byte budget. Lab baseline on a 4G profile: 62 ms TTFB, 0.5 s LCP, 247 kB.
- **Booking engine with guarantees in the database.** The availability engine is adapted from Cal.com's open-source code. Double booking is impossible by construction, enforced by a Postgres `EXCLUDE USING gist` constraint on time ranges and exercised by a concurrent race test. Confirmation emails go through an outbox drained every minute by `pg_cron` and `pg_net`, with hashed manage links and ICS feeds.
- **Multi-tenant security in Postgres.** Users, managers and admins; companies own fleets of cards, with brand overrides and locked fields enforced by the database. 26 RLS-protected tables, 133 policies, and triggers that guard role and tenant columns.
- **Scan analytics and lead pipeline.** Beacon-based event capture (source NFC/QR/direct, device, referrer, city-level location), validated with zod, deduplicated by a partial unique index and rate-limited in the database. Per-company analytics run behind role-checked SQL functions. Leads export to CSV/XLSX and push to any CRM that accepts webhooks, signed with HMAC-SHA256.
- **Built for real customers:** French, English and Dutch, installable as a PWA, Apple Wallet passes, vCard 3.0.

**Quality:** strict TypeScript (`noUncheckedIndexedAccess`), 2,400+ Vitest tests, 118 pytest API tests and SQL smoke tests, 78 SQL migrations. Six CI workflows: lint and typecheck, tests with coverage, security (gitleaks, npm audit, Semgrep, a scan of the live bundle), hourly production probes that open an issue when something fails, and the performance bench.

`Next.js 15` `React 19` `TypeScript` `Tailwind` `shadcn/ui` `PostgreSQL` `Supabase` `pg_cron` `zod` `Recharts` `Vitest` `pytest` `GitHub Actions` `Vercel`

*Private codebase. Product overview in [wavydream-showcase](https://github.com/PixisSensei/wavydream-showcase).*

---

## Also

- **AI workshops.** I design and run hands-on workshops that teach students to study with AI: source-grounded prompting, active recall and spaced repetition, with live polls and exercises on their own course material.
- **[Sports AI Web App](https://github.com/PixisSensei/sports-ai-webapp).** Training planner with AI-generated programs and a dietician chatbot.

## How I build

- **The database is the security boundary.** Row-Level Security first, app code second.
- **Measure, then claim.** Performance, cost and model quality get numbers before and after a change.
- **Every incident becomes a regression test** that replays it.
- **AI-assisted, human-owned.** I use Claude Code with a review pipeline (implementer and reviewer agents) and own every merge.

## Tech Stack

**Languages**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat&logo=postgresql&logoColor=white)
![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat&logo=cplusplus&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat&logo=openjdk&logoColor=white)

**AI & Data Science**

![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat&logo=numpy&logoColor=white)
![Scikit--learn](https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikitlearn&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat&logo=tensorflow&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat&logo=pytorch&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Mistral AI](https://img.shields.io/badge/Mistral%20AI-FA520F?style=flat&logo=mistralai&logoColor=white)
![Google Gemini](https://img.shields.io/badge/Gemini-8E75B2?style=flat&logo=googlegemini&logoColor=white)
![pgvector](https://img.shields.io/badge/pgvector-4169E1?style=flat&logo=postgresql&logoColor=white)

**Web & Cloud**

![React](https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat&logo=nextdotjs&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind-06B6D4?style=flat&logo=tailwindcss&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat&logo=supabase&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat&logo=firebase&logoColor=black)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=flat&logo=stripe&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat&logo=vercel&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat&logo=githubactions&logoColor=white)

## Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/nkana-jym)
[![Email](https://img.shields.io/badge/Email-EA4335?style=flat&logo=gmail&logoColor=white)](mailto:jym.nkana@gmail.com)
[![Academic Weapon](https://img.shields.io/badge/Academic%20Weapon-academicweapon.dev-111111?style=flat)](https://academicweapon.dev)
[![Wavydream](https://img.shields.io/badge/Wavydream-wavydream.com-111111?style=flat)](https://wavydream.com)

---

*Brussels, Belgium | Eischen, Luxembourg*
