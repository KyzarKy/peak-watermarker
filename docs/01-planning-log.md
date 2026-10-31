# Planning log

The running record of the planning dialogue. The request and its constraints are in
[00-kickoff-brief.md](00-kickoff-brief.md), and what the app has to do is in
[capabilities.md](capabilities.md). Decisions are added here as Ky makes them, with the
date. When the questions converge, the written plan goes in `02-plan.md`. Nothing gets
built until Ky signs off on it.

## Status

Rounds 1 to 4 answered. Rounds 5 and 6 are for Mom; Ky asks her. Asked 2026-10-30, waiting
on her answers. Round 7, on styling, is for Ky (asked 2026-10-30).

## Decided

Settled in the brief, before planning started:

- Free for Mom, permanently.
- Nothing leaves her phone, and saving works without internet.
- She picks photos she already has. The app never uses the camera.
- One photo or many at a time.
- One placement applied to every selected photo, or a spot chosen per photo by on-device
  vision.
- Every format an iPhone produces.
- Bare-bones first, easy to change from Mom's feedback.
- Parity with standard watermark tools, not competition.

Decided in the dialogue:

1. **No Mac** (2026-10-29). Development is Windows 11 only, so the iOS app compiles in
   cloud CI.
2. **Running it costs $0** (2026-10-29). No Apple Developer Program. The fallback, only if
   sideloading fails: pay $99 once for a free App Store listing. Apple says installed apps
   "will still function" after a membership lapses. After that there are no updates, and
   reinstalling, for example on a new phone, would likely mean renewing (an inference, not
   checked).
3. **Portfolio piece** (2026-10-29), under the working agreement in AGENTS.md.
4. **iPhone first, Android later** (2026-10-29). The code stays portable, so Android is a
   later milestone, not a rewrite.
5. **Mom's iPhone is a 15 or newer** (2026-10-29, Ky's estimate; round 5 confirms). Its
   default photo is then most likely a 24MP HEIC with an HDR gain map, and 48MP is possible.
   The iPhone 15 Pro made 24MP the default, and the 15 and later capture Adaptive HDR from
   iOS 18 on.
6. **Full resolution is required** (2026-10-29). She uses the copies for social media, her
   website, and prints or clients. A web-size export is a later convenience.
7. **Batches of 50 or more** (2026-10-29). Batch speed, memory use, and a review grid
   matter.
8. **Working name: peak-watermarker** (2026-10-29). "Peak" doesn't point to Mom
   (2026-10-30).
9. **Expo (React Native) plus one local Expo module in Swift and Kotlin** (2026-10-29).
   Native code handles decoding, compositing, metadata, saving, and Vision.
   TypeScript decides where the mark goes and how it looks. Why: native fidelity for 24MP
   HDR HEICs and full-resolution batches, React Native portfolio value, and no Mac needed to
   build.
10. **Sideload with AltStore** (2026-10-29). GitHub Actions builds an unsigned iPhone app
    (macOS runners are free for public repos). AltStore signs it with a free Apple ID, and
    renews the 7-day signature when her phone is on the same Wi-Fi as Ky's PC with
    AltServer running. JS
    changes ship over the air with EAS Update. The first milestone proves this route on her
    iOS version before any feature work.
11. **v1 scope** (2026-10-29): a text mark in the same relative spot across a batch;
    auto-placement with a review grid where she can nudge any photo; and a remove-location
    toggle. Logo and signature marks come later.
12. **Public GitHub repo from the first commit** (2026-10-29), with Fluent Prior's "public to
    read" license (2026-10-30). Everything is visible, so Mom's details, her mark's text,
    and secrets stay out.

## Still open

- Mom's phone settings (round 5).
- How Mom wants it to work (round 6).
- Styling (round 7).
- Ky's question bank (below).
- Later features: logo or signature, copyright in metadata, keeping HDR, an album, a
  web-size export, more saved styles, Android.

## Biggest uncertainties

1. **Sideloading has to hold up in daily life.** AltStore's support for iOS 27 is
   unverified; Windows AltServer 1.7.4 fixed an iOS 26.4 crash in March 2026. If her phone
   is away from home Wi-Fi, or Ky's PC is off, for more than 7 days, the app won't open
   until it's renewed. Whether her saved style survives that is unverified. Milestone 0
   tests the route, and the fallback is $99 once.
2. **Native code without a Mac.** Swift compiles only in CI, with no debugger, and every
   native change costs a CI build. So the module stays small with a narrow, typed API, and
   the decisions (layout, scoring) live in TypeScript with unit tests.
3. **The unsigned CI build is a do-it-yourself path.** One developer documented it (GitHub
   Actions plus AltStore, August 2025). Whether EAS's `withoutCredentials` option can do it
   is unverified. Milestone 0 tests this too.
