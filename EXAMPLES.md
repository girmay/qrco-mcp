# QRCO MCP example workflows

Use these prompts in your AI assistant after connecting QRCO. The assistant chooses the tool arguments; natural-language interpretation belongs to the assistant, while QRCO provides generation, measurements, storage and exports.

## From a brief to design tokens

> My client wants their brand to feel expensive, warm, intelligent and slightly unconventional. Create three palette directions and explain the trade-offs. After I pick one, build light and dark brand roles, report measured contrast, and return CSS and Tailwind CSS v4 tokens plus a QRCO Studio link.

Public tools: `generate_palette_set`, `build_brand_system`, `create_studio_link`. Generation returns up to five directions in one call. Use `compare_palettes` for measured comparison and `audit_palettes` for batch checks; batch export is not yet offered.

## Keep a brand anchor

> Keep #A65F40 exactly as the primary brand color. Create a five-color palette around it for a thoughtful consultancy. Give each color a memorable name and explain its role. Return an editable Studio link with those names.

Public tools: `generate_palette` with a fixed color, then `create_studio_link`. Opening the link does not save to an account.

## Work from your actual library

> List my saved palettes, following pagination. Let me choose one, then show its colors, custom names, saved brand roles and measured contrast. Do not save or publish anything.

Account tools: `list_saved_palettes`, `get_saved_palette`. Requires `palettes:read`.

## Preview a revision, then save deliberately

> On the saved brand system I selected, preview adding the missing dark or light mode. Preserve its palette colors and existing anchors. Show the exact role changes and contrast results. Stop before saving.

Account tool: `preview_brand_revision`. Requires `palettes:read`. The palette must already contain a saved brand system.

After reviewing a feasible preview:

> Save this exact reviewed revision as a new version. Keep its public page unchanged.

Account tool: `save_brand_revision`. Requires `palettes:revise`, the reviewed options, expected version and `previewHash`. Changed palette versions or options require a fresh preview. Reuse the same request ID and arguments when retrying the same write.

## Inspect history

> Show tracked versions of this saved palette, including older pages. Retrieve the earlier version I choose so we can compare it with the current system. Do not overwrite anything.

Account tool: `get_palette_history`. Requires `palettes:read`. Retrieving an old snapshot is read-only; it does not restore it as the current saved version.

## Publish a chosen palette

> Publish the saved palette I selected with this description: “A quiet balance of copper warmth and cool, thoughtful greens.” Use its current color names and saved roles, and return the public page link.

Account tool: `publish_palette`. Requires `palettes:publish` and explicit user instruction. Unsaved palettes need a separate private save first with `palettes:save`. Review any agent-drafted description before publication. Private editing never silently republishes the snapshot.

## Compare real choices and audit them together

> Compare these three palettes for a light website on #FFFFFF. Keep #A65F40 as an exact brand anchor. Show which swatches meet 4.5:1 as text on white, which colors changed from the first direction, and any near-duplicates. Explain the trade-offs without assuming every color must work as body text.

Public tool: `compare_palettes`. Supply a unique ID for each palette, `backgrounds: [{id: "page", color: "#FFFFFF", minimum: 4.5}]`, and `anchors: ["#A65F40"]`. Unordered comparison suits unassigned palettes; use indexed only when corresponding positions have the same roles.

> Audit these approved directions together. In each palette, color 0 is button text and color 1 is its fill. Require 4.5:1 for that pairing. Return failures, keep the palette IDs, and do not save anything.

Public tool: `audit_palettes`, with each palette's `pairings: [{foregroundIndex: 0, backgroundIndex: 1, minimum: 4.5}]` and `detail: "failures"`. Full-palette contrast opportunities are diagnostic; specified uses determine which failures matter.

## Build a tonal range without losing the original

> Build a 50–950 tonal scale from #7D4B3F using the token prefix roast. Keep that exact original color as its own token. Check every stop as text on white and #17141D. Show which stops meet 4.5:1, flag any gamut adjustments, and return CSS plus Tailwind CSS v4 tokens. Do not save anything.

Public tool: `generate_tonal_scale` with `seed: "#7D4B3F"`, `prefix: "roast"` and `backgrounds: [{id: "paper", color: "#FFFFFF", minimum: 4.5}, {id: "ink", color: "#17141D", minimum: 4.5}]`. The original stays `roast-original`; stop 500 is not assumed to match.

For a smaller range, request `stops: [100, 300, 500, 700, 900]`. Those values match the corresponding stops in the full scale. Select at most 10 colors when creating a Studio palette link; the complete 11-stop scale is intended for token exports.

## Exact private save → preview → commit data flow

The variables below represent parsed JSON tool results, not literal strings to send. Account scopes: `palettes:read palettes:save palettes:revise`. No publication scope is needed. Ask the user before saving a new palette or committing a revision.

