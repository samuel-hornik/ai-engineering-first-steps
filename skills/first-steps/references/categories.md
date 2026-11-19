# Decision categories

Each category lists **what it decides**, **sensible defaults** for common situations, and **signals** that should change the default. Defaults are starting points, not rules: pick what fits the app the user described. Prefer boring, well-documented, widely used choices; prefer managed services and fewer moving parts for beginners, solo builders and small scale.

Shorthand used below:
- **Small** = prototype or up to ~1,000 users. **Medium** = ~1k–100k. **Large** = 100k+.
- **Beginner / Some / Experienced** = the user's self-reported experience.

---

## 1. Core features & user flows

**Decides:** the MVP feature list (must-have vs. later), the main user flows step by step, user roles.

- Push for a small MVP: 3–7 core features. Move everything else to "Later".
- Write the primary flow as numbered steps (e.g. *Sign up → create a shop → add a product → share link → receive order*).
- Identify roles early (visitor, user, admin, team owner…), because they drive auth and data access.

**Signals:** a long wish list → help the user cut; marketplace / two-sided apps → two primary flows (one per side).

## 2. Look & feel (UI/UX)

**Decides:** visual style, design system / component library, navigation pattern, light/dark mode, accessibility baseline, responsiveness.

- **Defaults:** a ready-made, accessible component library that matches the frontend stack; system font or one clean typeface; light + dark mode if cheap; responsive (mobile-first) for web; platform conventions (Human Interface Guidelines / Material) for native mobile.
- Accessibility baseline: semantic markup, keyboard navigation, sufficient contrast, labels on inputs (aim for WCAG 2.1 AA).
- Record: 3–5 adjectives for the feel, primary color direction, navigation pattern (tab bar, sidebar, top nav), key screens.

**Signals:** existing brand → use its colors/fonts; data-heavy tool → dense tables, keyboard shortcuts; consumer/social → more visual, motion, onboarding.

## 3. System design

**Decides:** the big-picture behavior: main components and how data flows between them; synchronous vs. background work; real-time; offline; consistency needs.

- **Defaults (Small/Medium):** client ↔ one backend/API ↔ one primary database, plus object storage for files. Slow or failure-prone work (emails, image processing, AI calls, webhooks) goes to a **background job queue**, not the request path.
- Real-time only if a feature needs it (chat, live collaboration, live dashboards): use the platform's real-time feature or WebSockets/SSE.
- Offline only if the user asked for it: local database on the device + sync strategy (decide conflict handling: last-write-wins vs. merge).

**Signals:** heavy integrations → webhook handling + idempotency; financial data → strong consistency, transactions, audit trail.

## 4. System architecture

**Decides:** how the code and deployables are structured.

- **Default:** a **modular monolith**: one deployable backend, organized by feature/domain modules. One repo (a monorepo if there are web + mobile + shared code).
- **Backend-as-a-Service** (managed Postgres + auth + storage + functions in one product) is a good default for Beginners and small solo projects: less code to own.
- Microservices only for Large scale *and* multiple teams. Don't recommend them to small teams.
- Put business logic in the backend (or database policies with BaaS), never only in the client.

## 5. Frontend

**Decides:** framework, rendering strategy, styling, state/data fetching, forms, mobile approach.

- **Web default:** React with a full-stack framework (e.g. Next.js) or a Vite SPA for app-only dashboards; TypeScript; utility-first CSS or the component library's styling; a data-fetching/cache library for server state; schema-validated forms.
- **SEO / content sites:** server rendering or static generation.
- **Mobile default (iOS + Android, one codebase):** React Native with Expo (pairs well with a React/TS web app) or Flutter. **Native** (Swift/SwiftUI, Kotlin/Compose) if only one platform, heavy platform APIs, or top-tier performance/feel matters.
- **Desktop:** web tech in a desktop shell (Tauri – lighter; Electron – most mature) or native if platform-specific.
- **Web + mobile:** share TypeScript types and API client; consider a monorepo.

**Signals:** Experienced user with a preferred framework → use theirs; existing code → keep it (Detected).

## 6. API & backend logic

**Decides:** backend language/framework, API style, validation, background jobs, integrations.

- **Defaults:** same language as the frontend where possible (TypeScript on Node) to share types. API style: **REST + JSON** with an OpenAPI description, or type-safe RPC (e.g. tRPC/server actions) when one TS team owns both sides. **GraphQL** only for many clients with varied data needs.
- Validate every input at the boundary with a schema library. Consistent error format.
- Background jobs: a managed queue or the platform's job/cron feature.
- Webhooks in/out: verify signatures, make handlers idempotent, retry with backoff.
- Python (FastAPI/Django) is a good default when the app is data/AI heavy or the user knows Python; Go for high-throughput services and Experienced teams.