4. **"Optimal" is Mom's call.** The scorer picks calm, readable spots, but calm areas are
   also the easiest to crop out or paint over. Round 6 asks what she means by a good spot.

## Round 1 (asked and answered 2026-10-29)

1. **Mac?** Answer: no Mac (decision 1).
2. **Apple's fee, about $99 a year?** Answer: no, it must be $0 (decision 2).
3. **Portfolio piece under the working agreement?** Answer: yes (decision 3).
4. **When does Android come in?** Answer: iPhone first, Android later (decision 4).

## Round 2 (asked and answered 2026-10-29)

Ky's best guesses, to confirm with Mom.

1. **Which iPhone?** Answer: a 15 or newer (decision 5).
2. **What does she do with the copies?** Answer: social media, her website, and prints or
   clients (decision 6).
3. **Photos per batch?** Answer: 50 or more (decision 7).
4. **Working name?** Answer: peak-watermarker (decision 8).

## Round 3 (asked and answered 2026-10-29)

1. **The route at $0 with no Mac:** an Expo app sideloaded with AltStore, the same app on
   the App Store for $99 once, or a home-screen web app? Answer: the sideloaded Expo app
   (decisions 9 and 10). The web app was set aside for three reasons. WebKit bug reports say
   Safari's picker converts HEIC to JPEG. Redrawing on a canvas drops HDR. And iOS may refuse
   a 48MP canvas. It also isn't React Native.
2. **What goes in the first version?** Answer: the text mark in the same spot,
   auto-placement, and the remove-location toggle. The logo waits (decision 11).
3. **GitHub repo?** Answer: public from the start (decision 12).

## Round 4 (asked and answered 2026-10-30)

1. **Is "Peak" part of Mom's name, business, or brand?** Answer: no (decision 8).
2. **License?** Answer: public to read, like Fluent Prior (decision 12).

## Round 5: Mom's phone settings (Mom, asked 2026-10-30)

Ky can check these on her phone in about two minutes.

1. **Settings > General > About:** which iPhone, and which iOS version? This decides whether
   AltStore works and which APIs exist.
2. **Settings > Camera > Formats:** "High Efficiency" or "Most Compatible"? Is ProRAW on? Is
   the photo resolution 24MP or 48MP?
3. **Photos and iCloud** (Settings > Apps > Photos, or her name > iCloud > Photos):
   "Optimize iPhone Storage" or "Download and Keep Originals"? With Optimize, the full-size original has to download before the app
   can use it, and that needs internet.
4. **Free storage:** how much is left? Each copy takes about as much space as its original.

## Round 6: how Mom wants it to work (Mom, asked 2026-10-30)

Record the shape of her answers only, never her name or the mark's text (AGENTS.md rule 4).

1. **The mark:** what should it say, and how should it look: classic, clean, or
   handwritten?
2. **Automatic spots:** when the app picks a spot by itself, what makes a spot good or bad?
   Would she nudge a few in a review grid before saving?
3. **Location:** remove it from every copy by default, or leave it on and turn it off when
   she wants?
4. **Format:** keep each photo's format (HEIC stays HEIC), or always save JPEG so any
   website or print lab takes it?
5. **Today:** what does she use now, if anything, and what bugs her about it?

## Round 7: styling (Ky, asked 2026-10-30)

From `research/styling-nativewind-vs-plain.md`.

1. **Styling for v1:** plain React Native styles with @kyzarky/ui's colors copied into
   `src/theme.ts`, or NativeWind v5 with a native copy of its theme?
   - Neither Tailwind library that Expo's guide names, NativeWind or Uniwind, reads
     `theme.css` 0.5.2 as published and keeps dark mode. Compiled for iOS, NativeWind v5's
     release candidate kept only the light colors, and Uniwind compiled every @kyzarky/ui
     color to black. So every route copies the 12 color pairs, and the package's
     components are web-only either way.
   - Recommendation: plain styles, plus a unit test that fails when the copy and the
     pinned package disagree. It adds no styling library or build config, and it works on
     SDK 58 today. NativeWind v5 is a pre-release tested on SDK 57, and it needs
     lightningcss pinned to an older version.
   - Revisit when NativeWind v5 is stable on the SDK in use, or when a second native app
     needs the colors. Then the tokens could move to their own package, which @kyzarky/ui's
     ADR 0001 already allows for.

## Ky's question bank (later rounds)

Reorder as answers come in.

1. **Dev phone:** should Ky's Android be the everyday dev phone (free builds, the Kotlin half
   early), with her iPhone getting milestone builds?
2. **Styling:** researched and moved to round 7 (2026-10-30).
3. **v1 done:** proposal: Mom watermarks a real 50-photo batch on her own, and the copies
   sort beside the originals.
4. **Feedback:** how will Mom say what to change, and how fast should fixes reach her? EAS
   Update can ship a JS change the same day.
