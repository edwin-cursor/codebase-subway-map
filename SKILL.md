---
name: subway-map
author: edwinarbus
description: Visualize a codebase as a subway map. Stations = files/modules, tracks = imports/exports, lines = folders/domains.
version: 0.3.2
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
  - GenerateImage
output:
  format: markdown
  constraints:
    - Output exactly one image (not just a text prompt).
    - Do not stop after writing the prompt. Always trigger image generation by calling Nano Banana Pro with a GenerateImage tool request.
    - Do not invent folders/files/modules. Use only the actual repo structure discovered during repo inspection.
---

# Subway Map

Generate an image using Cursor’s built-in **Nano Banana Pro** image generation integration. The map must reflect the **real repo structure** (no invented folders/files). Poster-worthy: crisp, clean, legible, vector-like, print-friendly.

## Encoding
- **Stations** = files or modules
- **Tracks** = imports/exports
- **Lines** = folders/domains (colored + legend)
- **Transfer stations** = high-traffic hubs
- **Terminal stations** = entrypoints

## Workflow

### Step 1: Style Selection (required — must be immediate)
Use **exactly one** AskQuestion call. After the user answers, proceed immediately to repo inspection.

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

### Step 2: Repo Inspection (required — no questions)
After style selection, inspect the repository BEFORE generating the image prompt.

You MUST derive from the real repo:
- Real folder/domain groupings to use as **lines** (e.g., `src/*`, `apps/*`, `packages/*`, `lib/*`, etc.).
- Real entrypoints (candidate terminals): framework entry files and `main/index/app/server`-like entrypoints where applicable.
- Real hubs (candidate transfers): core/shared modules or high fan-in/fan-out nodes.

Hard rules:
- Never fabricate file/folder names. If unknown, omit or generalize (“core/shared modules”) instead of guessing.
- If there are too many domains, group by real parent folder (avoid inventing “misc” unless it literally exists).
- Keep line groups readable (~6–14). Prefer top-level or `src/*` domains.

### Step 3: Scope Selection (only if required — single shot)
Only run this if scope is genuinely unclear after repo inspection (rare).

If needed, use **exactly one** AskQuestion call:
- **Question:** Should stations represent files (detailed) or modules/folders (cleaner for big repos)?

Defaults (use `scope_hint` if provided):
- Large repos / monorepos → **modules**
- Small repos → **files**
- When uncertain → **modules** (readability wins)

Hard rule:
- Do not ask any questions after Step 3.

### Step 4: Generate Image (must execute)
1) Build a final image prompt using:
   - Chosen style pack
   - Chosen scope
   - Real derived folder/domain names for line naming (from repo inspection)
2) Immediately generate the image by calling:

**GenerateImage(prompt: FINAL_PROMPT_TEXT)**

Hard rules:
- Do not output just the prompt.
- Generate exactly one image.
- Do not add extra narration after generating.

## Prompt Structure (FINAL_PROMPT_TEXT)
Make FINAL_PROMPT_TEXT concise (3–10 lines):

1) Opening line:
> Draw a {STYLE NAME} subway-map poster of my codebase dependency graph in a crisp flat vector style, print-ready.

2) Encoding + real lines line:
> Stations are {files|modules}; imports/exports are tracks; lines are colored by these real folders/domains: {DERIVED_LINE_GROUPS}; include a clear legend.

3) Topology line:
> Big hubs are transfer stations; entrypoints are terminal stations; minimize crossings and keep spacing tidy.

4) Readability line:
> Truncate paths; keep labels legible and non-overlapping; prioritize poster clarity over perfect fidelity.

5) Style keywords line:
> {STYLE KEYWORDS}

6) Constraints + background + title line:
> Retro flat only — no gradients, no shadows, no glow, no texture, no 3D, no paper grain. Background pure white. Add a creative subway-map title based on {repo_name if provided} (no fixed title).

## Style Packs

### A) Retro NYC (1970s Manhattan) (DEFAULT)
**STYLE NAME:** Retro NYC / Vignelli Manhattan (1970s)

**STYLE KEYWORDS:**
- Modernist museum poster layout: BIG bold header, wide white margins, graphic and sparse.
- EXTRA-WIDE banded ribbon routes built from multiple parallel stripes; mostly vertical trunks + horizontal crossbars.
- Stations are small solid black dots in rhythmic sequences; transfers are larger bold circular nodes.
- Aggressively schematic: crisp 90° turns with occasional 45°; almost no curves; clean layering at crossings.
- Modernist sans-serif typography; labels compact and secondary; strong hierarchy.
- Flat, saturated transit colors (orange/yellow/green/blue/purple) with uniform fills; black used sparingly for dots; lots of white space.

### B) Madrid (thin + minimal)
**STYLE NAME:** Madrid Metro (minimal)

**STYLE KEYWORDS:**
- Calm, airy technical diagram with lots of whitespace; subtle hierarchy.
- THIN, crisp line strokes; simple angles; minimal visual bulk.
- Stations are small WHITE circles with COLORED outlines; transfers slightly emphasized but still minimal.
- Restrained typography; tidy label placement; minimal ornamentation.
- Clean flat colors with restrained saturation; white background dominates; color differentiates lines without shouting.

### C) Washington DC (thick, bright colors, bold)
**STYLE NAME:** Washington DC Metro (bold)

**STYLE KEYWORDS:**
- Very bold, high-contrast: thick routes dominate; map reads instantly.
- VERY THICK saturated colored bands with DARK/BLACK OUTLINES (cased lines).
- Transfer stations are LARGE CONCENTRIC circles (target-like); regular stations prominent and high-contrast.
- Bright system palette (vivid red/deep blue/strong orange/bold yellow/bright green) with crisp black casing.
- Optional subtle pale flat accents for water/parks, but keep the schematic dominant and clean.

### D) London Underground (classic Tube)
**STYLE NAME:** London Underground (classic Tube)

**STYLE KEYWORDS:**
- Dense network but extremely ordered; official transit artifact feel; consistent symbol system.
- MEDIUM line weight (thicker than Madrid, thinner than DC); mostly straight segments with a few controlled rounded corners/curves.
- Stations are white circles with black outlines; interchanges larger or multi-ring nodes; walking links can be outlined connectors (sometimes dotted/segmented).
- Classic Tube palette: distinct flat line colors (strong red, deep blue, rich green, warm yellow, purple, brown) with balanced “official” saturation.
- Optional ultra-subtle pale blue river / muted light-green parks, kept faint and flat.

### E) Surprise me
Choose randomly from A–D and proceed immediately.

### F) Other
Use the user’s style text as the style direction; keep all encoding/readability/background/title rules.

## Readability Rules (always apply)
- Use ONLY real folders/domains and real file/module names from the repo; never invent.
- Truncate paths (`src/ui/components/Button.tsx` → `ui/Button` or `Button`)
- Keep labels legible; avoid overlaps and text-on-line collisions
- Prefer line continuity and clear spacing; minimize crossings
- Prefer “poster truth” over perfect fidelity
- Background must be **pure white** (no beige/off-white)
- Include a map legend in the corner, with lines named after the real folders/domains
