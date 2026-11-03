# Plan

**Status: signed off by Ky on 2026-11-03.** M0 is next. A change to anything documented here
needs an ADR in `docs/decisions/` first.

The reasons behind each choice are in the [planning log](01-planning-log.md), and the API
details are in [capabilities.md](capabilities.md). "D13" means decision 13 in the log, and
"C7" means capability 7.

## Calls Ky made at sign-off

The dialogue didn't settle these. Ky accepted each recommendation on 2026-11-03, and the rest
of this plan assumes them.

1. **Which Apple ID signs the app?** Recommendation: a new free Apple ID used only for this,
   which Ky creates. A user report says a fresh ID worked on iOS 27.2 where an existing one
   failed, and it keeps Mom's main account away from sideloading tools. Whether AltStore
   accepts an ID that differs from the one signed in on the phone is unverified, and M0
   tests it.
2. **What sends us to the $99 fallback?** Recommendation: AltStore and then Sideloadly both
   fail in M0 with the new Apple ID, or the app misses a renewal it should have made and
   stays stuck for more than a week after M0. Ky then chooses between paying $99 once and
   waiting for the tools to catch up.
3. **Does v1 handle originals that are only in iCloud?** Recommendation: only the minimum.
   She keeps originals on the phone (D15), so v1 keeps the immediate copy and a timeout, and
   a stuck photo lands in the skip-and-report list. C2's download progress and Retry button
   wait for a setting she'd actually use.
4. **When do the Android tools get installed?** Recommendation: right after M0 passes, since
   Android work is wasted if the iPhone route fails and Ky chooses to stop.

## Goal

Mom picks photos she already has on her iPhone, places a text mark once or lets the app pick
a spot per photo, and saves full-resolution copies. It's free, it works offline, nothing
leaves the phone, and originals are untouched.

**v1 is:** the text mark in the same relative spot across a batch, auto-placement with a
review grid, per-photo mark color and opacity, and a remove-location toggle (D11, D16). C1 to
C17 are the requirements, as narrowed by call 3 above.

**v1 is not:** logos, tiled marks, kept HDR, metadata copyright, an album, a web-size export,
or a public Android release. Those are L1 to L9.

## Stack

- Expo SDK 58 (React Native, New Architecture), with Expo Router and TypeScript (D9).
- One local Expo module in Swift and Kotlin for decoding, compositing, metadata, saving, and
  Vision (D9).
- Plain `StyleSheet` with `src/theme.ts`, plus a drift test against an exact-pinned
  @kyzarky/ui (D13).
- GitHub Actions builds the unsigned iPhone app. AltStore signs and renews it from AltServer
  on Ky's PC (D10). EAS Update ships JS changes.

Running it costs $0 (D2).

## Layout

```
app/                        screens (Expo Router)
src/core/                   pure TypeScript: layout math, placement scoring, styles; unit-tested
src/theme.ts                @kyzarky/ui's colors, checked by the drift test
modules/watermark-engine/   local Expo module: Swift (ImageIO, Core Image, Vision, PhotoKit), Kotlin
docs/decisions/             ADRs, from sign-off on
```

`src/core` imports nothing from React, React Native, or Expo. Values that feedback will tune
(sizes, insets, scoring weights) live there as data, so changing them is a JS change (C15).

## Milestones

Each one closes with `/triage` and depurple. Every commit follows the working agreement in
AGENTS.md.

### M0: prove the install route

This comes first because the route is the biggest risk. See uncertainty 1 in the log: no
report shows AltStore with Windows AltServer working on iOS 27.2.

1. **Smoke test, with no project code.** Ky installs iTunes and iCloud from Apple's site and
   the current AltServer, installs AltStore on her phone with the new Apple ID, and turns on
   Developer Mode. Then he sideloads any small app. If that fails, he repeats it with
   Sideloadly.
2. **A blank Expo app with a one-function local module.** GitHub Actions builds it unsigned.
3. **Install it on her phone.** It opens and calls the Swift function.
4. **Metro connects** over home Wi-Fi, and **EAS Update delivers** a JS change.
5. **Survive a renewal.** The app opens after one 7-day renewal with a stored value intact.

**Done when** step 5 passes. Step 1 takes an evening, so the route is known to be workable or
not before any project code exists. If it fails, call 2 applies.

### M1: the core loop

Install the Android tools first (call 4), so the daily loop runs on Ky's phone (D18).

- Pick many photos in order, without the camera (C1), and copy each one in at once (C2).
- Decode whichever format each photo is (C3, D14).
- Style a clean sans-serif text mark (C4, D16), and save one style (C14).
- Place it with the 9-point grid or a drag, sized relative to each photo (C5, C6).
- Render at full resolution, one photo at a time (C8).
- Save copies in the same format with their metadata and capture date (C9, C11, D17).
- Add the remove-location toggle, off by default (C10, D17).
- Show progress, cancel, and skip-and-report (C12).
- Add `src/theme.ts` and the drift test (D13), with `"userInterfaceStyle": "automatic"`.

**CI (D21):** the macOS runner watermarks sample photos and checks size, metadata, and
orientation, then saves the results to look at. Ky shoots the samples on her phone, since
his Android can't produce iPhone formats. They hold nothing personal and no real location,
because the repo is public. Large ones go in a GitHub release asset instead of git. How much
of this can run without a device, given that PhotoKit saving needs a library, is unverified,
and the fallback is checking saves by hand.

Android exercises the UI and `src/core` every day. It doesn't prove the Swift half, so each
Swift change also needs a passing CI run and a milestone build on her phone.

**Done when** Ky runs a same-spot batch of 50 of his own photos on her phone in airplane mode,
and the copies keep their format, resolution, and capture date (C13).

### M2: auto-placement, which completes v1

- The module supplies a thumbnail, Vision saliency, and face boxes. `src/core` scores the
  candidates (C7).
- A good spot is off faces and the subject, easy to read, and in the same corner across the
  batch when possible (D16).
- The review grid lets her nudge any photo and change its mark's color and opacity (D16).
- Accessibility pass: touch targets, Dynamic Type, VoiceOver, and contrast (C16).

**Done when** Ky runs a 50-photo auto-placed batch on her phone in airplane mode (D19), and
the scorer's unit tests pass. That's v1.

### M3: Mom's first round

Mom uses it for real and shows Ky what to change in person (D20). JS changes ship with EAS
Update, and native changes need a CI build and an AltStore install.

**Done when** her first round of changes is on her phone.

### Later

L1 to L9 in [capabilities.md](capabilities.md), in the order her feedback suggests.

## Testing

- **`src/core`:** unit tests for layout math and scoring.
- **`src/theme.ts`:** the drift test.
- **Native output:** sample photos in CI (D21).
- **Every release:** a run in airplane mode (C13).
- **Not covered:** there's no Mac, so no Swift debugger. Swift failures show up in CI logs
  and on her phone, which is why the module stays small (uncertainty 2).

## Rules that hold throughout

AGENTS.md has the full list. Nothing leaves the phone, originals are never changed, full
resolution is the default, and Mom stays anonymous in this public repo.
