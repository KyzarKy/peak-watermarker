# Planning log

The running record of the planning dialogue. The request and its constraints are in
[00-kickoff-brief.md](00-kickoff-brief.md), and what the app has to do is in
[capabilities.md](capabilities.md). Decisions are added here as Ky makes them, with the
date. When the questions converge, the written plan goes in `02-plan.md`. Nothing gets
built until Ky signs off on it.

## Status

Rounds 1 to 7 and the question bank are answered. Ky answered rounds 5 and 6 for Mom on
2026-10-31. [02-plan.md](02-plan.md) is written and waiting on Ky's sign-off.

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
5. **Mom's iPhone is a 16 Pro Max on iOS 27.2** (2026-10-31 and 2026-11-02, round 5; Ky's
   first estimate was "a 15 or newer"). Its default photo is a 24MP HEIC with an HDR gain map, and 48MP and ProRAW are
   options. The iPhone 15 Pro made 24MP the default, and the 15 and later capture Adaptive
   HDR from iOS 18 on.
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
13. **Plain React Native styles for v1** (2026-10-31, round 7). @kyzarky/ui's 12 light and
    dark color pairs are copied into `src/theme.ts`, and `useColorScheme()` picks light or
    dark. A unit test parses `theme.css` from the package, pinned to an exact version as a
    dev dependency, and fails when the copy and the package disagree. Why: neither
    NativeWind v5 nor Uniwind kept dark mode when compiling the published `theme.css`, so
    every route needed a copy, and plain styles add no styling library or build config
    (`research/styling-nativewind-vs-plain.md`). Revisit when NativeWind v5 is stable on the
    SDK in use, or when a second native app needs the colors.
14. **Every capture setting is supported** (2026-10-31, round 5). She may switch between High
    Efficiency, Most Compatible, 48MP, and ProRAW at any time, so the app handles each photo
    by its actual format. Her current setting only decides what gets tested first.
15. **Her originals are on the phone** (2026-10-31, round 5). She uses "Download and Keep
    Originals" and has over 50GB free, so batches work offline and space isn't tight. A
    setting for iCloud-only originals may come later. Whether v1 still handles iCloud
    downloads (capabilities C2) is for `02-plan.md`.
16. **A clean sans-serif mark, adjustable per photo** (2026-10-31, round 6). Auto-placement
    picks a readable default, and in the review grid she can nudge any photo's mark or change
    its color and opacity. A good spot is off faces and the subject, easy to read, and in the
    same corner across the batch when possible.
17. **Copies keep each photo's format and location by default** (2026-10-31, round 6). HEIC
    stays HEIC. The remove-location toggle starts off.
18. **Android is the everyday dev phone** (2026-10-31, question bank). Builds run free from
    Ky's PC, and the Kotlin half of the module is written alongside the Swift half. Her
    iPhone gets milestone builds through AltStore. Android Studio, a JDK, and adb need
    installing first.
19. **v1 is done when Ky runs a 50-photo batch on her phone** (2026-10-31, question bank),
    with photos he took. Her first real use feeds milestone 3.
20. **Feedback comes in person** (2026-10-31, question bank). Mom shows Ky what to change,
    and fixes ship in batches. The app has no feedback feature.
21. **Native output is checked in CI** (2026-10-31, question bank). The macOS runner
    watermarks sample photos Ky took (HEIC, ProRAW, a panorama), checks size, metadata, and
    orientation, and saves the results to look at.

## Still open

- The calls `02-plan.md` lists for Ky's sign-off.
- Later features: logo or signature, copyright in metadata, keeping HDR, an album, a
  web-size export, more saved styles, Android.

## Biggest uncertainties

1. **Sideloading has to hold up in daily life.** This is the biggest risk, and the check on
   2026-11-02 made it larger. Users report that Apple's sign-in rejected AltStore, Sideloadly,
   and SideStore several times in September and October 2026. Fixes shipped in late October,
   but reports on Windows are still mixed, and no report shows AltStore with Windows
   AltServer working on iOS 27.2. Each renewal needs that sign-in, so a future break could
   stop the app opening until a tool is patched. If her phone is away from home Wi-Fi, or
   Ky's PC is off, for more than 7 days, the app also won't open until it's renewed. Whether
   her saved style survives that is unverified. Milestone 0 tests the route, and the fallback
   is $99 once.
2. **Native code without a Mac.** Swift compiles only in CI, with no debugger, and every
   native change costs a CI build. So the module stays small with a narrow, typed API, and
   the decisions (layout, scoring) live in TypeScript with unit tests.
