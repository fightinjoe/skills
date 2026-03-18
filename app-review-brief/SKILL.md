---
name: app-review-brief
description: Revisit and update an existing BRIEF.md after prototyping. Trigger when the user says "I want to revisit my brief", "let's update the brief", or "I've finished a prototype and want to reassess". Reads the current BRIEF.md, interviews the user about what they learned from prototyping, and rewrites the brief in place to reflect new understanding.
---

# App Review Brief

A skill for updating a `BRIEF.md` after prototype work. Assumptions get validated or invalidated, open questions get answered or refined, and the brief is rewritten to reflect what's actually true now — not what was hoped at the start.

---

## Philosophy

The brief was written with imperfect information. Prototyping is how you get better information. This review is not a formality — it's where the brief earns its honesty.

- Don't preserve things that turned out to be wrong
- Don't add new scope just because something was interesting to build
- Promote assumptions to decisions when validated; demote them to risks or remove them when invalidated
- Close open questions that are now answered; surface new ones that emerged

---

## Phase 1: Read the Brief

Read the existing `docs/BRIEF.md`. Note:
- All open questions (especially 🔴 and 🟡)
- All assumptions
- The prototype map — what was planned and why

---

## Phase 2: Prototype Debrief Interview

Ask the user to walk through what they built and what they learned. Don't interrogate — let them tell the story, then probe.

Work through these dimensions:

**What got built**
- Which prototypes from the map did you complete?
- Were there any prototypes you started but didn't finish? Why?
- Did you build anything that wasn't in the prototype map?

**What you learned**
- For each completed prototype: what did you learn? Did it answer the question it was meant to answer?
- What surprised you?
- What confirmed what you already thought?

**Impact on open questions**
- Go through each 🔴 and 🟡 question from the brief. For each:
  - Is it answered? What's the answer?
  - Is it still open? Did it change shape?
  - Did it turn out not to matter?

**Impact on assumptions**
- For each assumption in the brief: validated, invalidated, or still unknown?
- Did any new assumptions surface during prototyping?

**Scope changes**
- Did anything you learned change what you think the app should do?
- Did you discover things you should *stop* planning to build?
- Did you discover things you *should* build that weren't in the brief?
- Did prototyping change how you'd phase the build? Does the phase breakdown still make sense?

**New open questions**
- What questions did prototyping raise that weren't there before?
- Is there anything you still need to build to understand?

As you listen, track:
- Questions to **close** (answered)
- Questions to **update** (still open but changed)
- Questions to **add** (newly surfaced)
- Assumptions to **promote** (validated → decision), **demote** (invalidated → removed or flagged as risk), or **keep**
- Scope to **add or remove**

---

## Phase 3: Summarize Before Rewriting

Before touching the file, tell the user what you're about to change:

- Questions being closed and what the answers were
- Assumptions being promoted, demoted, or removed
- Any scope changes
- New open questions being added
- Any sections that will change substantially

Ask: *"Does this match what you're thinking? Anything I missed?"*

---

## Phase 4: Rewrite BRIEF.md

Rewrite `docs/BRIEF.md` in place with all changes applied. Preserve the same structure and formatting. Specific rules:

- **Open Questions**: remove answered questions, update changed ones, add new ones with appropriate priority
- **Prototype Map**: mark completed prototypes `[x]`; remove entries for answered questions; add new entries for new 🔴/🟡 questions (as `- [ ]`)
- **Features & Phases**: update if scope or phasing changed based on what was learned
- **Assumptions**: promote validated ones to the relevant section as stated facts; remove invalidated ones or move to a **Risks** note if they matter; keep still-unknown ones
- **Goals / Non-Goals**: update if scope changed
- **Out of Scope**: move any newly deferred items here
- Do not add a changelog or revision history — git handles that

---

## Finishing Up

After rewriting, give the user a short summary:
- What changed and why
- Which open questions remain (especially 🔴)
- Whether there are new prototypes to build before the PRD can be written
- If all 🔴 questions are answered, suggest running `app-architect` next to define the tech stack before writing feature PRDs
