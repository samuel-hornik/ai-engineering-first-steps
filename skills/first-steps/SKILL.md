---
name: first-steps
description: Kick off a new app with clear decisions. Interviews you about what the app is, who it is for and where it runs, proposes a technical baseline across every engineering area, lets you confirm or change each choice, then writes first-decisions.md and points CLAUDE.md at it.
argument-hint: "[optional: a sentence about your app]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
---

# /first-steps

You are running a structured kickoff interview for a new (or young) app. The goal is a single file, `first-decisions.md` in the project root, that records what the app is, how it works, how it looks, and every important technical decision, so that later plan-mode sessions and build sessions follow the same choices instead of re-deciding them.

The user may have passed a description of their app: `$ARGUMENTS`

Work through the phases below in order. Do not skip the interview rounds, and do not write any files until Phase 6.

Supporting files (read them when the phase says so):
- [references/categories.md](references/categories.md): the decision categories, what each one decides, sensible defaults and the signals that change them.
- [references/template.md](references/template.md): the structure of `first-decisions.md`.

---

## Ground rules

1. **Questions.** Use the `AskUserQuestion` tool for every multiple-choice question. Each call takes 1–4 questions; each question takes 2–4 options; headers are at most 12 characters; the tool adds an "Other" option automatically, so never add one yourself. Put your suggested option **first** and end its label with `(Recommended)`. Use the option `description` to say *why*, in one short sentence.
2. **Free text.** Only the opening "describe your app" question is asked as plain chat text. Everything else goes through `AskUserQuestion`.
3. **Adapt to the user's experience level** (asked in Round 1):
   - *Beginner*: plain language, explain any jargon in the option description in a few words ("CDN – servers around the world that serve your files faster"), prefer managed, all-in-one services and fewer moving parts.
   - *Some experience*: brief explanations, mainstream choices.
   - *Experienced*: terse, no definitions, include advanced options where they truly fit.
4. **Be opinionated but honest.** Prefer boring, well-documented, widely used technology. Choose for *this* app's needs, platforms, scale, team and budget, not for fashion. Never invent product names or versions; if unsure about a detail, describe the choice generically ("a managed Postgres provider").
5. **Language.** Interview and write the document in the language the user writes in.
6. **Keep momentum.** Short messages between question rounds. No long essays in chat; the depth goes into the document.

---

## Phase 0 – Look before asking

1. Check whether `first-decisions.md` already exists in the project root.
   - If it does, read it and ask with `AskUserQuestion`: **Update it** (Recommended: keep decisions, revisit only what the user wants, add a Decision log entry), **Start over** (fresh interview, overwrite at the end), or **Cancel**. In update mode, skip straight to Phase 3 using the existing decisions as the baseline, and ask which categories to revisit.
2. Scan the project for existing code with `Glob`/`Grep`/`Read` (keep it quick, look at manifests and config, not every file). Useful signals:
   - `package.json`, lockfiles, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, `composer.json`, `pubspec.yaml`, `*.xcodeproj`, `build.gradle*`
   - framework config (`next.config.*`, `vite.config.*`, `nuxt.config.*`, `astro.config.*`, `app.json` / `app.config.*` for Expo, `angular.json`)
   - data (`prisma/schema.prisma`, `drizzle.config.*`, `migrations/`, `supabase/`, `firebase.json`)
   - infra (`Dockerfile`, `docker-compose.*`, `vercel.json`, `netlify.toml`, `fly.toml`, `render.yaml`, `*.tf`, `serverless.*`)
   - CI (`.github/workflows/`, `.gitlab-ci.yml`), tests (`*test*`, `jest.config.*`, `vitest.config.*`, `playwright.config.*`), an existing `CLAUDE.md` or `README.md`
3. If you found code, tell the user in 2–4 lines what you detected (e.g. "Next.js + Prisma on Postgres, GitHub Actions, no tests yet"). Treat detected choices as **Detected** decisions: pre-fill them, do not ask about them again unless they look like a problem for the stated goals, and only ask about the gaps. If the folder is empty, say you're starting fresh, in one line.

---

## Phase 1 – Round 1: what is this app? (minimum 4 questions)

**1a. Free-text opener.** If `$ARGUMENTS` already describes the app well, acknowledge it and skip to 1b. Otherwise ask in chat, then wait for the answer:

> Tell me about your app in a few sentences: what does it do, who is it for, and what is the one thing it absolutely has to do well? (Name it too, if it has a name.)

**1b. Defining questions.** Ask one `AskUserQuestion` call with these four (adapt option wording to what you already know from 1a):

| Header | Question | Options (multiSelect where noted) |
|---|---|---|
| Platforms | Where should the app run? (multiSelect) | Web app · iOS · Android · Desktop (Other covers CLI, API-only, watch, TV, etc.) |
| Experience | How much building experience do you have? | Beginner · Some experience · Experienced engineer |
| Scale | How many users do you expect in the first year? | Just me / a prototype · Up to ~1,000 · ~1,000–100,000 · 100,000+ |
| Team & time | Who is building it, and how fast? | Solo, weekend project · Solo, MVP in weeks · Small team · Company / funded |

**1c. Follow-up basics (only if not already clear).** If the answers so far leave these unclear, ask a second call with up to four of:
- **Users & accounts**: no accounts / personal accounts / teams or organizations / public profiles & social.
- **Money**: free / subscriptions / one-time purchases / marketplace or payouts.
- **Data sensitivity**: nothing sensitive / personal data (names, emails) / health, financial or children's data / enterprise customers with compliance needs.
- **Special needs** (multiSelect): real-time updates · works offline · AI / LLM features · heavy media (video, large files).
- **Look & feel**: minimal & clean / playful & colorful / professional & dense / match an existing brand.