```js
// Public build_brand_system response: { system, pairings, css, ... }
const saveArguments = {
  requestId: freshUUID(),
  name: "My reviewed brand system",
  colors: built.system.palette,
  brandSystem: built.system
};
// saved = result of account save_palette(saveArguments)
const previewArguments = {
  id: saved.id,
  version: saved.version,
  options: { addMissingMode: true }
};
// preview = result of preview_brand_revision(previewArguments)
// Review changes and failures. Proceed only if preview.canSave === true and approved.
const commitArguments = {
  id: preview.id,
  version: preview.version,
  options: preview.options, // complete returned object, including defaults
  previewHash: preview.previewHash,
  requestId: freshUUID()
};
// committed = result of save_brand_revision(commitArguments)
// Read get_saved_palette and get_palette_history to verify.
```

For this example, build a light-only system first; a system already containing both modes may have nothing to change. Keep `commitArguments` unchanged when retrying that exact commit. Omitting `options` now fails input validation; an altered preview hash/options still cannot commit.

## Verify a brand color as link text

> Build a dual-mode brand system from #767676 and #FEDCBA. Show the existing role pairings and the separate usageChecks. Can the original primary serve as normal-sized link text on the page and card surfaces in each mode? Explain failures, show any suggested link shades with their measured checks, and confirm they are not applied to the saved system or exports. Do not save or publish.

Public tool: `build_brand_system`, `colors: ["#767676", "#FEDCBA"]`, `mode: "both"`. Expect the original primary to pass on white yet fail on the tinted light page; #747474 is the separate light-mode suggestion. The dark-mode suggestion is #7C7C7C. Original primary stays #767676 everywhere in `system`, CSS, Tailwind and Studio metadata. Account `get_saved_palette` returns the same `usageChecks` for a valid saved system, read-only.

## Keep a named family without losing explicit locks

> Generate five colors with baseColor red. Verify metadata.baseColorConstraint and show the actual HEX values. Then repeat with material fabric and a green #00FF00 locked at index 1. The green must remain exact and be reported as a family exception; all other colors must stay within the returned red family bounds. Do not save or claim these colors are automatically safe for alerts.

For `generate_palette`, pass `size: 5`, `baseColor: "red"`; second call adds `material: "fabric"`, `lockedColors: [{index: 1, hex: "#00FF00"}]`. For three family-constrained options, use `generate_palette_set` with `paletteCount: 3`, `colorsPerPalette: 5`, `baseColor: "red"`.

## Recognize a no-op revision before committing

> Read my selected brand system and current version. Preview the baseline textContrast 4.5 and uiContrast 3. Report status, canSave and saveMessage. If status is unchanged, tell me the existing roles already meet the targets and stop; do not attempt a save. Lowering targets should not be presented as restoring older colors.

This is read-only. `feasible:true` and `canSave:false` is a valid unchanged outcome.

## Reject a poor catalog approximation

Ask: “Find the nearest Pantone for #FF00FF and inspect its RAL approximation. Report the metric, distance, match-quality band and warning. If it is poor, say so; don't treat lookup success as proof that it is close.”

Use `match_swatch({"hex":"#FF00FF"})` and `inspect_color({"hex":"#FF00FF","includePantone":true})`. Public v1.4.0 provides `distanceMetric` and `matchQuality`; the Pantone assessment should agree between tools. The catalog lookup still succeeds when no close match exists. Bands are QRCO review heuristics; physical proofing remains necessary even for a near result.

## Explore without cumulative whitening

Ask: “Keep these original colors as referenceColors while we explore Soft, Bold and Muted alternatives. Reuse the original reference on every vary_palette call, even if colors contains our latest selection. Compare changes against the original and recheck our actual text/background pairings.”

Public v1.5.0 adds fixed-reference generation. It is stateless: omit referenceColors and transforms apply to the latest input, with a warning in variationContext. Reference mode deliberately regenerates alternatives from the original instead of accumulating edits.

## Explain where each brand role came from

Ask: “Build both modes from my palette. Explain the primary/accent selections, darkest/lightest text seeds and derived roles using construction. List unreferencedColors without dropping them from my palette.”

Public v1.6.0 reports source indices and exact HEX matches separately. Save the unchanged result.system object, not the construction report.

## Count a library and read its history dates

Ask: “Tell me my saved palette total using list_saved_palettes.totalCount. For a palette with history, show its latest version dates using createdAtIso. Read only; do not change anything.”

Account v1.2.0 returns the full total even on an empty page. Follow nextOffset to inspect every palette; separate pages can reflect intervening library changes. History ISO dates are UTC version timestamps, alongside the original millisecond values.