3. **The unsigned CI build is a do-it-yourself path.** One developer documented it (GitHub
   Actions plus AltStore, August 2025). Whether EAS's `withoutCredentials` option can do it
   is unverified. Milestone 0 tests this too.
4. **"Optimal" is Mom's call.** The scorer picks calm, readable spots, but calm areas are
   also the easiest to crop out or paint over. Round 6's answer (decision 16) didn't ask for
   hard-to-crop spots, so calm corners are fine for now.

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

## Round 5: Mom's phone settings (Mom, asked 2026-10-30, Ky answered 2026-10-31)

Ky can check these on her phone in about two minutes.

1. **Settings > General > About:** which iPhone, and which iOS version? This decides whether
   AltStore works and which APIs exist. Answer: an iPhone 16 Pro Max on iOS 27.2 (decision
   5).
2. **Settings > Camera > Formats:** "High Efficiency" or "Most Compatible"? Is ProRAW on? Is
   the photo resolution 24MP or 48MP? Answer: she may change it, so support every setting
   (decision 14).
3. **Photos and iCloud** (Settings > Apps > Photos, or her name > iCloud > Photos):
   "Optimize iPhone Storage" or "Download and Keep Originals"? With Optimize, the full-size
   original has to download before the app can use it, and that needs internet. Answer:
   Keep Originals, for now (decision 15).
4. **Free storage:** how much is left? Each copy takes about as much space as its original.
   Answer: over 50GB (decision 15).

## Round 6: how Mom wants it to work (Mom, asked 2026-10-30, Ky answered 2026-10-31)

Record the shape of her answers only, never her name or the mark's text (AGENTS.md rule 4).

1. **The mark:** what should it say, and how should it look: classic, clean, or
   handwritten? Answer: clean (decision 16).
2. **Automatic spots:** when the app picks a spot by itself, what makes a spot good or bad?
   Would she nudge a few in a review grid before saving? Answer: off faces and the subject,
   easy to read, and the same corner when possible. Yes to the review grid, and she also
   wants to change the mark's color and opacity per photo (decision 16).
3. **Location:** remove it from every copy by default, or leave it on and turn it off when
   she wants? Answer: leave it on (decision 17).
4. **Format:** keep each photo's format (HEIC stays HEIC), or always save JPEG so any
   website or print lab takes it? Answer: keep each format (decision 17).
5. **Today:** what does she use now, if anything, and what bugs her about it? Answer:
   nothing yet.

## Round 7: styling (Ky, asked 2026-10-30, answered 2026-10-31)

From `research/styling-nativewind-vs-plain.md`.

1. **Styling for v1:** plain React Native styles with @kyzarky/ui's colors copied into
   `src/theme.ts`, or NativeWind v5 with a native copy of its theme? Answer: plain styles
   (decision 13).
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

## Ky's question bank (answered 2026-10-31)

1. **Dev phone:** should Ky's Android be the everyday dev phone (free builds, the Kotlin half
   early), with her iPhone getting milestone builds? Answer: yes (decision 18).
2. **Styling:** decided in round 7 (decision 13).
3. **v1 done:** proposal: Mom watermarks a real 50-photo batch on her own, and the copies
   sort beside the originals. Answer: Ky's own 50-photo batch on her phone instead
   (decision 19).
4. **Feedback:** how will Mom say what to change, and how fast should fixes reach her? EAS
   Update can ship a JS change the same day. Answer: in person, in batches (decision 20).
5. **Testing without a Mac:** check rendered sample photos (ones Ky took) in CI? Answer:
   yes (decision 21).
6. **Patent:** Shutterstock's US10475145B1 describes saliency-based watermark placement.
   Read it before demoing the auto mode publicly? Not legal advice. Answer: not for now.
   Ky's view is that saliency maps are long-established, open computer vision (Itti, Koch,
   and Niebur, 1998).

## Milestones

These moved to [02-plan.md](02-plan.md).

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

Checked 2026-11-02, for her iOS version. These are user reports, with no official statement
found. Details are in `research/frameworks-and-distribution.md`.

- **Sign-in failures:** -80045 from Oct 12, then -80046, across AltStore and other tools
  ([#1828](https://github.com/altstoreio/AltStore/issues/1828),
  [#1840](https://github.com/altstoreio/AltStore/issues/1840)).
- **After the fixes, on Windows:** one user reports AltServer 1.8.1.0 with AltStore 2.3.2
  working, without giving an iOS version
  ([#1840](https://github.com/altstoreio/AltStore/issues/1840)). Others report a signing
  error, including on iOS 27.2, where Sideloadly v0.72 then worked with a new free Apple ID
  ([#1856](https://github.com/altstoreio/AltStore/issues/1856)).