## 7. Databases & storage

**Decides:** primary database, ORM/query layer, migrations, file storage, backups.

- **Default:** **PostgreSQL**, managed (it covers relational data, JSON, full-text search, and with extensions even vectors). Use migrations from day one; an ORM/query builder with typed results.
- **SQLite** (or a hosted SQLite) for very small, single-server or local-first apps; on-device for mobile offline.
- Document DB (e.g. Firestore/MongoDB) when the data is naturally document-shaped and the user is in that ecosystem; warn about harder relational queries later.
- Files/images/video → **object storage** (S3-compatible), never in the database; serve through a CDN.
- Backups: provider's automated daily backups + point-in-time recovery when data matters. Note the restore procedure.

## 8. Auth & permissions

**Decides:** sign-in methods, auth provider, session model, roles/permissions, multi-tenancy.

- **Default:** a managed auth provider or the BaaS's built-in auth. Don't build password storage yourself. Methods: email magic link or email+password plus one or two social logins that match the audience (Google; Sign in with Apple is **required** on iOS if any other social login is offered).
- Permissions: start with **role-based** (user/admin, or owner/member/viewer for teams). Enforce on the server (or row-level security in the DB), never only in the UI.
- Teams/B2B: organizations + memberships from day one; enterprise SSO (SAML/OIDC) only when selling to enterprises.
- Mobile: native auth flows, tokens in secure storage (Keychain/Keystore).

## 9. Hosting & cloud

**Decides:** where frontend, backend, database, jobs and files run; regions; environments.

- **Defaults (Small/Medium):** a managed platform (PaaS) for web/backend, managed database, managed object storage. Avoid self-managed servers and Kubernetes for small teams.
- Environments: **production + preview/staging**, with preview deploys per pull request where the platform supports it.
- Mobile: App Store / Google Play via the platform's build service (e.g. EAS for Expo, Xcode Cloud / Fastlane for native).
- Region: closest to most users; mind data-residency (e.g. EU data for EU users).
- Large / compliance heavy → a major cloud (AWS/GCP/Azure) with infrastructure as code.

## 10. CI/CD & version control

**Decides:** git hosting & branching, CI checks, deploy pipeline, release process.

- **Defaults:** GitHub; trunk-based (short-lived feature branches → pull request → main); CI on every PR: install, lint, type-check, test, build; auto-deploy main to production, PRs to preview environments.
- Conventional commits optional; protect `main`; dependency updates bot (Dependabot/Renovate).
- Mobile: CI builds + over-the-air updates where allowed; store releases via a scripted pipeline.
- Database migrations run as a deploy step, not by hand.

## 11. Security

**Decides:** secrets handling, data protection, dependency hygiene, common web risks, compliance basics.

