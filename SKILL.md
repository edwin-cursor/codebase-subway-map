---
name: subway-map
author: edwinarbus
description: Visualize a codebase as a subway map. Stations = files/modules, tracks = imports/exports, lines = folders/domains.
version: 0.1.0
inputs:
  repo_name:
    type: string
    required: false
    description: Optional repo/project name to use in the poster title.
  scope_hint:
    type: string
    required: false
    description: Optional hint like "small repo" / "large monorepo" / "focus on src/app".
tools:
  - AskQuestion
output:
  format: markdown
  constraints:
    - Output exactly one image prompt (3–10 lines).
---

# Subway Map

Generate **one** image-generation prompt that turns a codebase dependency graph into a classic transit-style schematic that is **poster-worthy**: crisp, clean, legible, vector-like, print-friendly.

## Encoding
- **Stations** = files or modules
- **Tracks** = imports/exports
- **Lines** = folders/domains (colored + legend)
- **Transfer stations** = high-traffic hubs
- **Terminal stations** = entrypoints

## Workflow

### Step 1: Style Selection (required — must be immediate)
Use **exactly one** AskQuestion call, then begin prompt generation immediately after the user answers.

AskQuestion:
- **Question:** Which subway-map style should I emulate?
- **Options:**
  - A) Retro NYC (Vignelli-esque Manhattan poster)
  - B) Madrid (thin + minimal)
  - C) Washington DC (thick, bright colors, bold)
  - D) London Underground (classic Tube)
  - E) Surprise me
  - F) Other (type your own style)

Hard rules:
- Do **not** acknowledge the choice with extra commentary.
- If user picks **E** or provides no selection, choose a random option from **A–D** and proceed immediately.
- If user picks **F**, use their typed style direction and proceed immediately.

### Step 2: Scope Selection (only if required — single shot)
Only run Step 2 if scope is genuinely unclear.

If needed, use **exactly one** AskQuestion call:
- **Question:** Should stations represent files (detailed) or modules/folders (cleaner for big repos)?

Hard rules:
- Do **not** ask any additional questions after the user answers Step 2.
- After receiving the Step 2 answer, immediately output the final prompt (Step 3) in the same turn.

Defaults (use `scope_hint` if provided):
- Large repos / monorepos → **modules**
- Small repos → **files**
- When uncertain → **modules** (readability wins)

### Step 3: Generate Prompt (single output)
Output **one** image prompt (3–10 lines) using this structure:

1) Opening line:
> Draw a {STYLE NAME} subway-map poster of my codebase dependency graph in a crisp flat vector style, print-ready.

2) Encoding + legend line:
> Stations are {files|modules}; imports are tracks; top-level folders/domains are colored lines with a clear legend.

3) Topology line:
> Big hubs are transfer stations; entrypoints are terminal stations; minimize crossings and keep spacing tidy.

4) Readability line:
> Truncate paths (e.g., `src/ui/components/Button.tsx` → `ui/Button` or `Button`); keep labels legible and non-overlapping.

5) Style keywords line:
> {STYLE KEYWORDS}

6) Constraints + background + title line:
> Flat design only — no gradients, no shadows, no glow, no texture, no 3D, no paper grain. Background **pure white**. Add a creative subway-map title based on {repo_name if provided} (no fixed title), e.g., “{repo_name} City Metro Map” / “{repo_name} Underground” / “Dependency City Transit Map”.

## Style Packs

### A) Retro NYC (1970s Manhattan) (DEFAULT)
**Keywords:**

Retro NYC / Massimo Vignelli-style “Manhattan” diagram (match the reference image closely):

Overall composition & hierarchy
- Feels like a modernist museum poster: BIG bold header centered at the top, lots of clean white margin, very graphic and sparse.
- Map is intentionally abstract: not geography, but a structured systems diagram with strong design hierarchy (title > lines > stations > labels).

Linework (the signature look)
- Routes are EXTRA-WIDE “ribbons” built from multiple parallel stripes (banded lines). Each route looks like a stack of colored lanes, not a single stroke.
- The ribbons run mostly as clean vertical columns and horizontal crossbars, with hard 90° bends and sharp joins—almost no curves.
- Overlaps are handled by clear layering: ribbons cross cleanly without messy tangles; keep intersections crisp and readable.