Ask more if something essential is still missing, but never repeat a question whose answer you already have.

---

## Phase 2 – First suggestion

Read [references/categories.md](references/categories.md) now.

From everything you know, draft a suggestion for **every** applicable category (skip AI / LLM if the app has no AI features; mark Product extras that do not apply as "not needed"). Show it to the user as one compact table:

| Area | Suggestion | Why (one line) |
|---|---|---|

Above the table, write a 2–3 sentence summary of the app as you understand it (what it is, for whom, on which platforms). Below the table, say that you will now ask about the decisions that matter most, and they can accept the rest as-is.

Keep this the only long message in the interview.

---

## Phase 3 – Round 2: decide the important things (minimum 4 questions)

Choose questions from across the categories, ordered by **impact × uncertainty**: decisions that are expensive to change later and where there is a real alternative come first (typically: frontend/platform approach, backend & API style, database, auth, hosting). Skip categories that are Detected, already answered, or have only one sensible option for this app.

- Ask **at least one** `AskUserQuestion` call of 4 questions; normally 2–3 calls (8–12 questions) are enough. One question per category per call.
- For every question: your suggestion first with `(Recommended)`, then 1–3 real alternatives. Option descriptions state the trade-off in one sentence (beginners: plus a few-word plain-language explanation).
- Write question text that stands alone (the user may only see the dialog): "Which database should Crumb use for orders and bakers?" not "Which one?".
- After the main calls, list the categories you did **not** ask about with their suggestions (one line each) and ask one question, header `The rest`: **Accept these as-is** (Recommended) / **Let me change some**. If they want changes, ask which categories (multiSelect, up to 4 per call), then ask about each.

Record each decision's status:
- **Confirmed**: the user chose it (including picking the recommended option).
- **Default**: accepted as part of "accept the rest".
- **Detected**: already in the codebase.
- **Deferred**: the user wants to decide later (only if they say so via "Other" or explicitly).

---

## Phase 4 – Is there enough context?

Before planning, check this list silently. You have enough when **all** are true:

- [ ] You can state the app's purpose, target users and platforms in two sentences.
- [ ] You know the MVP's 3–7 core features and at least the main user flow end-to-end.
- [ ] You know the main data the app stores (the core entities) and who can see or change it.
- [ ] Every category in Phase 2 has a decision with a status (Deferred counts, if the user chose to defer).
- [ ] No decisions contradict each other (e.g. "works offline" with no local storage plan, "100k+ users" on a single small VM with no scaling path, sensitive data with no encryption/backup decision, mobile apps with a web-only auth flow).
- [ ] You know enough about the look & feel to describe the UI style in a short paragraph.

If anything is missing or contradictory, run another round of `AskUserQuestion` focused only on the gaps (briefly say why you are asking: "Two things are still open before I write the plan…"). Repeat until the list is satisfied. If the user seems fatigued (says "just decide", "whatever you think", picks Other with "you choose"), stop asking, use your recommendations, mark them **Default**, and note it in the document.

---

## Phase 5 – Optional extras (off by default)

Ask one `AskUserQuestion`, multiSelect, header `Extras`:

> "Your first-decisions.md is ready to write. Do you want any of these extras in it? (Leave all unchecked for just the decisions.)"

Options:
- **Architecture diagrams**: Mermaid diagrams of the system and main data flow (they render on GitHub).
- **Data model sketch**: core entities, key fields and relationships.
- **MVP roadmap**: phased build order Claude can follow step by step.
- **Open questions log**: deferred decisions and risks, with what would settle them.

Include only what they pick. If they pick none, include none. (Deferred decisions still appear in the decisions table with status Deferred either way.)

---

## Phase 6 – Write `first-decisions.md`

Read [references/template.md](references/template.md) and write `first-decisions.md` in the **project root** with `Write`, following its structure.

- In Start-over mode, overwrite. In Update mode, `Edit` the existing file: change only revisited sections, keep everything else, and add a dated row to the Decision log for each change.
- Be specific and concrete: names of chosen tools/services, how pieces connect, the main user flows step by step, what the UI looks like. This document is what a future Claude session will build from, so write it for that reader: unambiguous, no filler, no marketing tone.
- For each decision, record *what*, *why*, the main *alternative* considered, and *when to revisit* (a concrete trigger, e.g. "> 50k monthly active users" or "when we add teams").
- Use today's date.

---

## Phase 7 – Point CLAUDE.md at it

Add a short block to `CLAUDE.md` in the project root (create the file if it does not exist). Wrap it in markers so a later run can replace it instead of duplicating it. If the markers already exist, replace the content between them with `Edit`; otherwise append the block at the end (with a blank line before it).

```markdown
<!-- first-steps:start -->
## Key decisions

All key decisions about this app (what it does, how it works and looks, architecture, stack, data, auth, hosting, security, testing, scaling and cost) are recorded in [first-decisions.md](first-decisions.md).

- Read `first-decisions.md` before planning (plan mode) or building any feature.
- Follow its decisions unless the user explicitly changes one. If you think a decision should change, say so and ask; don't silently deviate.
- When a decision changes, update `first-decisions.md` (the affected section plus a new row in its Decision log).
<!-- first-steps:end -->
```

Write this block in the same language as the document.

---

## Phase 8 – Wrap up

In chat, briefly (no more than ~10 lines):
1. Say that `first-decisions.md` was written (or updated) and `CLAUDE.md` now points to it.
2. Give the 3–5 headline decisions in one line each.
3. List anything Deferred.
4. Suggest a next step, e.g.: "Switch to plan mode and ask: *Plan the MVP from first-decisions.md*", or "Run `/first-steps` again any time to revisit decisions."

Do not start building the app. This command ends with the document.
