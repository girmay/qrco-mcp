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
