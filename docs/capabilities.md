# Capabilities

Everything the app has to be able to do, and which platform API does each part. The v1
list follows the scope in the planning log (decision 11). "Later" items wait for their own
milestone.

**Owners.** `src/core` is pure TypeScript: it decides layout, placement, and style, and it
has unit tests. The module is `modules/watermark-engine`, a local Expo module in Swift and
Kotlin that handles decoding, compositing, metadata, and saving. The UI is the Expo app's
screens. An Expo library is used where
one already does the job. Infra covers builds, installs, and updates.

**Sources.** The API details come from `research/iphone-photos-and-platform-apis.md` and
`research/frameworks-and-distribution.md` (local, checked 2026-10-29). The Apple pages
behind the key facts are linked inline.

## v1

### C1. Pick many photos, in order, without the camera
- **Owner:** an Expo library (expo-image-picker); the module only if the library falls
  short.
- **iOS:** `PHPickerViewController` runs in a separate process and needs no photo-library
  permission ([Apple](https://developer.apple.com/documentation/photokit/delivering-an-enhanced-privacy-experience-in-your-photos-app)).
  Use `selectionLimit = 0` (the system maximum) and `.ordered`, plus
  `preferredAssetRepresentationMode = .current` so HEIC isn't transcoded to JPEG.
  expo-image-picker exposes the selection limit and the representation mode; whether it
  exposes ordered selection is unverified. With editing off, quality 1, and `current`, it
  copies the original file byte for byte.
- **Android (later):** `PickMultipleVisualMedia`, no permission. Picks arrive with GPS
  stripped by default.
- The picker returns each photo's edited version. That is the source photo.

### C2. Bring each photo onto the phone before working on it
- **Owner:** the module (or the Expo library), plus UI states.
- With "Optimize iPhone Storage", the original lives in iCloud and the picker downloads it,
  which needs internet. Offline, the download can stall with no error.
- Copy each file into the app's container right away; the picker deletes its temporary file
  when the completion handler returns.
- Each photo shows one state: ready; downloading, with progress; or "In iCloud, connect to
  download" with Retry. A timeout catches a stall, and one stuck photo never blocks the rest
  of the batch.

### C3. Decode every iPhone format at full resolution
- **Owner:** the module.
- **iOS:** read the metadata with `CGImageSourceCopyPropertiesAtIndex`. Load pixels with
  `CIImage(contentsOf:options:)` and `.applyOrientationProperty`, which decodes to SDR. Decode
  ProRAW (DNG) with `CIRAWFilter`. Detect the type from the file's UTType, not its
  extension.
- A Live Photo becomes its still image. Portrait and spatial photos become their primary
  image. HDR gain maps are dropped in v1 (see L3). The result says "saved as a still photo"
  when that happens.
- **Android (later):** `ImageDecoder`. HEIF decoding is device-optional, so check
  `ImageDecoder.isMimeTypeSupported("image/heif")` and report a failure per photo.

### C4. A styled text mark
- **Owner:** `src/core` holds the style as data; the module draws it.
- Style: text, font, size, color, opacity, and a shadow or outline for legibility.
- **iOS:** `CIFilter.attributedTextImageGenerator()`, which Apple's docs describe as commonly
  used for watermarks
  ([Apple](https://developer.apple.com/documentation/coreimage/cifilter-swift.class/attributedtextimagegenerator())).

### C5. Layout relative to each photo
- **Owner:** `src/core`.
- A 9-point anchor, an inset, and a size given as a fraction of the photo's short edge. One
  layout then looks the same on portrait, landscape, and 63MP panorama photos. A review of
  iWatermark+ complains that its mark resized "to fit the odd photos" in a mixed batch, and
  Watermarkly and Visual Watermark advertise auto-sizing to prevent that.

### C6. Preview and place
- **Owner:** the UI, with thumbnails from the module.
- Preview on a downscaled copy: `CGImageSourceCreateThumbnailAtIndex` cut memory from about
  220 MB to 24 MB in NSHipster's test. Drag the mark, or snap it to the 9 points.
- The same normalized layout drives the full-resolution render, so the saved copy matches
  the preview.

### C7. Auto-placement with a review grid
- **Owner:** `src/core` decides. The module supplies thumbnail pixels, saliency, and faces.
- **The scorer** works on a thumbnail of about 256px:
  - Candidates: the four corners and bottom-center.
  - Each candidate scores by edge density plus luminance variance.
  - A penalty applies for overlapping Vision's attention saliency (a 64x64 or 68x68 heat
    map) or a face box.
  - The lowest score wins.
- WCAG contrast picks a white or black mark, plus a soft shadow.
- **Review grid:** she sees every photo with its spot and taps any one to nudge it, or to
  change that photo's mark color and opacity (decision 16).
- No model downloads. Foundation Models was ruled out: its iOS 27 image input returns text,
  not coordinates, at seconds per photo.
- **Android (later):** the same scorer. Add bundled face detection only if faces turn out to
  matter.

### C8. Full-resolution render, one photo at a time
- **Owner:** the module.
- One shared `CIContext` with `cacheIntermediates: false` and a raised memory limit (iOS
  defaults to 256 MB). Render the batch in sequence and release each photo before the next.
- A 48MP photo is about 195 MB decoded, and about 390 MB in Core Image's default working
  format.

### C9. Encode with the metadata kept
- **Owner:** the module.
- Write the same container as the input: HEIC stays HEIC, JPEG stays JPEG, PNG stays PNG,
  and DNG becomes HEIC. She keeps each format rather than always getting JPEG (decision 17).
- **iOS:** `CGImageDestinationAddImage` with the source's properties
  ([Apple](https://developer.apple.com/documentation/imageio/cgimagedestination)). Set
  `Orientation = 1`, because the pixels are already rotated. Set a compression quality, and
  keep the source color space (Display P3).

### C10. Remove-location toggle
- **Owner:** the module and the UI.
- **iOS:** delete the `{GPS}` dictionary (or use `kCGImageMetadataShouldExcludeGPS`), and
  don't set the saved asset's location. The toggle starts off, so copies keep their location
  unless she turns it on (decision 17).
- **Android (later):** the picker already strips GPS.

### C11. Save copies to Photos
- **Owner:** the module (or expo-media-library).
- **iOS:** add-only access (`NSPhotoLibraryAddUsageDescription`) with
  `PHAssetCreationRequest` and `addResource(with: .photo, fileURL:)`
  ([Apple](https://developer.apple.com/documentation/photos/phassetcreationrequest)).
  - Never use the `data:` variant: it stripped EXIF on iOS 26.6 to 26.7.
  - Set `creationDate`, and `location` unless it was removed, from the source photo. The
    copy then sorts beside the original when Photos sorts by Date Captured.
  - expo-media-library's `Asset.create` keeps the file's metadata but sets neither property.
- No album in v1, because albums need more than add-only access (see L5).
- **Android (later):** MediaStore with `IS_PENDING`, no permission. Copy `DateTimeOriginal`
  and `OffsetTimeOriginal` so the date taken is right.

### C12. Batch progress
- **Owner:** the UI and `src/core`.
- Progress, cancel, and skip-and-report for a photo that fails. The screen stays awake
  during a batch (expo-keep-awake).

### C13. Offline and private
- **Owner:** everyone.
- Photos, the mark, and settings never touch the network. No analytics, crash reporting,
  accounts, or cloud ML.
- Only update downloads (EAS Update) use the network, and the app works without them.
- Test every release in airplane mode.

### C14. One saved style
- **Owner:** `src/core`, with on-device storage.
- The style has to survive app updates and the 7-day renewals. Milestone 0 checks the
  renewals.

### C15. Small changes reach her fast
- **Owner:** infra.
- EAS Update ships JS changes over the air; its free tier covers 1,000 monthly users. A
  native change needs a new CI build and an AltStore install.
- So the values feedback will tune (sizes, insets, scoring weights) live as data in
  `src/core`, where changing them is a JS change.

### C16. Easy for Mom, and accessible
- **Owner:** the UI.
- Large touch targets, Dynamic Type, VoiceOver labels, and good contrast.
- Three steps: pick, place, save.

### C17. Install and renew at $0
- **Owner:** infra; this is milestone 0.
- **Build:** an unsigned iOS build on GitHub Actions; macOS runners are free for public
  repos.
- **Install:** AltStore signs it with a free Apple ID. AltServer on Ky's PC needs iTunes and
  iCloud from Apple's site, not the Microsoft Store. Developer Mode goes on her phone.
- **Renewal:** every 7 days, when her phone is on the same Wi-Fi as Ky's PC with AltServer
  running.

## Later

- **L1. Logo or signature mark:** a PNG with transparency.
- **L2. Tiled mark** across the whole photo.
- **L3. Keep HDR:** Core Image's `.hdrGainMapImage`, or ImageIO's ISO gain map, with the gain
  map neutral under the mark and HEIC output.
- **L4. Copyright in metadata:** TIFF Copyright and Artist, and IPTC CopyrightNotice. DMCA
  §1202 covers a © notice and the author's name, whether visible or embedded.
- **L5. A "Watermarked" album:** needs limited or full library access.
- **L6. Web-size export:** for example, a 2048px long edge for social posts.
- **L7. More saved styles.**
- **L8. Android build.**
- **L9. App Store listing ($99 once),** if sideloading fails.

## Parity with the market

From `research/market-watermark-apps.md`. The tiers are how common each feature is across
the apps surveyed.

| Feature | Market tier | Plan |
|---|---|---|
| Text mark with font, color, opacity, size | Table stakes | v1 (C4) |
| Rotation | Table stakes | Later |
| Transparent PNG logo | Table stakes | Later (L1) |
| Drag plus preset anchor points | Table stakes | v1 (C5, C6) |
| One design applied to many photos | Table stakes | v1, free, no batch cap |
| Saved templates | Table stakes | v1 has one (C14); more later (L7) |
| New copy, original untouched, full resolution | Table stakes | v1 (C8, C11) |
| Size relative to each photo across orientations | Common | v1 (C5) |
| Per-photo override inside a batch | Common | v1 (C7's review grid) |
| Margins or inset | Common | v1 (C5) |
| Shadow or outline | Common | v1 (C4) |
| Tiled pattern | Common | Later (L2) |
| Signature capture | Common | Later (L1) |
| Resize or change format on export | Common | Later (L6); format question in round 6 |
| QR codes, date or EXIF stamps | Common | Not planned |
| Keeps EXIF, including capture date | Differentiator | v1 (C9, C11) |
| Automatic color for contrast | Differentiator | v1 (C7) |
| Placement based on the photo's content | Differentiator | v1, on the device (C7) |
| Offline, no data collected, no ads | Differentiator (paid on iOS; iWatermark+ Lite's free text marks may not batch) | v1, free (C13) |
| Copyright written into metadata | Differentiator | Later (L4) |
| Per-image variation to resist removal | Differentiator | Not planned |
| Invisible mark | Differentiator | Not planned |