5. **Testing without a Mac:** check rendered sample photos (ones Ky took) in CI?
6. **Patent:** Shutterstock's US10475145B1 describes saliency-based watermark placement.
   Read it before demoing the auto mode publicly? Not legal advice.

## Draft milestones (for 02-plan.md, not signed off)

- **M0, prove the install route.** A blank Expo app with a one-function local module.
  GitHub Actions builds it unsigned, and AltStore installs it on her phone with Developer
  Mode on. Metro connects over home Wi-Fi, EAS Update delivers a JS change, and the app
  opens after one 7-day renewal with its data intact. If any step fails, this log takes up
  $99 once versus a web app.
- **M1, the core loop.** Pick many photos in order, style a text mark, place it once
  (9-point grid, drag, size relative to each photo), render full resolution one photo at a
  time, save copies with their metadata and original capture date, and toggle location off.
  Done when Mom watermarks a real 50-photo batch.
- **M2, auto-placement.** A native thumbnail plus Vision saliency and face boxes feed the
  TypeScript scorer. A review grid lets her nudge any photo before saving.
- **M3, her feedback.** The first round of Mom's changes, shipped over the air.
- **Later:** the "later" list in [capabilities.md](capabilities.md).

## Facts the decisions rest on

Checked 2026-10-29. The full research, with every source, is in `research/` (gitignored,
on Ky's machine).

- **EAS signing:** "All builds that run on an iPhone device require a paid Apple Developer
  account for build signing"
  ([Expo](https://docs.expo.dev/develop/development-builds/create-a-build/)).
- **A lapsed membership:** apps "will still function for users who have already installed
  or downloaded them"
  ([Apple](https://developer.apple.com/help/account/membership/renewal/)).
- **AltStore with a free Apple ID:** apps expire after 7 days and refresh when the phone is
  on the same Wi-Fi as a PC running AltServer, with at most 3 active apps
  ([AltStore](https://faq.altstore.io/altstore-classic/altserver)). The iOS 26.4 fix is in
  the [AltServer release notes](https://faq.altstore.io/release-notes/altserver).
- **CI minutes:** GitHub-hosted runners are free for public repos
  ([GitHub](https://docs.github.com/en/billing/concepts/product-billing/github-actions)).
- **Her photos:** the iPhone 15 Pro made 24MP the default
  ([Apple](https://www.apple.com/newsroom/2023/09/apple-unveils-iphone-15-pro-and-iphone-15-pro-max/)),
  and iPhone 15 and later capture Adaptive HDR (ISO 21496-1) from iOS 18
  ([WWDC24](https://developer.apple.com/videos/play/wwdc2024/10177/)).
- **Safari's canvas limit:** iOS has long refused canvases over 16,777,216 pixels
  ([PQINA](https://pqina.nl/blog/canvas-area-exceeds-the-maximum-limit/)). A 17.2MP canvas
  worked on an iPhone 15 Pro Max on iOS 26.6.1
  ([DPLA](https://github.com/dpla/dpla-frontend/issues/1622)).
- **On-device model:** Foundation Models image input (iOS 27) returns text, at 2.6 to 3.8
  seconds per response in one beta test
  ([Classmethod](https://dev.classmethod.jp/en/articles/foundation-models-multimodal-image-analysis/)).
- **The market:** every offline iOS watermark app found with no data collection and no ads
  was paid. They were iWatermark+ ($4.99), EasyWatermark ($3.99), Batch Watermark Pro
  ($3.99), and LocalStamp ($0.99 after a trial, Chinese-only). iWatermark+ Lite's text
  marks are free, but whether it batches is unverified. Details are in
  `research/market-watermark-apps.md`.

Checked 2026-10-30, for round 7. Details are in `research/styling-nativewind-vs-plain.md`.

- **NativeWind v5:** its docs say "This is a pre-release version of Nativewind. It is not
  intended for production use." The release candidate's tested target is "Expo 57.0.22
  with React Native 0.86.3"
  ([NativeWind](https://www.nativewind.dev/v5/guides/migrate-from-preview)). The upgrade
  guide says to "pin lightningcss 1.30.1", and that v4, the stable release, "uses Tailwind
  CSS 3" ([NativeWind](https://www.nativewind.dev/v5/guides/migrate-from-v4)).
- **Uniwind:** version 1.13.1, MIT ([npm](https://www.npmjs.com/package/uniwind)). Expo's
  Tailwind guide names it and NativeWind for iOS and Android
  ([Expo](https://docs.expo.dev/guides/tailwind/)).
- **The compile results** come from running each library's compiler on @kyzarky/ui's
  `theme.css` 0.5.2. The research file has the steps.
- **Dark mode in Expo** needs `"userInterfaceStyle": "automatic"`. Without it, the app
  stays light ([Expo](https://docs.expo.dev/develop/user-interface/color-themes/)).