Stations & stop patterns
- Stations are small solid black dots placed in a steady rhythm along the ribbons—often appearing as dotted sequences down a corridor.
- Express/local feel is implied by different dot groupings or spacing patterns; keep the “dotted stop” language consistent and minimal.
- Transfer/interchange points should read as “special” but still simple: slightly larger nodes, clustered dot groupings, or emphasized junction points—avoid complex interchange symbols.

Typography (very specific vibe)
- Type is clean, sans-serif, modernist: high legibility, minimal decoration, strong weight contrast.
- Station labels are compact and secondary to the line structure; many labels are small and placed with generous breathing room.
- Limited use of bold: major anchors/areas get stronger weight; most station names are understated.

Color (very specific palette behavior)
- Flat, saturated, “primary-ish” transit colors with sharp edges: strong oranges, yellows, greens, blues, and occasional purple.
- Each route ribbon reads like a bold color block; the multi-stripe/banded construction can introduce adjacent stripes in closely related hues (e.g., orange + lighter orange, blue + lighter blue).
- Black is used sparingly but strongly for stations (dots) and small stop patterns; white space is dominant.
- No shading, no gradients—colors are uniform fills.

Finish & background
- No gradients, no shadows, no glow, no texture, no paper grain.
- Background is PURE white, bright and clinical; poster should look like a clean vector print.

Geometry & spacing rules
- Strong rectilinear grid logic: align cleanly, maintain consistent spacing between ribbons, keep margins and gutters wide.
- Everything feels “designed”: consistent dot size, consistent stripe widths within a ribbon, consistent label offsets.
- Overall effect: simplified corridor diagram with dominant vertical trunks + occasional horizontal connectors; minimal clutter.

Title/legend treatment
- Bold top title block in the same modernist style; title is creative based on the codebase.
- Legend is compact and clean: line color ↔ folder/domain names, set like a real transit agency key.

### B) Madrid (thin + minimal)
**Keywords:**

Madrid Metro-style minimal schematic (match the reference image closely):

Overall composition & hierarchy
- Feels like a clean technical diagram: calm, airy, lots of whitespace, information-dense but not heavy.
- Visual hierarchy is subtle: lines and stations first, labels second; everything looks tidy and systematic.

Linework (the signature look)
- Routes are THIN, crisp strokes with clean joins; much lighter than London/DC.
- Geometry favors straight segments and simple angles; curves are rare and gentle.
- Lines are separated with generous spacing; avoid thick overlaps and visual bulk.

Stations & interchange language
- Stations are small WHITE circles with COLORED outlines; consistent size and spacing.
- Interchanges are only slightly emphasized (slightly larger circles or tighter clusters), still minimal.
- The map avoids heavy black casings or big “target” markers—everything stays delicate.

Typography
- Labels are small, neat, and restrained; consistent weight; careful placement to avoid collisions.
- Text never dominates the diagram; it supports navigation without clutter.

Color (very specific palette behavior)
- Colors are clean and flat but not overly saturated—more “diagram” than “poster neon.”
- Each line is a single, clear hue (reds, blues, greens, yellows, purples), but with restrained intensity.
- Station circles are white with colored rings; outlines are crisp and consistent.
- Background white carries most of the visual weight; color is used as an accent to differentiate lines.

Finish & background
- No gradients, shadows, glow, texture, paper grain, or 3D.
- Background is bright PURE white; overall look is light and minimal.

Geometry & spacing rules
- Consistent padding between lines, stations, and labels; lots of breathing room.
- Keep symbols small and uniform; avoid oversized hubs unless absolutely necessary.
- Overall effect: airy schematic that can show many stops without feeling crowded.

Title/legend treatment
- Title is modest and diagram-like but still creative based on the codebase.
- Legend is compact and functional, placed cleanly with minimal decoration.

### C) Washington DC (thick, bright colors, bold)
**Keywords:**

Washington DC Metro-style bold schematic (match the reference image closely):

Overall composition & hierarchy
- Feels bold and high-contrast: thick color routes dominate; stations and transfers read instantly.
- Includes light “map context” shapes (water/parks) but the transit diagram remains primary and very clear.

