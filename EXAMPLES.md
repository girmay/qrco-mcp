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
