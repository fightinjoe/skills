---
name: app-architect
description: Define the technical architecture for a new app after the product brief is written. Trigger when a user wants to make tech stack decisions, decide on frontend/backend shape, or produce an ARCH.md. Should be run after app-init and after any critical prototypes are done.
---

# App Architect

A skill for translating a product brief into a concrete technical architecture. The output is `docs/ARCH.md` — a living document that defines the technical shape of the app and constrains all future feature development.

This is not a rubber-stamp exercise. Your job is to present real trade-offs, push back on mismatches between the product's needs and the proposed approach, and help the user make decisions they'll still be comfortable with six months in.

---

## Philosophy

Architecture decisions have long tails. A wrong call early (client-only when you need server-side auth, a heavy framework when you need a static site) costs far more to undo later than to get right now.

- Present options honestly — don't just validate what the user seems to want
- Name the trade-offs explicitly: what does each option make easy, and what does it make hard?
- Flag mismatches: if the product brief implies something the proposed stack makes painful, say so
- Distinguish *decided* from *deferred* — unknown is fine, but it must be named
- Prefer boring, proven technology unless there's a specific reason not to

---

## Before You Start

Read `docs/BRIEF.md`. Before asking anything, internalize:
- What the product does and who it's for
- The platform implied by the brief (web? mobile? CLI?)
- Any technical notes or constraints already stated
- The features and phases — these shape what the architecture must support

If `docs/BRIEF.md` doesn't exist, ask the user to run `app-init` first.

Then open the interview. Don't dump all the questions at once — work through one dimension at a time, in a natural conversation.

---

## Interview Dimensions

Work through each of these. For each, briefly describe the relevant options and their trade-offs *before* asking the user what they want. If the brief already answers a question clearly, confirm it rather than re-litigating it.

---

### 1. Platform & Reach

Start here — it gates everything else.

**Options to discuss:**
- **Web only** — broadest reach, easiest deployment, no app store friction. Right for most things.
- **Mobile (native or cross-platform)** — React Native/Expo for cross-platform, Swift/Kotlin for native. Required if you need device APIs (camera, push notifications, sensors) or a first-class mobile experience.
- **Desktop** — Electron or Tauri for cross-platform desktop. Niche; mostly right for tools that need local file system access or offline-first.
- **CLI** — right for developer tools or automation. No UI framework needed.
- **Multiple** — e.g. web + mobile. Increases complexity; make sure both are needed at launch vs. later.

Ask: does the brief's target user and use case demand a specific platform, or is web sufficient?

---

### 2. Frontend Shape

Only relevant if there's a UI.

**Options to discuss:**
- **SPA (client-rendered)** — React, Vue, Svelte. Fast after first load, simple deployment (static files), but poor SEO and slower initial load. Right for authenticated apps where SEO doesn't matter.
- **SSR / full-stack framework** — Next.js, Remix, SvelteKit, Nuxt. Server renders HTML; good for SEO, faster first load, can colocate backend logic. Requires a server (or edge runtime). Right for public-facing content or when you want one framework for both.
- **Static site** — Astro, plain HTML, 11ty. Zero JS by default, fastest possible load, trivially deployable. Right if the app is mostly content with little interactivity.
- **Native mobile UI** — React Native, Expo, or platform-native. Required for app store distribution.

Flag if the brief mentions SEO, public landing pages, or content — those push toward SSR or static, not SPA.

---

### 3. Backend Shape

**Options to discuss:**
- **Client-only (no backend)** — all logic runs in the browser or device. Simplest to deploy. Only works if there's no sensitive logic, no server-side auth, and no data that needs to be shared across devices/users.
- **Backend-as-a-Service (BaaS)** — Supabase, Firebase, Convex. Database + auth + storage + realtime, hosted. Fast to start, less control. Right when you don't want to write a backend but need persistence and auth.
- **Serverless functions** — Vercel Functions, Cloudflare Workers, AWS Lambda. Good for lightweight API needs alongside a frontend framework. Scales automatically, but cold starts and statelessness can be limiting.
- **Dedicated server** — Node/Express, Fastify, Hono, Python/FastAPI, Go. Full control, persistent connections, background jobs, websockets. More ops overhead. Right for complex business logic, real-time features, or heavy processing.

Flag if the brief implies: user accounts, cross-device sync, payments, or any server-side secrets — these rule out client-only.

---

### 4. Data & Storage

**Options to discuss:**
- **Local only** — localStorage, IndexedDB, SQLite (via WASM or native). No server needed; data stays on device. Right for personal tools or offline-first apps.
- **Hosted relational DB** — Postgres (via Supabase, Neon, Railway, RDS). SQL, strong consistency, good tooling. Default choice for most apps with structured data.
- **Hosted document DB** — MongoDB Atlas, Firestore. Flexible schema, good for hierarchical or variable-shape data. Often overkill when relational works fine.
- **BaaS-integrated storage** — Supabase or Firebase include their own DB. Convenient if you're already using them for auth.
- **File/blob storage** — S3, Cloudflare R2, Supabase Storage. For user-uploaded files, images, etc. Usually paired with one of the above.

Ask about: how structured is the data? Does it need to sync across devices? Is there user-generated content (files, images)?

---

### 5. Auth

**Options to discuss:**
- **None** — no user accounts. Valid for single-user tools or anonymous apps.
- **Third-party auth service** — Clerk, Auth0, WorkOS. Fast to integrate, handles all edge cases (MFA, SSO, sessions). Costs money at scale but saves significant dev time.
- **BaaS auth** — Supabase Auth, Firebase Auth. Built-in if you're already using those platforms. Covers most cases well.
- **Roll your own** — sessions + bcrypt, or JWTs. Full control, but auth is a security-sensitive area where rolling your own is often a mistake. Only recommend if there's a specific reason.
- **Magic link / passwordless** — Resend + a token table, or via a third-party. Good UX for low-friction apps.
- **OAuth only** — Sign in with Google/GitHub/etc. Fine if your user base reliably has those accounts.