- **Baseline (always):** HTTPS everywhere; secrets in the platform's secret manager / env vars, never in git; validate and sanitize input; parameterized queries (ORM); secure cookies (HttpOnly, Secure, SameSite) or short-lived tokens; CORS locked to known origins; security headers (CSP, HSTS); dependency scanning in CI; least-privilege DB/API keys; encrypted backups.
- Personal data → privacy policy, data export/delete flow (GDPR/CCPA), data minimization.
- Sensitive (health/finance/children's) data → flag it clearly: encryption at rest, audit logs, compliance review (HIPAA/PCI/COPPA as relevant). Payments: never touch raw card data, use the payment provider's hosted checkout/elements.

## 12. Rate limiting

**Decides:** what is limited, where, and how.

- **Defaults:** rate limit auth endpoints (login, sign-up, password reset, magic-link send) and any expensive or abuse-prone endpoints (AI calls, uploads, emails, search). Per IP for anonymous traffic, per user/API key when signed in. Return 429 with Retry-After.
- Where: the platform/edge (WAF or provider rate limiting) or middleware backed by a shared store (e.g. Redis) once there is more than one server.
- Add bot protection (CAPTCHA/turnstile-style challenge) on public forms if spam is likely.

## 13. Caching & CDN

**Decides:** static asset delivery, HTTP caching, application caching.

- **Defaults:** serve static assets and user files through a **CDN** (usually included in the hosting platform); long cache lifetimes for hashed assets; cache public pages at the edge (static generation / revalidation).
- App-level cache (Redis or in-memory) only when there is a measured hot path: expensive queries, third-party API results, sessions/rate-limit counters.
- Decide invalidation up front: time-based (TTL) by default; event-based for data that must be fresh.
- Mobile: client-side cache for offline/fast startup (the data-fetching library's cache or local DB).

## 14. Error tracking & logs

**Decides:** crash/error reporting, log format and storage, what is never logged.

- **Defaults:** an error-tracking service (e.g. Sentry) on frontend, backend and mobile, with release tracking and source maps; structured (JSON) logs with request IDs; the hosting platform's log viewer, or a log service once there are several services.
- Never log secrets, passwords, tokens or full payment/health data; scrub PII.
- Retention: 7–30 days for logs is fine at small scale.

## 15. Monitoring & alerts

**Decides:** what is measured, who gets alerted, where.

- **Defaults (Small):** uptime checks on the main URL and API health endpoint; error-rate alerts from the error tracker; platform metrics (CPU/memory/DB). Alerts to email or a chat channel.
- **Medium+:** key metrics dashboard (latency p95, error rate, throughput, queue depth, DB connections), alert thresholds, a public or internal status page, OpenTelemetry tracing when there are multiple services.
- Business metrics (sign-ups, payments) belong in analytics (see Product extras), but alert on sudden drops.

## 16. Testing

**Decides:** test types, tools, what is required before merge.

- **Defaults:** unit tests for business logic; integration tests for the API with a real (containerized or branch) database; a few end-to-end tests for the critical flows (sign up, the core action, payment) with a browser-automation tool; type-checking and linting in CI.
- Mobile: unit tests + a small set of UI/E2E tests on the main flows.
- Beginner: start with E2E tests of the main flow + type checking; add unit tests as logic grows.
- Rule of thumb: tests must pass in CI before merge; every bug fix adds a test.

## 17. Scaling

**Decides:** the plan for growth: what scales first and the triggers.

- **Defaults:** stateless app servers (scale horizontally on the platform); a managed DB that can scale up; indexes on every foreign key and frequent filter; pagination everywhere; background jobs for heavy work; CDN for static/media.
- Write down the **next step and its trigger**, e.g. "add read replica when DB CPU > 70% sustained", "add Redis cache when p95 > 500 ms on X", "split service Y out when a separate team owns it".
- Don't pre-build for Large scale when the target is Small: note the path instead.

## 18. Product extras

Only include what the app needs; mark the rest "not needed".

- **Payments:** a payment provider's hosted checkout + webhooks (e.g. Stripe); in-app purchases are **required** for digital goods in iOS/Android apps (consider a cross-platform subscription service). Marketplaces → the provider's connected-accounts product for payouts.
- **Email:** a transactional email service with a verified domain (SPF/DKIM/DMARC); templates in code.
- **Push notifications:** the platform push services (APNs/FCM), via the mobile framework's notification service; web push optional.
- **Search:** Postgres full-text search to start; a dedicated search engine when relevance/typo-tolerance/facets matter.
- **File uploads:** direct-to-object-storage with pre-signed URLs; size/type limits; image resizing in a background job or via an image CDN.
- **Analytics:** privacy-friendly product analytics; define 3–5 key events (sign-up, activation action, retention action, payment).
- **Feature flags / admin panel:** a simple admin view for support tasks; feature flags when shipping to many users.

## 19. AI / LLM features (only if the app uses AI)

**Decides:** model/provider, where calls happen, prompt management, cost and safety controls, evaluation.

- **Defaults:** call the model **from the backend only** (never ship API keys to clients); stream responses for chat-like UX; store prompts in code (versioned) with clear templates; use structured outputs / tool calling for anything parsed by code.
- Cost control: per-user rate limits and quotas, cache repeated results, use the smallest model that passes your quality bar, set max tokens, track cost per user/feature.
- Retrieval (RAG) only if the app answers from the user's own data: start with Postgres + a vector extension before adding a separate vector database.
- Safety & quality: input/output moderation where users generate content; a small evaluation set of real examples run in CI or before prompt changes; log prompts/outputs (without PII) for debugging.
- Background jobs for long-running AI work; timeouts and retries with backoff.

## 20. Cost & budget

**Decides:** expected monthly cost now and at the scale target; budget guardrails.

- Give a rough monthly estimate for **MVP** and for the **scale target**, per major item (hosting, DB, storage/CDN, auth, email, error tracking, AI usage, app-store fees). Use ranges and say they are estimates; don't invent exact prices.
- Point out free tiers that fit the MVP, and which choice becomes expensive first as usage grows.
- Guardrails: billing alerts on every provider, spending caps where available, usage quotas on AI and other metered APIs.
- Note fixed fees: Apple Developer Program (yearly), Google Play (one-time), domain name.
