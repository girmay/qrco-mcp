# QRCO MCP — Palette and Color Intelligence MCP

**Palette and color intelligence for AI agents: generate color palettes, build brand systems, check contrast, export CSS/Tailwind tokens and connect saved libraries.**

[QRCO Studio](https://qrco.ca/) · [Connection guide](https://qrco.ca/connect) · [Tool reference](TOOLS.md) · [Example workflows](EXAMPLES.md)

QRCO is a hosted color intelligence service for designers and their AI agents. Generate color schemes, compare directions, build light and dark semantic roles, measure contrast and export CSS or Tailwind tokens. An optional account connection lets your agent work with your saved palette library, preview revisions and inspect version history.

This repository documents the hosted MCP service. It does not contain the application source or a self-hosted server. Public tools and account access are free during the beta; your AI application's own charges may still apply.

## Connect in your AI application

Use an MCP client that supports **remote Streamable HTTP** connections. Client interfaces and OAuth support vary.

| Connection | Server URL | Authentication | Tools |
| --- | --- | --- | --- |
| `qrco` | `https://qrco.ca/mcp` | None | 17 public color tools |
| `qrco-account` | `https://qrco.ca/mcp/account` | OAuth | 4 read tools; up to 4 additional write tools with approval |

1. Add `https://qrco.ca/mcp` as a remote MCP server. No API key is needed.
2. Try: **“Create a five-color palette for a modern luxury coffee brand. Name it, explain each color's role and give me a QRCO Studio link.”**
3. For your saved library, add `https://qrco.ca/mcp/account` as a **separate** OAuth connection. Keep the public connection for generation.
4. Sign in and approve access on QRCO's authorization page. Start with `palettes:read` for a read-only test.
5. Try: **“List my saved palettes, checking all pages, then show the details of the one I choose. Don't change anything.”**

Manage your connections at [QRCO Agent Access](https://qrco.ca/agent-access). Users never need to share passwords, login codes or access tokens in chat. Google sign-in identifies your QRCO account; it does not grant agents Gmail access.

## What you can do

- **Expand a brand color:** create a 50–950 tonal scale with the exact original retained separately, gamut diagnostics, measured contrast and CSS/Tailwind exports.
- **Review color-vision differences:** simulate protanopia, deuteranopia and severe tritanomaly; inspect pair-separation flags with explicit model limits. Add non-color cues; no accessibility certification.
- **Compare and audit:** compare up to five directions with a baseline and brand anchors; audit up to ten palettes against real backgrounds and intended text pairings. Compact summaries keep agent conversations focused.
- **Explore directions:** generate 2–10 colors with fixed anchors, compare up to five palettes, find public Explore palettes and create variations.
- **Build a usable color system:** derive light/dark backgrounds, surfaces, text, primary, accent, border and focus roles with measured contrast pairings.
- **Export design tokens:** CSS, Tailwind CSS v4, SCSS, JSON and SVG palette exports; semantic CSS and Tailwind exports for brand systems.
- **Continue saved work:** read palettes, custom color names and saved brand systems from your own account.
- **Revise with control:** preview stronger contrast or an additional mode, review exact changes, then save a new version with separate permission.
- **Share deliberately:** publish an explicitly approved palette snapshot to a public QRCO page. Private saves and revisions do not update public pages.

See the [complete tool tables](TOOLS.md) and [copyable example prompts](EXAMPLES.md).

## Account permissions

| Scope | Allows |
| --- | --- |
| `palettes:read` | List/read saved palettes, preview brand revisions and read tracked history |
| `palettes:save` | Save palettes privately |
| `palettes:revise` | Save an exact reviewed brand revision as a new version |
| `palettes:publish` | Publish or unpublish a saved palette on explicit user instruction |

A read-only connection intentionally hides write tools. `save_brand_revision` is implemented; it becomes available with `palettes:revise` approval. Existing connections do not automatically gain new permissions.

## Troubleshooting

**“My agent can't see my saved palettes.”** Confirm it has the account connection, not only `/mcp`. Device-only palettes must first be synced to your QRCO account.

**“My agent only found 25 palettes.”** That is one page. Follow `nextOffset` until it is `null` before claiming a library total.

**“I can preview but cannot save a revision.”** Approve `palettes:revise` and refresh the client's tool list. A preview alone never changes a saved palette.

**“The public page didn't change after a revision.”** Public pages are separate snapshots. Publish an update explicitly when ready.

**“OAuth never opened a QRCO page.”** Use your client's real hosted OAuth flow. It must support remote MCP OAuth; do not construct an unrelated callback flow or relay credentials through chat. See [connection guidance](https://qrco.ca/connect).

## Measurements and limits

The legacy `accessible` generation option targets only measured adjacent pairs; inspect `metadata.contrastTarget` for met/unmet status, especially with locks. It does not certify an entire palette.

Contrast measurements cover the specified opaque sRGB pairings, not complete accessibility certification. The separate `check_color_vision` tool supplies bounded simulations and heuristic pair review; it does not simulate tritanopia or certify distinguishability. CMYK values and Pantone/RAL matches are digital approximations, not physical print specifications. Material modes are creative adjustments, not measured fabric or lighting predictions. Trend generation uses a stored model, not live trend forecasting.

Image extraction, batch exporting, cultural research and trademark clearance are not currently offered. Custom color names are supported; evocative naming is not guaranteed. Version history begins with tracked edits and cannot recover states overwritten before tracking began.

## Links and feedback

- [QRCO website](https://qrco.ca/)
- [Connect QRCO MCP](https://qrco.ca/connect)
- [Live plain-text reference](https://qrco.ca/mcp-guide.md)
- [Agent-readable documentation index](https://qrco.ca/llms.txt)

For feedback, include the MCP client, tool name, expected result and a sanitized error. Never post credentials or private palette contents in public issues. Documentation last reviewed September 29, 2026.