Flag if the brief implies: teams, roles/permissions, enterprise customers — those push toward a service that handles the complexity.

---

### 6. Deployment & Hosting

**Options to discuss:**
- **Vercel / Netlify** — zero-config deployment for frontend-heavy or Next.js/framework apps. Excellent DX. Vercel is the natural home for Next.js.
- **Fly.io / Railway / Render** — for apps that need a real server, persistent processes, or background jobs. Good balance of simplicity and control.
- **Cloudflare Pages + Workers** — edge-native. Very fast globally, generous free tier. Good for lightweight full-stack apps. Some constraints (no long-running processes).
- **App stores (iOS/Android)** — required for native mobile. Plan for review times and update latency.
- **Self-hosted / VPS** — full control, lowest cost at scale. Higher ops burden. Right if there are compliance/data residency requirements.

Ask about: expected traffic, latency requirements, budget, and whether any data residency or compliance constraints apply.

---

### 7. Key Integrations

Ask what third-party services the app will depend on. Common ones to probe:
- **Payments** — Stripe is the default. Ask if there are marketplace/platform payment needs (Stripe Connect) vs. simple checkout.
- **Email** — transactional (Resend, Postmark) vs. marketing (Loops, Mailchimp). Different tools.
- **AI / LLMs** — Claude API, OpenAI. Ask what role AI plays — is it core or peripheral?
- **Storage** — user uploads? Need S3/R2.
- **Analytics** — Posthog, Plausible, Mixpanel.
- **Error tracking** — Sentry.

Only log integrations that are confirmed needed. Don't speculate.

---

### 8. Cross-Cutting Constraints

Before wrapping up, ask:
- Are there any performance requirements that should shape architecture choices? (e.g. sub-100ms response times, offline support, real-time collaboration)
- Any compliance or security requirements? (GDPR, HIPAA, SOC 2)
- Any hard budget constraints on infrastructure?
- Is this a solo project, small team, or larger org? (affects how much to optimize for DX vs. scalability)

---

## Knowing When to Move On

Move to writing ARCH.md when:
- Platform, frontend shape, and backend shape are decided (or explicitly deferred with rationale)
- Data storage and auth approach are decided
- Deployment target is at least directionally clear
- Key integrations are named

Not everything needs to be decided. Deferred decisions are valid — but they must be named and reasoned about in ARCH.md.

Summarize what you've captured before writing: *"Here's what I've got — does this match your understanding before I write it up?"*

---

## Write docs/ARCH.md

```markdown
# [Product Name] — Architecture

> One-line summary of the technical shape: e.g. "SSR web app with Supabase backend, deployed on Vercel."

## Status
**As of:** [date]
**Brief:** [link or note: see docs/BRIEF.md]

## Platform
[What platform(s) this runs on and why.]

## Frontend
**Approach:** [SPA / SSR / Static / Native / None]
**Framework:** [e.g. Next.js, SvelteKit, React Native + Expo]
**Why:** [One or two sentences on the decision rationale]

## Backend
**Approach:** [Client-only / BaaS / Serverless / Dedicated server]
**Runtime/Framework:** [e.g. Supabase, Vercel Functions + Hono, Fly.io + Fastify]
**Why:** [Rationale]

## Data & Storage
**Primary store:** [e.g. Postgres via Supabase, Firestore, SQLite]
**Schema sketch:** [Optional — key entities and their relationships if known]
**File/blob storage:** [if applicable]
**Why:** [Rationale]

## Auth
**Approach:** [e.g. Clerk, Supabase Auth, none]
**Mechanism:** [e.g. magic link, OAuth, password]
**Why:** [Rationale]

## Deployment
**Target:** [e.g. Vercel, Fly.io, App Store + Play Store]
**Environments:** [prod, staging, local dev — any notable differences]
**Why:** [Rationale]

## Key Integrations
- **[Service]** — [what it's used for]
- ...

## Constraints
[Things every feature and future PRD must respect. E.g.: "All user data must stay in EU region", "App must function offline for core features", "No background jobs — serverless only"]

## Deferred Decisions
[Architecture questions that are not yet resolved. For each, note why it's deferred and what would cause it to be decided.]
- **[Decision]** — Deferred because: [reason]. Will be resolved when: [trigger].

## What This Rules Out
[Honest accounting of what this architecture makes hard or impossible. E.g.: "Real-time collaboration would require rearchitecting the backend", "Native mobile features (push notifications, camera) are not supported by this web-only approach"]
```

After writing, share it and ask: *"Does this match your understanding? Anything wrong, missing, or that you want to reconsider?"*

---

## Iterative Refinement

As with the brief, don't treat the first draft as final. For each round:
1. Update ARCH.md to reflect changes
2. Flag if any change creates a conflict with BRIEF.md
3. Note any new deferred decisions that emerged

Keep refining until the user is satisfied and all major dimensions are either decided or explicitly deferred.

---

## Finishing Up

After ARCH.md is approved, summarize:
- The core technical shape in one sentence
- Any deferred decisions the user should keep an eye on
- What to do next: write feature PRDs, which should reference ARCH.md for stack and constraints

Remind them: feature PRDs should not re-litigate architecture decisions already captured here. If a feature requires departing from the architecture, that should be noted explicitly in the PRD and the architecture updated.