Linework (the signature look)
- Routes are VERY THICK, saturated colored bands with DARK/BLACK OUTLINES (cased lines).
- Lines have strong color blocking and stand out aggressively against the white background.
- Intersections are simplified and clear; avoid tangled crossings.

Stations & interchange language
- Stations are prominent and high-contrast; regular stops are clearly marked.
- Transfer stations are LARGE CONCENTRIC CIRCLES (target-like) at multi-line junctions.
- Junctions feel engineered: clean merges, clean splits, strong node emphasis.

Typography
- Labels are larger and more practical than Madrid; optimized for wayfinding.
- Clear placement with enough spacing; prioritize legibility.

Color (very specific palette behavior)
- Bright, saturated “system colors” with very high contrast: vivid red, deep blue, strong orange, bold yellow, bright green.
- Each thick colored band is edged with a crisp black outline that increases contrast and readability.
- Transfer circles and station markers use black/white contrast heavily (targets/rings pop).
- Background accents (if any) are very pale and flat: light blue for water, soft green for parks—kept subtle so the bold lines dominate.

Finish & background
- Flat vector only: no gradients, shadows, glow, texture, paper grain, or 3D.
- Background is PURE white (avoid beige); keep it poster-clean.

Geometry & spacing rules
- Strong, bold geometry: mostly straight segments and clean turns.
- Keep line widths consistently thick; keep station markers and transfer targets consistent.
- Overall effect: friendly, bold, poster-clean system map with extremely strong line identity.

Title/legend treatment
- Title can be moderate/agency-like but still creative based on the codebase.
- Legend is clear and prominent; line colors are a key feature and should be easy to scan.

### D) London Underground (classic Tube)
**Keywords:**

London Underground classic Tube schematic (match the reference image closely):

Overall composition & hierarchy
- Dense network but disciplined: lots of stations and lines, yet orderly and readable.
- Feels like an official transit artifact: consistent symbol system, consistent spacing, consistent typography.

Linework (the signature look)
- Lines are MEDIUM thickness (thicker than Madrid, thinner than DC) with consistent stroke.
- Mix of straight segments and controlled rounded corners/curves when it improves readability.
- The network can feel packed, but spacing rules keep it from becoming chaotic.

Stations, interchanges, and connectors
- Stations are white circles with black outlines; frequent stops are common.
- Interchanges are clearly differentiated nodes where multiple lines meet cleanly (often larger rings or emphasized junction circles).
- Walking/transfer connectors can appear as black/white outlined links; sometimes dotted/segmented; keep them neat and angular.

Typography
- Strong hierarchy: station names are bold black and very legible; secondary place labels smaller.
- Label placement is disciplined with consistent offsets; avoid overlaps.

Color (very specific palette behavior)
- Flat, distinct line colors with a classic Tube feel: strong red, deep blue, rich green, warm yellow, purple, brown, black/gray auxiliary lines.
- Colors are saturated but slightly more “official” and balanced than DC (less neon, more standardized).
- Black outlines are used primarily for station rings and connector links, not for casing every line.
- Optional background context is extremely subtle: pale blue river shapes, muted light-green park blocks—kept faint and flat.

Finish & background
- Flat vector only: no gradients, shadows, glow, texture, paper grain, or 3D.
- Background is pure white or very light gray; overall finish is polished, crisp, and print-ready.

Geometry & spacing rules
- “Engineered” interchange geometry: clean multi-line joins, consistent ring sizes, consistent stroke weights.
- Keep crossings minimal; prefer rerouting with gentle curves or spacing adjustments to preserve clarity.
- Overall effect: classic Tube look—dense but extremely controlled.

Title/legend treatment
- Title feels official and system-like but creative based on the codebase.
- Legend is tidy and compact with consistent line samples and text alignment.

### E) Surprise me
Pick one of A–D randomly; use its keywords.

### F) Other
Use the user’s style text as the style direction; keep all encoding/readability/background/title rules.

## Readability Rules (always apply)
- Truncate paths (`src/ui/components/Button.tsx` → `ui/Button` or `Button`)
- Keep labels legible; avoid overlaps and text-on-line collisions
- Prefer line continuity and clear spacing; minimize crossings
- Prefer “poster truth” over perfect fidelity
- Background must be **pure white** (no beige/off-white)
- Include a map legend in the corner, with lines named after the folders/domains
