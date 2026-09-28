# QRCO MCP tool reference

[Setup](README.md#connect-in-your-ai-application) · [Examples](EXAMPLES.md) · [Live reference](https://qrco.ca/mcp-guide.md)

## Public endpoint: https://qrco.ca/mcp

No authentication; Streamable HTTP. Twelve tools.

| Tool | Purpose |
| --- | --- |
| `generate_palette` | 2–10 colors, one generation mode and optional fixed colors. |
| `generate_palette_set` | compare 1–5 generated directions in one call. |
| `random_palette` | generate a random palette. |
| `inspect_color` | HEX, RGB, HSL, HSB, Lab, CMYK and nearest digital swatch approximations. |
| `vary_palette` | produce variations from existing colors. |
| `match_swatch` | nearest digital Pantone approximation and color-distance result; verify physically for print. |
| `check_contrast` | WCAG 2.x text contrast for two opaque sRGB colors. |
| `audit_palette` | measured contrast across palette pairs and black/white foreground suggestions. |
| `create_studio_link` | carry colors, palette title and custom color names into Studio; does not save. |
| `export_palette` | return CSS, Tailwind CSS v4, SCSS, JSON or SVG content; does not write files. |
| `search_explore` | browse the public Explore catalog by name or exact HEX. |
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
