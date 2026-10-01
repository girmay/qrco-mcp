# QRCO MCP tool reference

[Setup](README.md#connect-in-your-ai-application) · [Examples](EXAMPLES.md) · [Live reference](https://qrco.ca/mcp-guide.md)

## Public endpoint: https://qrco.ca/mcp

No authentication; Streamable HTTP. Seventeen tools.

| Tool | Purpose |
| --- | --- |
| `generate_palette` | 2–10 colors, one generation mode and optional fixed colors. |
| `generate_palette_set` | compare 1–5 generated directions in one call. |
| `random_palette` | generate a random palette. |
| `inspect_color` | HEX, RGB, HSL, HSB, Lab, CMYK and nearest digital swatch approximations. |
| `vary_palette` | produce variations from existing colors. |
| `match_swatch` | nearest digital Pantone approximation and color-distance result; verify physically for print. |
| `check_contrast` | WCAG 2.x text contrast for two opaque sRGB colors. |
| `audit_palettes` | Audit 1–10 identified palettes with actual backgrounds and intended pairings. Compact summaries or detailed failures. |
| `compare_palettes` | Compare 2–5 palettes against a baseline: Delta E 2000, contrast opportunities and exact anchor retention. |
| `audit_palette` | measured contrast across palette pairs and black/white foreground suggestions. |
| `create_studio_link` | carry colors, palette title and custom color names into Studio; does not save. |
| `export_palette` | return CSS, Tailwind CSS v4, SCSS, JSON or SVG content; does not write files. |
| `search_explore` | browse the public Explore catalog by name or exact HEX. |
| `check_color_vision` | Simulate protanopia/deuteranopia/severe tritanomaly and flag pair separation for review. |
| `generate_state_colors` | Generate interaction/focus colors with actual-surface checks, disabled exclusions and CSS/Tailwind tokens. |
| `generate_tonal_scale` | Build QRCO 50–950 stops from a seed, preserve an original token, measure contrast and export CSS/Tailwind. |
| `build_brand_system` | derive light, dark or dual-mode roles, measured pairings, semantic CSS and Tailwind exports. |

## Account endpoint: https://qrco.ca/mcp/account

OAuth; Streamable HTTP. Four read tools and up to four additional write tools, depending on approval. Keep the public connection for color generation.

| Tool | Permission | Purpose |
| --- | --- | --- |
| `list_saved_palettes` | `palettes:read` | List up to 25 saved palettes per page; follow nextOffset. |
| `get_saved_palette` | `palettes:read` | Read colors, custom names, saved brand roles, measured contrast and publication state. |
| `preview_brand_revision` | `palettes:read` | Preview supporting-role changes, stronger contrast or a missing mode. Returns a previewHash; never saves. |
| `get_palette_history` | `palettes:read` | List tracked versions or retrieve an exact snapshot. Follow nextBeforeVersion for older pages. |
| `save_palette` | `palettes:save` | Save a palette privately. |
| `save_brand_revision` | `palettes:revise` | Commit an exact reviewed feasible preview as a new tracked version. |
| `publish_palette` | `palettes:publish` | Publish an explicitly approved snapshot and description to a public page. |
| `unpublish_palette` | `palettes:publish` | Make the public page unavailable while retaining the private saved palette. |

## OAuth integration details

QRCO supports dynamic public-client registration, authorization code with PKCE S256 and rotating refresh tokens. Follow the protected-resource and authorization-server discovery metadata rather than hardcoding registration or token endpoints.

The account resource is `https://qrco.ca/mcp/account`. Supply that resource indicator when supported. For generic OAuth compatibility, omission defaults exclusively to the account resource; explicitly different resources are rejected. Request only the scopes the user needs. Sign-in and consent happen on QRCO's website.

## Versioning and write semantics

A revision preview identifies exact options, expected palette version and a `previewHash`. Saving that revision requires the same reviewed values and a separate `palettes:revise` grant. Palette colors, names and anchors remain fixed; supporting roles may change. Conflicting locked roles or targets can make a preview infeasible.

History begins with tracked edits; older overwritten states are not reconstructed. Opening a historical snapshot does not restore it over the current version. Deleting a palette also deletes its history. Private saves and revisions do not update its published snapshot.

Use the same request ID and identical arguments for retries of the same write. A replay receipt confirms the earlier operation, not necessarily the palette's present state; read again when current state matters.

This document summarizes capabilities. The server's advertised tool schemas are authoritative for arguments and validation. Read the [measurement limits](README.md#measurements-and-limits) before treating results as print, accessibility or legal certification.

## Comparison and batch audits (public v1.1.0)

Each palette has a unique `id`, `colors` (2–10 HEX values), optional `name` / `names`, and optional `pairings` with zero-based `foregroundIndex`, `backgroundIndex` and `minimum` (default 4.5). Shared `backgrounds` have `id`, `color` and `minimum`. Limits: 4 backgrounds, 20 pairings per palette. Invalid batches reject completely.

`detail` defaults to `summary`. Choose `failures` for failed intended checks and similar pairs, or `full` for all measurements. Without requested checks, `allRequestedChecksPass` is null. Diagnostic all-pair counts never certify a palette.

Comparison defaults: `baselineIndex: 0`, `matching: "unordered"`. Unordered distance averages both directional nearest-neighbor lists; matches can be many-to-one. `indexed` requires equal lengths and compares corresponding positions. Optional `anchors` check exact color retention. Results retain input order and do not choose a winner.

Delta E 2000 uses standard weighting, without capping at 100. `similarityThreshold` defaults to 2, a configurable heuristic rather than a colorblindness or indistinguishability guarantee. All thresholds use unrounded values. No generation or network calls, private mutations or publishing.

## Tonal scales (public v1.2.0)

`generate_tonal_scale` accepts `seed`, optional safe `prefix` (default `brand`), optional `stops` and up to four `backgrounds` with `id`, `color`, and `minimum` (default 4.5). Stops are unique members of 50/100/200/300/400/500/600/700/800/900/950 and returned in ascending order. A subset retains the same colors as the full scale.

The exact seed is preserved as `brand-original`, independently of numbered stops. QRCO uses fixed OKLCH lightness targets, starts from the seed’s hue/chroma, and reduces chroma where needed to fit sRGB. Neutral seeds remain neutral. Each stop returns final HEX/OKLCH, target lightness, gamut diagnostics, measured white/black contrast and any requested background checks. CSS and Tailwind CSS v4 exports include the original plus generated stops.

This is QRCO’s recipe, not Tailwind’s built-in palette or equal-perceptual-distance steps. Stop 500 need not equal the seed. Measurements use final 8-bit colors and unrounded thresholds; no complete-accessibility guarantee. The full 11-stop scale exceeds Studio’s 10-color limit: use export_palette for all11stops plus original, or choose a subset for Studio/saving/audits/brand systems. No account save, publication or network calls.

## Argument and save contract (public v1.2.1 / account v1.0.1)

Unknown arguments are rejected instead of silently discarded. Use the refreshed tool schemas: `generate_palette.size`, `generate_palette_set.paletteCount` and `colorsPerPalette`, `random_palette.count` (colors in ONE palette), `create_studio_link.name`, `export_palette.format`, and `compare_palettes.baselineIndex` (zero-based index, not an ID). `vary_palette` has no `variants` filter. Neither generator accepts a free-text `brief`; the assistant maps the brief to supported controls. Choose at most one generation mode. `baseColor` constrains unlocked colors to the requested family; use `lockedColors` for an exact anchor. Recognized families: red, orange, yellow, green, teal, blue, purple, pink, brown, neutral.

For `save_palette`, supply 2–10 six-digit HEX values with `#`; uppercase and lowercase are accepted and stored uppercase. To attach roles, copy **the `system` field** from the public `build_brand_system` response into `brandSystem`, and use `system.palette` as `colors`. Do not pass the whole response or just `{mode, roles}`.

Supported persisted shapes:

| Mode | Required fields |
| --- | --- |
| light | `version: 1`, `mode: "light"`, `palette`, `roles` |
| dark | `version: 2`, `mode: "dark"`, `palette`, `roles` |
| both | `version: 2`, `mode: "both"`, `palette`, `roles` (light), `darkRoles` |

Every roles object contains exactly `background`, `surface`, `text`, `mutedText`, `primary`, `onPrimary`, `accent`, `onAccent`, `border`, `focus`. The palette must match saved colors in order. Primary/accent must be palette members and agree across modes; measured role contrast must pass validation. A system's format `version` is distinct from a saved palette's revision `version`.

`preview_brand_revision` requires an existing saved `metadata.brandSystem`. Preview options may be omitted or partial; the result returns their complete normalized form. **Saving requires all four returned option fields** (`addMissingMode`, `textContrast`, `uiContrast`, `lockedRoles`), plus the preview's `id`, saved-palette `version`, `previewHash`, and a fresh UUID `requestId`. Copy them; do not rebuild options from the initial request. Only save a reviewed canSave:true preview; changes may be roles or targetChange. Changed inputs or palette versions require a new preview. Reuse identical arguments/requestId only to retry the same save. No publish occurs.

HEX ratios concern opaque sRGB colors. They do not predict tattoo healing/appearance, physical skin, fabric, paint or lighting. Nearest Pantone catalog results need physical proofing with the production provider; they are not production approval or trademark clearance. Neutral tonal seeds intentionally yield neutral ramps.

## Brand colors used as links (public v1.3.0 / account v1.1.0)

`build_brand_system` and `get_saved_palette` (when a valid brand system is attached) now return `usageChecks` separately from existing `pairings`. This tests primary and accent as normal-sized link text against **both background and surface in each stored mode**, at 4.5:1. Four checks per mode; eight for dual mode. An existing system can pass all button/body/outline pairings and still fail these link uses.

Inspect `failureCount`, `originalColorsPassAllListedSurfaces`, and `modes[].links[]`. Each link has its original HEX, per-surface checks, `passesOnAllListedSurfaces`, and a `status`. Failures may include `suggestedLinkColor` with a HEX, measured checks and `applied: false`. A passing link needs no suggestion. `no-suggestion-found` means the bounded search found none, not proof that no color exists.

Example: primary #767676 passes on white at 4.5422:1 but fails on its generated #FBFBFB page at 4.3895:1. The light-mode suggestion #747474 passes both. Dark mode is checked independently and may need a different shade. Pass/fail uses unrounded measurements.

Suggestions are separate colors for review, obtained by bounded mixing toward black or white and least squared RGB change among sampled passing candidates. They do not modify anchors, saved roles, history, the Studio URL or CSS/Tailwind exports. No schema migration, new scope or new tool is needed. Existing revisions still operate on the existing ten roles; suggestions are not saved link roles.

Use a persistent non-color cue such as an underline for inline links. Surrounding body-text contrast is informational; this audit does not certify link identification or focus behavior. See [W3C Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color) and [Contrast Minimum](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum). Other surfaces, overlays, images and hover/visited states are outside this check; use explicit `audit_palettes` background/pairing inputs or `check_contrast` for other opaque HEX pairs.

## Named base families (public v1.3.1)

`generate_palette` and `generate_palette_set` now constrain all generated **unlocked** colors to the requested `baseColor` family. The engine uses explicit sRGB HSL bounds, handles red across 360°/0°, and rechecks family membership after material styling. The previous generator incorrectly treated circular red as a linear interval and fed HSL-style family angles into LCH. Derived color-name hue classification now uses HSL too; existing saved/custom names are untouched.

Responses include `metadata.baseColorConstraint`: requested family, color space, hue/saturation/lightness bounds, `unlockedColorsInFamily`, and `lockedExceptions`. Exact user locks override the family and remain exact even with material styling. For example, a locked green in a red palette remains green and is reported as an exception. Unlocked red swatches stay within hue345–15°, saturation0.60–0.95, lightness0.28–0.52. These are QRCO product-defined family ranges, not universal color naming or a safety standard.

This makes the family request enforceable; it does not assign danger/status meaning, guarantee contrast or simulate physical materials. Check the actual foreground/background pairs and use appropriate labels/icons. A stronger signal or icon does not automatically repair insufficient text contrast.

`brief` and `variants` were already absent from the preceding MCP schemas; strict validation now rejects them instead of ignoring them. `vary_palette` already required 2–10 colors. Brand-system `pairings` already include focus/background and focus/surface checks; geometry, occlusion and full focus accessibility remain outside those measurements.

## Nearest Pantone option by tool

`includePantone` is accepted by `generate_palette` and `inspect_color`, but **not** by `generate_palette_set`. Omit it from batch-generation calls; unknown arguments are rejected. To obtain a nearest Pantone catalog match for a chosen batch color, call `match_swatch` with its `hex`. This documents the current schemas; it does not add batch matching or certify physical print results.

## Save identity and revision readiness (account v1.1.1)

`save_palette` deduplicates within the authorized account against **current stored content**: normalized palette name, ordered colors, and complete metadata. Normalization trims palette/custom-name whitespace, uppercases HEX colors and canonicalizes object-key order. Array order matters. Custom color names, the complete brand system, and Studio metadata (`locks`, `style`, `harmony`, `source`) participate. Missing metadata is not equivalent to an explicitly populated field. Two entries with the same palette name and HEX values may therefore be distinct. No automatic merging or deletion occurs.

`requestId` is separate retry protection. For the same action, reuse the same UUID and identical arguments; a replay returns the original receipt. With a new UUID, identical normalized current content returns the existing palette ID with `existing:true`. If the original palette was subsequently renamed or revised, saving its old payload may create a new palette. Deleting it also removes the current-content match. There is no `description` parameter on private `save_palette`; public descriptions belong to the explicit publication flow.

For `preview_brand_revision`, provide the saved palette's `id` and current `version`. Inspect the new additive fields:

| status | canSave | Meaning |
| --- | --- | --- |
| `ready-to-save` | true | Feasible changes exist; review before committing. |
| `unchanged` | false | Existing roles already satisfy the minimums; no commit is needed. |
| `infeasible` | false | Requested constraints cannot be met by the adjustment method. |

`feasible:true` means the constraints are satisfied, not that anything changed. Lowering a contrast target does not lighten roles or restore a previous version. `saveMessage` explains the outcome. Commit only a reviewed `canSave:true` result with the exact returned `id`, `version`, complete normalized `options`, `previewHash`, and a fresh request UUID. Re-preview after v1.14.0: measured pair details and persisted targets change the hashed result. An unchanged commit is rejected without creating history or advancing the version.

## Swatch match quality (public v1.4.0)

`match_swatch` and `inspect_color` now return `distanceMetric` and `matchQuality` alongside each nearest catalog result. Pantone ranking retains **CIE76**; RAL retains **CIEDE2000**. These numeric distances are not interchangeable. Existing nearest matches and displayed distances are unchanged.

`success:true` means the lookup completed, not that a close match exists. Inspect `matchQuality.band`, `isCloseMatch`, and `warning` before recommending a swatch. QRCO's conservative review bands are: near ≤2, approximate >2–5, distant >5–10, poor >10. They are product heuristics, not industry acceptance tolerances. Decisions use the unrounded `evaluatedDeltaE`; approximate, distant and poor results carry warnings. Even near matches have `physicalProofRequired:true` for the intended substrate, finish and lighting.

For example, `match_swatch({"hex":"#FF00FF"})` currently finds a nearest Pantone swatch at CIE76 ΔE 30.686, explicitly marked `poor` and `isCloseMatch:false`. Report it as a poor catalog approximation, not a production-ready match. No physical rendering or trademark clearance is implied.

## Repeatable variations (public v1.5.0)

`vary_palette` accepts optional `referenceColors`: retain the original HEX array, in the same count and role order as `colors`, and send it on every call. All alternatives are generated from that reference instead of the latest input. Keeping the same reference, material and chosen variation ID produces the same colors across repeated calls; switching materials or references deliberately changes the result.

Without `referenceColors`, existing relative behavior remains: Soft adds lightness and reduces chroma; repeated Soft/Muted choices can accumulate lightening/desaturation. Each returned palette now includes `variationContext` with `mode`, normalized `sourceColors`, measured `meanLightnessChangeFromSource` (CIELAB L* units, not ΔE), a cumulative-transform warning in relative mode, and a note explaining reference mode. The server cannot infer or remember an omitted original. This does not preserve individual anchors or certify contrast; use `compare_palettes` and explicit pairing checks to review alternatives.

Example: `vary_palette({"colors":["#7D4B3F","#A65F40","#292321","#F3EBDD"],"referenceColors":["#7D4B3F","#A65F40","#292321","#F3EBDD"],"count":3,"material":"digital"})`. For a follow-up, replace `colors` with a result while keeping `referenceColors` unchanged. No `variants` filter is supported; select a returned `id`.

## Brand construction transparency (public v1.6.0)

`build_brand_system` now returns `construction` and `unreferencedColors` outside the saved `system`. Primary defaults to index 0; accent defaults to the next index (wrapping to 0). Explicit `primaryIndex`/`accentIndex` override these choices. Light-mode text and muted text start from the darkest input by relative luminance; dark mode starts from the lightest. Ties use the first input. Other roles derive from these selected sources; light surface is fixed white, and on-colors choose black or white by contrast.

`construction.anchors` records the selected indices. Each entry in `construction.modes` reports its text seed and every role's `origin` (`input`, `derived`, or `constant`), `sourceIndices`, and `matchesInputIndices`. A derived role may equal an original HEX; exact equality is reported separately from how it was generated.

`unreferencedColors` lists input indices not selected as anchors or text seeds across the requested modes. They are retained in the full palette and Studio link, and may happen to equal a derived output. This is not a recommendation to discard them. Generation, contrast checks, exports and the saveable `system` shape are unchanged. Save `result.system` as before; the construction explanation describes this build, not the history of later saved revisions.

## Library totals and history dates (account v1.2.0)

`list_saved_palettes` now returns numeric `totalCount`: all saved palettes owned by the connected account, including both private and published entries, not just the current page. The count and page come from one database statement. Page size remains 25, ordering remains newest creation first with ID as tie-breaker, and `nextOffset` is unchanged. Empty libraries report 0; an offset beyond the end returns an empty page but still reports the full total. Separate calls may observe saves/deletes, so offset pagination is not a frozen cross-call snapshot.

`get_palette_history` keeps `createdAt` in epoch milliseconds and adds `createdAtIso`, an ISO 8601 UTC string ending in Z, for each version summary. Exact-version reads include both fields alongside `palette`, leaving the historical palette object unchanged. These dates describe when that version was saved; they are not necessarily the palette's original creation date. Old history availability limits and ownership checks remain unchanged. The website history API receives the same additive dates. No writes or new permissions are required.

## Current palette dates (account v1.2.1)

`get_saved_palette` now returns top-level `createdAt` / `createdAtIso` for the palette's original creation and `updatedAt` / `updatedAtIso` for its current version's save time. Numeric values are epoch milliseconds; ISO strings are UTC. Compare its `updatedAt` with the current `get_palette_history` summary's `createdAt`, or with an exact history read's top-level `createdAt`. History dates describe individual versions; creation and last-save dates are deliberately distinct. Reads do not modify timestamps, snapshots, roles, versions or publication.

## Version change summaries (account v1.3.0)

`get_palette_history` now includes `changeSummary` in every version summary and alongside `palette` for exact-version reads. It compares that saved snapshot only with version N−1. No palette, timestamp, role or stored history is modified.

The report contains `status`, `comparedToVersion`, readable `text`, and structured `changes` with stable `type` labels. It identifies palette renames, changed color positions (including added/removed positions), color names, locks, metadata fields, brand-system additions/removals, added/removed light or dark modes, and changed roles within existing modes. A new mode is an addition, not a claim that all its roles were edited. HEX arrays are compared by index, so reordering counts as changed positions.

Version 1 reports `initial`. A missing immediately preceding snapshot reports `previous-unavailable`; no comparison is made against a more distant version. `compared` may report no saved content changes. Changes do not establish who edited a palette, why, or whether it improved. Page-boundary comparisons include the already fetched lookahead row, without a query per version. Historical palette objects remain unchanged and user text remains untrusted data.

### History summary status/type contract (account v1.3.1)

Branch on `changeSummary.status` and `changes[].type`, not the readable `text` or `message`.

| Status | Meaning |
| --- | --- |
| `initial` | Version 1; no predecessor comparison. |
| `compared` | Compared with version N−1; an empty changes array means no saved content differences. |
| `previous-unavailable` | Version N−1 is unavailable; no changes are inferred. |

Current change types: `renamed`, `colors-changed`, `color-names-changed`, `locks-changed`, `metadata-changed`, `brand-system-added`, `brand-system-removed`, `mode-added`, `mode-removed`, `roles-changed`, `brand-palette-changed`, `brand-format-changed`, `contrast-target-changed`.

`colors-changed` includes zero-based `indices`, `previousCount`, `currentCount`; name/lock changes include `indices`. `metadata-changed` includes `field`. Mode changes include `mode`; role changes include `mode` and `roles`. Format changes include `from` and `to`. Simple rename/system/palette change entries need no extra fields. Clients should handle an unfamiliar future type by displaying its message rather than dropping the whole summary.

## Interaction-state colors (public v1.7.0)

`generate_state_colors` creates a deterministic draft for an opaque filled control. Required: `baseColor`, `surface` (actual background). Optional: `foreground` (exact text color for enabled states), `focusColor` (exact ring color), `focusAdjacentColors` (up to four additional touching colors), `textMinimum` (4.5–7, default4.5), safe token `prefix` (default `brand`). Surface is always included in focus checks. No modes, hover overrides or saved-system input are accepted.

Default keeps the original HEX. Hover, active/pressed and selected use fixed OKLCH lightness offsets with chroma reduction into sRGB; selected is not a semantic substitute for hover. Without a fixed foreground, each state chooses black or white. Enabled text checks use the requested minimum; fill-to-surface checks use3:1 for uses where the fill conveys the control/state. They are pair measurements, not automatic interface violations or certification. Failed checks stay visible; the base is never silently repaired.

Focus checks only declared adjacent colors. Add the actual control fills when the ring touches them; otherwise an assumed offset gap must really exist. Automatic focus checks the base then257 lightness samples, selecting a passing candidate closest in OKLCH lightness; this is bounded search, not global optimization. Supplied focusColor is unchanged even when it fails. If no candidate passes all listed surfaces, focus.color is null, status is unresolved, the summary fails overall, and no focus token is exported. Rendered focus area, occlusion and same-pixel focused/unfocused contrast are outside the tool's scope.

Disabled text/surface checks are informational: `minimum:null`, `passes:null`, excluded from summary totals. This exemption applies only to genuinely inactive controls. Colors are opaque mixtures for the specified surface, not CSS opacity. Between-state contrast is also informational; use labels/icons and real interaction testing for distinguishability.

The result returns `states`, `focus`, a measured `summary`, warnings, CSS custom properties and Tailwindv4 `@theme` tokens. Exports contain draft tokens even when checks fail (except unresolved focus); inspect summary before using them. No account save, publication or existing brand role is changed.

Example: `generate_state_colors({"baseColor":"#A65F40","surface":"#FFFFFF","focusAdjacentColors":["#A65F40"],"prefix":"copper"})`.

Sources: [text contrast and inactive controls](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html), [non-text contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html), [focus appearance](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html). Adjacent-color contrast concerns1.4.11;2.4.11 concerns occlusion. HEX pairs alone do not establish2.4.13 focus appearance compliance.


## Color-vision review (public v1.8.0)

`check_color_vision` simulates a palette and measures pair separation in the resulting display colors. Required `colors`: 2–10 opaque 3/6-digit HEX strings (either case, optional #). Optional `names`: exactly one nonempty label per color, max100 characters. No saved-library permission is needed; it never saves, changes colors or modifies brand roles.

- `types`: unique subset of `protanopia`, `deuteranopia`, `tritanomaly`; default all three. Fixed Machado severity1. The last is explicitly **severe tritanomaly approximation, not tritanopia**. `tritanopia`, achromatopsia and custom severity are unsupported and rejected.
- `reviewThreshold`: 1–20, default5. Flags use **unrounded simulated CIEDE2000 < threshold**. This is a QRCO review heuristic, not a validated threshold for CVD discrimination or a WCAG requirement.
- `includeAllPairs`: defaultfalse. Always returns `flaggedPairs`; true also returns every `pairs` measurement (max45 per simulation).

`original` preserves normalized HEX, order and names. Each `cvdChecks` entry carries its type/label/severity, `simulatedColors` with clipping diagnostics, pair counts and flagged pairs. Pair indices are zero-based input positions; results stay in index order. Measurements include `originalDeltaE2000`, `simulatedDeltaE2000` and `distanceChange` (simulated minus original; negative means reduced separation), rounded to four decimals for display. Flags use raw values.

Full `pairs` entries carry `flagged` (boolean) and `reason`; `flaggedPairs` is the subset with `flagged:true`, retaining the same fields. There is no `flag` field. Pair `reason`: `original-duplicate` for identical original HEX; `already-close` for a flagged pair below threshold before simulation; `newly-close` for a flagged pair that crosses below threshold; `not-flagged` otherwise. `identicalSimulatedHex` identifies equality after display rounding. Summary `flaggedPairEvaluationCount` sums across simulations, so a pair can be counted three times. No pass/fail or whole-palette accessibility verdict is returned. No flags does **not** guarantee distinguishability.

Method: decode sRGB to linear RGB, apply published Machado/Oliveira/Fernandes matrices, clip channels to [0,1], encode and round to 8-bit HEX. `gamutClipped`/`clippedChannels` report out-of-range channels, including tiny matrix precision overshoots. Clipping and rounding can reduce separation. CIEDE2000 uses CIELAB D65 and standard weights on final displayed HEX; this metric is not validated as a measure of what a person with CVD can discriminate. Simulations approximate selected conditions and cannot reproduce every person's experience.

Use labels, icons, patterns or position for meaning. Run `check_contrast` separately on original text/background colors. No contrast-under-simulation WCAG claims, physical material predictions or automatic fixes. Existing contrast and brand-system tools are unchanged; these checks are a separate public tool, not embedded in saved reads.

Example: `check_color_vision({"colors":["#FF0000","#00AA00","#FFFFFF"],"names":["Stop","Go","Paper"],"includeAllPairs":true})`. Red/green becomes a newly-close pair under deuteranopia (simulated ΔE2000 about2.5798); use additional cues.

Sources: [Machado et al. paper](https://doi.org/10.1109/TVCG.2009.113), [published matrix values via Colour Science](https://raw.githubusercontent.com/colour-science/colour/develop/colour/blindness/datasets/machado2010.py), [Colour's documented tritanomaly limitation](https://colour.readthedocs.io/en/latest/_modules/colour/blindness/machado2009.html).


## Correction: legacy accessible generation (public v1.8.1)

`generate_palette` still accepts `accessible:"aa"` or `accessible:"aaa"` for compatibility. These now mean a **normal-text contrast target for consecutive palette pairs**: [0,1], [1,2], … (no wraparound), at4.5:1 or7:1 respectively. They do not certify a palette, arbitrary pairings, CVD accessibility or a finished interface. `generate_palette_set` does not expose this parameter; do not invent it. The backend generation/set/random paths share the correction when this mode is selected.

The old unconditional `metadata.accessible:true` has been removed. `metadata.accessibility` remains the requested level, never evidence of success. Read `metadata.contrastTarget`:

- `scope:"adjacent-pairs"`, `requestedLevel`, `minimum`, `status:"met"|"unmet"`, `allDeclaredPairsPass`, `pairCount`, `failureCount`.
- `pairs`: exact final colors, zero-based indices, displayed four-decimal ratio, target minimum and `passes` determined from the **unrounded** ratio.
- `lockedIndices`, `adjustedIndices`, bounded `search` details and explicit limitations.

Generation first creates seed colors and applies material styling. Exact locks are then restored. If the final adjacent pairs fail, QRCO searches two alternating dark/light directions, mixing unlocked colors toward black/white in256 bounded sRGB steps per direction. It takes the first passing step, testing dark-first before light-first at ties. This is not a minimum-change optimizer and adds no hidden contrast buffer. See optional contrastMinimum below to request a higher target. A passing result can be very close to the target; recheck after downstream color changes. With no locks, alternating black/white endpoints guarantee a passing candidate. Locks can leave a target unmet; no passing candidate means the post-material colors and exact locks are retained, `status:"unmet"`, and actual failures remain visible. No global infeasibility claim is made. Adjustments can change the material style's appearance.

Always choose the actual foreground/background pairing and check it with `check_contrast`. API `success:true` means generation completed, not that a locked contrast target was achieved. Old saved or published palettes are not rewritten. Short HEX locks now normalize correctly instead of being silently skipped.

Example: `generate_palette({"accessible":"aaa","size":5,"material":"fabric"})`; inspect all four declared pairs. Adversarial example: `generate_palette({"accessible":"aaa","size":2,"lockedColors":[{"index":0,"hex":"#777777"},{"index":1,"hex":"#777777"}]})` must preserve both locks, report ratio1 and statusunmet, with no blanket accessible flag.

Threshold references: [W3C normal-text AA](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html), [W3C normal-text AAA](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html). These requirements concern text/background usage, not palette certification.


## Optional contrast headroom (public v1.9.0)

`generate_palette`, `vary_palette` (v1.10.0), and measurement-only `export_palette` (v1.11.0) accept optional numeric `contrastMinimum` together with `accessible`. Example: `{"accessible":"aa","contrastMinimum":4.8,"size":5}` or `{"accessible":"aaa","contrastMinimum":7.5,"size":5}`. Minimum allowed is4.5foraa or7foraaa; maximum21. Missing accessible, a lower target, conflicting generation modes and invalid numbers are rejected. No custom headroom parameter on `generate_palette_set` or `random_palette`; those defaults are unchanged.

Omitting the option preserves existing generation exactly. With it, the existing bounded search targets the requested ratio on final adjacent pairs, after material styling and exact-lock restoration. The first passing candidate is used; no additional hidden buffer is added. Larger targets can change colors more or leave a locked target unmet. This is not a color-drift guarantee; remeasure downstream colors and actual text/background usage.

`metadata.contrastTarget.minimum` and pair `minimum`/`passes` now refer to the chosen generation target (unchanged when omitted). Additive fields distinguish the standard from the optional target:

- Report `standardMinimum` (4.5or7), `requestedHeadroom` (chosen minimum minus standard), `allStandardPairsPass`.
- Each pair: `standardMinimum`, `meetsStandard`, `marginAboveStandard`, `marginAboveTarget`.
- Margins are signed **contrast-ratio differences**, not percentages, rounded to four decimals for display. All pass decisions use unrounded ratios. A displayed zero margin can hide a tiny positive/negative value; use the booleans for the verdict.

Example with exact locks #767676/#FFFFFF and aa/contrastMinimum4.8: measured4.5422, `meetsStandard:true`, `passes:false`, standard margin+0.0422, target margin−0.2578, report `status:"unmet"` and `allStandardPairsPass:true`. Both colors remain exact. Standard compliance and requested headroom are deliberately separate. Saved palettes are not rewritten.


## Contrast-aware variations (public v1.10.0)

`vary_palette` accepts optional `accessible:"aa"|"aaa"` and `contrastMinimum` using the same adjacent-pair contract as generation. Supply them explicitly on **every call**; input HEX and referenceColors do not carry a prior target. The original defaults and variation operators remain unchanged when omitted.

Each variation returns `contrastContext` and `contrastTarget`:

- Without accessible: contrastContext.status=`not-requested`, inherited=false, an explicit warning that prior AA/AAA/headroom can be broken, and contrastTarget=null. Audit actual pairs before use; no implied carry-forward.
- With accessible: contrastContext.status=`explicit-target` (a request marker, not a verdict), inherited=false, and a newly measured contrastTarget report. Read its met/unmet status, exact final pairs and standard/target margins.
- Apply variation, then material styling, then bounded contrast re-targeting. This may weaken the intended style or make alternatives identical, especially at21:1. There is no uniqueness or aesthetic-preservation guarantee.
- contrastMinimum requires accessible and must lie between the selected standard (4.5/7) and21. Omit it for the standard target. Returned checks apply only to consecutive pairs, never all palette pairs or a whole interface.
- Fixed-reference stability requires the same referenceColors, material, operation **and contrast settings** each time. Contrast checks describe the final result, including re-targeting.

Example: `vary_palette({"colors":["#000000","#959595","#000000","#959595"],"accessible":"aaa","contrastMinimum":7.5,"count":3})`. This source passes7:1, while an ordinary un-targeted Soft variation can fail it; the explicit target is re-evaluated after the variation. No account saves.

The MCP variation tool does not accept locks. Its backend API's existing locked/keepLocked option preserves exact locks when an explicit target is requested; impossible targets remain unmet.

## Cross-tool contrast boundaries

A palette's adjacent-pair target is not inherited by other HEX-only tools. `audit_palette` tests all pairs; a failed non-adjacent pair does not contradict a met adjacent-pair target. `build_brand_system` derives different roles and checks its own listed4.5text/3UI pairings, not the input palette's AAA/headroom target. `export_palette` now optionally remeasures and embeds an explicitly supplied target (v1.11.0); omitted targets are not inherited. Public v1.14.0 adds explicit persisted textContrast/uiContrast; see below.

`random_palette` defaults to five colors when count is omitted (the old description saying random size was incorrect). A target is a requested contract; always inspect the measured verdict.


## Contrast context in exports (public v1.11.0)

`export_palette` accepts optional `accessible:"aa"|"aaa"` and `contrastMinimum`. Supply the requested target explicitly, as with generation/variation. Minimum4.5foraa or7foraaa, maximum21; contrastMinimum alone, lower targets and forged contrastTarget objects are rejected. The exporter **measures, never repairs**. Passing generation results and stale reports are not trusted or inherited from HEX.

The response includes `contrastContext` (`explicit-target` or `not-requested`, inherited:false) and `contrastTarget` (report or null). These are separate: explicit-target means a target was supplied, not that it passed. No option preserves existing artifact bytes; its outer warning says no contrast report was embedded.

With a target, the exact same freshly measured report travels in the content:

- JSON: top-level `contrastTarget` alongside name and colors.
- CSS, SCSS, Tailwind v4: a leading block comment with a short warning and the report as formatted JSON.
- SVG: XML-escaped JSON text inside `<metadata id="qrco-contrast-target">`.

Report schemaVersion1 carries requestedLevel, standardMinimum, minimum, requestedHeadroom, scopeadjacent-pairs, met/unmet status, standard/target summaries and pair counts. Every pair includes zero-based indices, exact exported token identifiers and HEX, ratio, raw-decision standard/target booleans and signed margins. There is no last-to-first/non-adjacent guarantee. `colorChangesApplied:false` makes preservation explicit. Ratios/margins display four decimals; decisions use raw values.

Token identifiers are the palette keys (e.g. brand-1); CSS adds --, SCSS adds $, Tailwind adds --color-. Names and prefixes keep existing escaping/validation; arbitrary labels are excluded from CSS comment metadata. All exports preserve original normalized colors, order and labels. Failed targets remain embedded failures, even when the standard itself passes.

Example: `export_palette({"colors":["#767676","#FFFFFF"],"names":["Ink","Paper"],"accessible":"aa","contrastMinimum":4.8,"format":"css","prefix":"proof"})`. It exports the unchanged colors with ratio4.5422: standard passes, requested4.8fails, statusunmet, margins+0.0422/−0.2578. There is no recoloring to make the report pass.

The report is handoff documentation, not runtime enforcement, a signature or accessibility certification. Comments/metadata may be removed by build tools or downstream applications; preserve the JSON export when a durable sidecar is needed. Recheck actual use and any edits. This v1.11.0 update did not add Adobe/Figma formats; v1.12.0 adds portable ASE/DTCG exports described below. Direct synchronization remains separate.

## Portable Adobe and Figma exports (public v1.12.0)

`export_palette` adds `format:"ase"` and `format:"dtcg"`. Both preserve input order and normalized RGB colors; neither writes files, saves palettes, syncs applications, or invents brand roles/modes. Existing five formats retain their behavior.

- **Adobe ASE:** `content` is **base64**, `encoding:"base64"`, `fileName:"<prefix>.ase"`, with a byteLength. Decode it to binary before saving. Ordered normal RGB swatches retain custom names (including Unicode); no CMYK conversion, ICC profile or spot-ink/Pantone specification. Import through the receiving Adobe application's swatch-library controls. Duplicate labels are retained in the file; apps may rename or merge them.
- **Figma/DTCG:** `content` is UTF-8 JSON, `encoding:"utf-8"`, `fileName:"<prefix>.tokens.json"`. Uses DTCG 2025.10 color tokens with sRGB components, alpha1 and HEX. Safe prefix-index keys become variable names; custom labels become `$description`, so duplicate names do not collide. In Figma Variables, create a new collection and drag this one JSON file into the Variables view to import one mode. Importing into an existing mode can replace matching variables.
- **Measured context:** optional accessible/contrastMinimum still measures exact adjacent token pairs without changing colors. DTCG carries context in `$extensions["ca.qrco"]`. ASE cannot carry the report: its `sidecar` contains fileName, mimeType, encoding and UTF-8 JSON content with tokens and contrast context. Keep that JSON alongside the ASE, including any failed target. With no target the context explicitly says not-requested. Importers may discard extensions/descriptions; retain the source report.

Example: `{"colors":["#767676","#FFFFFF"],"names":["Ink","Paper"],"prefix":"brand","format":"ase","accessible":"aa","contrastMinimum":4.8}` returns unchanged gray/white swatches and a sidecar reporting the unmet 4.8 target. Use `format:"dtcg"` for the matching token file.

Format and import references: [Adobe ASE exchange](https://helpx.adobe.com/uk/illustrator/desktop/manage-colors/use-swatches/share-swatches-between-applications.html), [Figma token import](https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables), [DTCG color format](https://www.designtokens.org/tr/2025.10/color/). Files are tested structurally and ASE with an independent decoder; actual Adobe/Figma application import remains a manual acceptance step. Direct sync and saved brand-system mode export remain later work.

### HEX contracts (public v1.12.0; account v1.3.2)

Published JSON Schema patterns explicitly include uppercase and lowercase HEX, matching existing runtime behavior. Public color arguments retain optional # and 3/6 digits; account save_palette.colors still requires # and six digits. No case/canonicalization or account permission changes.

`search_explore` rejects malformed hash-prefixed queries such as `#GG0000` or `#1234` before catalog retrieval. Valid HEX matches exact normalized colors; other queries still match literal palette names. An empty result for a valid literal name is not a semantic search verdict.

## State and audit diagnostics (public v1.13.0)

`generate_state_colors` now adds explicit warnings naming enabled states whose text or fill-to-surface checks fail. If all four enabled fills fail, the warning says so. These are draft colors; warnings do not recolor them or change export tokens, thresholds, existing pass flags, or totals. A failed fill check matters when that fill identifies the control/state; borders, labels and actual geometry still affect applicability. Disabled checks remain informational and excluded.

Each text, surface and focus check includes `standardMinimum`, `marginAboveStandard` and `marginAboveTarget`. The standard floor is4.5 for normal text and3 for surface/focus pairs; the target is the check's `minimum` (textMinimum can be higher). Signed margins are unrounded-ratio differences displayed to four decimals, not percentages or tolerance guarantees. Disabled standard/margins are null. Brand/link tools do not gain margins in this update.

`focus.fillDiagnostics` compares the chosen ring with each enabled default/hover/active/selected fill: state, fill, ratio, declaredAdjacent, potentialConflict and informationalOnly. potentialConflict means the raw ratio is below3 **if those colors touch**. Undeclared comparisons are informational, excluded from totals; they do not assume layout or downgrade `passes-listed-pairs`. A warning identifies undeclared fills needing review. Add actual touching colors to focusAdjacentColors; verify a real offset/alternative indicator otherwise. Already-declared colors are measured once in focus.checks, not double-counted. Unresolved focus has an empty diagnostic list and still exports no focus token.

`audit_palette` now returns `scope:"all-pairs"` and `duplicates:[{hex,indices}]` (normalized HEX, zero-based indices). Colors and ratio-1 duplicate pairs remain present. Its note explains raw-threshold decisions and that failing non-adjacent pairs do not contradict a separate adjacent-only generation target. The audit receives no prior target and does not claim to have verified one.

## Persisted brand-role targets (public v1.14.0; account v1.4.0)

`build_brand_system` accepts optional numeric `textContrast` (4.5–7; use7 for normal-text AAA on the listed pairs) and `uiContrast` (3–4.5 for listed border/focus pairs). These names match revision options. This tool does not accept accessible, contrastMinimum, or a caller-supplied contrast report. Prior palette targets cannot be inferred from HEX.

Example: `{"colors":["#123456","#FEDCBA"],"mode":"both","textContrast":7,"uiContrast":4.5}`. Palette colors, primary/accent and generated backgrounds stay fixed; supporting roles may move toward black or white. A successful explicit request returns `feasible:true`, `canSave:true` and a saveable `system` with `contrastTarget:{textMinimum:7,uiMinimum:4.5}`. The persisted object is only a constraint, not a trusted pass label. Save validation independently measures every listed role pair against it. Copy the complete system to save_palette. No database migration or automatic update of existing saved palettes occurs.

The outer `contrastTarget` is the freshly measured report: source, scope listed-brand-role-pairs, targets, met/unmet status, counts and exact pair measurements. CSS/Tailwind include this report as a comment, including default-floor context on legacy systems. Pair fields include exact role colors and signed margins above the AA/UI floor and requested target. Comments can be stripped downstream; preserve the source JSON. Studio links, saved metadata and account reads retain the constraint. Studio previews/exports use it; rebuilding in Studio retains an existing target and refuses invalid results.

**Infeasible request:** copper #A65F40 as primary cannot support7:1 button text while staying fixed. The builder returns `feasible:false`, `canSave:false`, measured pairings/failures and `system:null`, `css:null`, `tailwind:null`, `studioUrl:null`, `usageChecks:null`. It does not return a lower-target system as though the request succeeded. This is a bounded search result, not a proof that every alternative design is impossible.

`usageChecks` remains separate: primary/accent used as links are tested against the system's text target (or legacy4.5). Link failures remain visible even when all role pairs pass; suggestions stay `applied:false` and never become saved roles or replace anchors. Brand and link pair checks report signed margins; no interface, geometry, CVD or physical-print certification.

### Revision and history contract

- Omitted preview text/UI targets inherit the saved constraint; legacy systems use4.5/3. Partial requests preserve the omitted saved target. Normalized options still contain all four fields required at commit.
- Successful raised revision targets are persisted. Legacy no-op/default-floor previews remain unchanged. Explicit builder defaults are also persisted when supplied.
- `changes` remains a list of role changes. New `targetChange` separately carries before/after constraints (null before means legacy defaults). A target-only change can be `ready-to-save` with `changes:[]` and `canSave:true`; review it before committing. Lowering a persisted target changes the contract, not the colors.
- Unchanged constraints and roles yield canSave:false. Infeasible revisions still yield no saveable system. Commit requires the exact current version, normalized options and previewHash; **refresh previews made before this release**, because their measured result/hash changed.
- History adds machine type `contrast-target-changed`, with before/after values. Existing historical snapshots and publication snapshots are not rewritten. Save/revise remains private and never republishes automatically.
- Existing v1 light/v2 dark-or-both system formats remain accepted. Optional contrastTarget is strictly validated; forged status fields, omitted numeric target members and failed role pairs are rejected. Legacy saved systems are not retroactively claimed to meet stronger targets.

## Full tonal-scale exports (public v1.15.0)

`export_palette` accepts **2–12 colors** and matching optional names in all seven formats: CSS, SCSS, Tailwind, JSON, SVG, ASE and DTCG. Pass `generate_tonal_scale.colors.map(c => c.hex)` for all11stops; optionally prepend `original.hex` for12colors. Supply stop labels as names to preserve their meaning. Input order, exact HEX, custom names and duplicates are retained; export keys still use prefix-index, not the tonal generator's stop-number keys. The tonal tool's own CSS/Tailwind retains its original stop keys.

Optional contrast context measures each consecutive exported pair: 10 checks for11colors,11 for12. A tonal ramp is not expected to make every neighboring shade suitable for text; failed targets remain failures and colors are never repaired. ASE retains its JSON sidecar; DTCG retains context in its extension.

This is an export-only limit increase. Studio links, saving, brand systems and palette audits retain their existing10-color limit. Choose a meaningful subset there. Thirteen colors, mismatched labels and invalid HEX still reject. Existing2–10-color artifact content is unchanged.

## Text contrast cushion in new brand builds (public v1.16.0)

`build_brand_system` now prefers an extra **0.3 contrast ratio** for listed text roles: 4.8 at the default 4.5 floor, or 7.3 for an explicit 7 target. This is a QRCO generation preference, not a new WCAG requirement. It adjusts only new `text`/`mutedText` roles, searching 255 steps toward black and white and choosing the least squared RGB movement among passing samples. Palette colors, primary/accent, surfaces, UI roles and maximum-contrast black/white on-colors stay fixed. Already sufficient text is unchanged.

The top-level `textCushion` reports `minimum`, `preferredMinimum`, `additionalRatio:0.3`, `status:met|partial`, `shortfallCount`, exact pair colors, measured ratios, raw-derived `meetsPreferred`, signed margins and changes. An unavailable cushion is explicit, including fixed anchors whose best ink cannot reach it. A bounded search miss keeps the valid original text and reports a shortfall; it does not declare global impossibility. Actual requested-target failure still produces `feasible:false`, null system/exports and `textCushion:null`.

The saved numeric target remains the user's requested floor, not the preferred cushion (`persistedAsConstraint:false`). Generated role colors survive save/link/export as usual; the derived cushion report is outside the saved system. Existing saved reads, account revisions and Studio rebuilds are unchanged and do not automatically apply this preference. Link usage remains separate. Raw threshold decisions are unchanged: 4.5004 really passes a 4.5 requirement, but has little margin. The extra margin is not a guarantee under screen brightness changes, glare, later color edits, transparency, printing or all viewing conditions.
