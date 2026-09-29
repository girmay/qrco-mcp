# QRCO MCP tool reference

[Setup](README.md#connect-in-your-ai-application) · [Examples](EXAMPLES.md) · [Live reference](https://qrco.ca/mcp-guide.md)

## Public endpoint: https://qrco.ca/mcp

No authentication; Streamable HTTP. Sixteen tools.

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

This is QRCO’s recipe, not Tailwind’s built-in palette or equal-perceptual-distance steps. Stop 500 need not equal the seed. Measurements use final 8-bit colors and unrounded thresholds; no complete-accessibility guarantee. The full 11-stop scale exceeds Studio’s 10-color limit: choose a subset before palette tools. No account save, publication or network calls.

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

`preview_brand_revision` requires an existing saved `metadata.brandSystem`. Preview options may be omitted or partial; the result returns their complete normalized form. **Saving requires all four returned option fields** (`addMissingMode`, `textContrast`, `uiContrast`, `lockedRoles`), plus the preview's `id`, saved-palette `version`, `previewHash`, and a fresh UUID `requestId`. Copy them; do not rebuild options from the initial request. Only save a feasible preview with changes and user approval. Changed inputs or palette versions require a new preview. Reuse identical arguments/requestId only to retry the same save. No publish occurs.

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

`feasible:true` means the constraints are satisfied, not that anything changed. Lowering a contrast target does not lighten roles or restore a previous version. `saveMessage` explains the outcome. Commit only a reviewed `canSave:true` result with the exact returned `id`, `version`, complete normalized `options`, `previewHash`, and a fresh request UUID. Old preview hashes remain compatible because readiness fields are outside the hashed revision result. An unchanged commit is rejected without creating history or advancing the version.

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

Current change types: `renamed`, `colors-changed`, `color-names-changed`, `locks-changed`, `metadata-changed`, `brand-system-added`, `brand-system-removed`, `mode-added`, `mode-removed`, `roles-changed`, `brand-palette-changed`, `brand-format-changed`.

`colors-changed` includes zero-based `indices`, `previousCount`, `currentCount`; name/lock changes include `indices`. `metadata-changed` includes `field`. Mode changes include `mode`; role changes include `mode` and `roles`. Format changes include `from` and `to`. Simple rename/system/palette change entries need no extra fields. Clients should handle an unfamiliar future type by displaying its message rather than dropping the whole summary.

## Interaction-state colors (public v1.7.0)

`generate_state_colors` creates a deterministic draft for an opaque filled control. Required: `baseColor`, `surface` (actual background). Optional: `foreground` (exact text color for enabled states), `focusColor` (exact ring color), `focusAdjacentColors` (up to four additional touching colors), `textMinimum` (4.5–7, default4.5), safe token `prefix` (default `brand`). Surface is always included in focus checks. No modes, hover overrides or saved-system input are accepted.

Default keeps the original HEX. Hover, active/pressed and selected use fixed OKLCH lightness offsets with chroma reduction into sRGB; selected is not a semantic substitute for hover. Without a fixed foreground, each state chooses black or white. Enabled text checks use the requested minimum; fill-to-surface checks use3:1 for uses where the fill conveys the control/state. They are pair measurements, not automatic interface violations or certification. Failed checks stay visible; the base is never silently repaired.

Focus checks only declared adjacent colors. Add the actual control fills when the ring touches them; otherwise an assumed offset gap must really exist. Automatic focus checks the base then257 lightness samples, selecting a passing candidate closest in OKLCH lightness; this is bounded search, not global optimization. Supplied focusColor is unchanged even when it fails. If no candidate passes all listed surfaces, focus.color is null, status is unresolved, the summary fails overall, and no focus token is exported. Rendered focus area, occlusion and same-pixel focused/unfocused contrast are outside the tool's scope.

Disabled text/surface checks are informational: `minimum:null`, `passes:null`, excluded from summary totals. This exemption applies only to genuinely inactive controls. Colors are opaque mixtures for the specified surface, not CSS opacity. Between-state contrast is also informational; use labels/icons and real interaction testing for distinguishability.

The result returns `states`, `focus`, a measured `summary`, warnings, CSS custom properties and Tailwindv4 `@theme` tokens. Exports contain draft tokens even when checks fail (except unresolved focus); inspect summary before using them. No account save, publication or existing brand role is changed.

Example: `generate_state_colors({"baseColor":"#A65F40","surface":"#FFFFFF","focusAdjacentColors":["#A65F40"],"prefix":"copper"})`.

Sources: [text contrast and inactive controls](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html), [non-text contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html), [focus appearance](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html). Adjacent-color contrast concerns1.4.11;2.4.11 concerns occlusion. HEX pairs alone do not establish2.4.13 focus appearance compliance.
