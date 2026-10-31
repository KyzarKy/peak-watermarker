# AGENTS.md

Instructions for any coding agent working in this repo. Claude Code reads it through
CLAUDE.md. Humans are welcome to read it too: it's also the working agreement between Ky and
the agents.

## What this is

peak-watermarker (a working name) is a free, offline photo watermarking app for Mom's
iPhone, with Android later. She picks photos she has already taken, places a text mark once,
and saves watermarked copies of all of them. Or she lets the app pick a spot for each photo,
on the phone itself. Originals are never touched, and nothing leaves the phone. It's also one
of Ky Merritt's portfolio projects: a React Native app with a small native module.

## Status: planning

No application code until Ky signs off on a written plan (`docs/02-plan.md`, not written
yet). Read these first, in order:

1. [docs/00-kickoff-brief.md](docs/00-kickoff-brief.md): the request and its constraints.
2. [docs/01-planning-log.md](docs/01-planning-log.md): decisions so far, with dates, and the
   open questions.
3. [docs/capabilities.md](docs/capabilities.md): what the app has to do, mapped to iOS and
   Android APIs.
4. `research/README.md`: the index of cited research (local only, see rule 6).

Planning is a dialogue. Ask three to five questions at a time, short enough to answer on a
phone. Challenge assumptions, research when it sharpens a decision, and record each decision
in the planning log with its date. Questions only Mom can answer are marked (Mom), and Ky
asks her.

## How we work

It follows the same working agreement as Fluent Prior, another of Ky's portfolio apps. Ky
confirmed that in round 1.

- Ky orchestrates and decides. Agents propose and implement.
- Decisions are written down before code. During planning they go in the planning log. Once
  `02-plan.md` is signed off, a change to a documented decision needs an ADR in
  `docs/decisions/` first.
- Small changes. Each commit message or PR description says what changed and why, how to
  verify it, and three questions an interviewer might ask, with short answers.
- Docs move with code. A change that contradicts a doc updates that doc in the same commit.
- `/triage` and depurple before a milestone closes.
- Git: push straight to `main` for uncontroversial changes. Ask first when a change needs
  Ky's call. Never force-push or rewrite pushed history. Never commit secrets.

## Rules that are easy to break by accident

1. **Nothing leaves the phone.** Photos, marks, and settings never touch the network. No
   analytics, crash reporting, accounts, or cloud ML. The one network use allowed is
   downloading app updates, and the app works fully offline without them.
2. **Originals are never modified or deleted.** The app only adds new copies.
3. **Full resolution by default.** Never downscale silently. A smaller export is opt-in.
4. **Mom stays anonymous.** Refer to her as "Mom" in code, docs, commits, and screenshots.
   Her name, her mark's text, her logo, and her photos never enter the repo. The planning
   log records the shape of her answers ("name and year, script font"), never the text.
   Fixtures use "© Your Name" and photos Ky took.
5. **The repo is public.** Every commit is visible. No secrets, no personal details, and
   nothing copied from `research/`.
6. **`research/` is local only.** It's gitignored, so never force-add it. Summarize sources
   there in your own words, with the URL and the date checked. Its README has the rules.
7. **Facts carry a source.** A claim about an API, an OS version, a price, or a limit gets a
   link and the date checked, either in the doc or in the research file it points to.
8. **No em dashes or en dashes** in anything Ky might send or publish, this repo included.
   Use a spaced hyphen ( - ) for an aside, and "to" for a range.

## Layout

Now:

```
AGENTS.md    these instructions (CLAUDE.md imports this file)
README.md    what the project is, for people landing on the repo
LICENSE      public to read; all rights reserved
docs/        brief, planning log, capabilities
research/    cited research, gitignored, local only
```

Planned once code starts (final in `02-plan.md`):

```
app/                        screens (Expo Router)
src/core/                   pure TypeScript: layout math, placement scoring, styles; unit-tested
modules/watermark-engine/   local Expo module: Swift (ImageIO, Core Image, Vision, PhotoKit), Kotlin
```

Native code handles decoding, compositing, metadata, saving, and Vision. TypeScript decides
where the mark goes and how it looks. `src/core` imports nothing from React, React
Native, or Expo.

## Machine notes

- Ky's machine runs Windows 11 and Node 24. PowerShell blocks `npm.ps1`, so use `npm.cmd`.
- There is no Mac. The iOS project can't be opened or compiled locally, and Swift compiles
  only in CI. Keep native changes small, and expect compile errors to show up there.
- Android tooling (Android Studio, a JDK, adb) isn't installed yet.
