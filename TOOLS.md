# QRCO MCP tool reference

[Setup](README.md#connect-in-your-ai-application) · [Examples](EXAMPLES.md) · [Live reference](https://qrco.ca/mcp-guide.md)

## Public endpoint: https://qrco.ca/mcp

No authentication; Streamable HTTP. Fifteen tools.

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

## Pantone option by tool

`includePantone` is accepted by `generate_palette` and `inspect_color`, but **not** by `generate_palette_set`. Omit it from batch-generation calls; unknown arguments are rejected. To obtain a nearest Pantone catalog match for a chosen batch color, call `match_swatch` with its `hex`. This documents the current schemas; it does not add batch matching or certify physical print results.
