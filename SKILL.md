---
name: subway-map
author: edwinarbus
description: Visualize a codebase as a subway map. Stations = files/modules, tracks = imports/exports, lines = folders/domains. Must be exactly representative of the real project structure across the entire repo.
version: 0.3.7
inputs:
  repo_name:
    type: string
    required: false
    description: Optional repo/project name to use in the map title text.
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
    - Output exactly one image file (a single flat image suitable for printing). Do NOT generate or depict a photographed/physical poster mockup.
    - Do not stop after writing the prompt. Always trigger image generation by calling Nano Banana Pro with a GenerateImage tool request.
    - The map must be EXACTLY representative of the repo across the WHOLE PROJECT: do not invent or add any stations, lines, folders, modules, packages, or connections that are not present in the real project.
    - Every line name must correspond to a real folder/domain that exists in the repo (top-level folders OR nested domains). Every station label must correspond to a real file or real module/folder chosen in scope.
    - On repeat runs within the same chat, reuse the previously derived repo structure (folders/domains, scope decision, entrypoints, hubs) from conversation context instead of re-deriving it—unless the user explicitly indicates the repo changed.
---

# Subway Map

Generate **one flat image file** using Cursor’s built-in **Nano Banana Pro** image generation integration. The output should be a clean vector-like subway map graphic (print-friendly), **but not a physical poster mockup** (no paper texture, no framing, no wall, no shadows).

The map must reflect the **real repo structure across the entire project** (no invented structure). Crisp, clean, legible, print-friendly.

## Encoding
- **Stations** = files or modules (real project only)
- **Tracks** = imports/exports (real edges only)
- **Lines** = folders/domains across the whole repo (real folders only; colored + legend)
- **Transfer stations** = high-traffic hubs (real hubs only)
- **Terminal stations** = entrypoints (real entrypoints only)

## Repeat-run behavior (must use chat context)
If the user runs this skill again in the same chat/session:
- Always run **Step 1 (Style Selection)** again (user can pick a different style).
- Reuse the **previously derived repo structure** from earlier messages in the conversation:
  - derived line groups (folders/domains)
  - chosen station scope (files vs modules), unless user asks to change it
  - derived entrypoints and hubs
  - any exclusions applied
- Do NOT hallucinate or “refresh” structure.
- Only redo repo inspection if the user explicitly says the repo changed (new folders, refactor, different branch, etc.).

## Non-negotiable accuracy rules
- **Do not invent anything.** No extra stations, no extra lines, no fake branches, no decorative stops.
- **Whole repo coverage is mandatory:** include the full project structure, not only `src/`.
- If simplification is needed, you may only **merge/remove** nodes; never add new ones. Merged nodes must correspond to real folders/modules.
- If the repo is small, the map should be small (do not expand it for aesthetics).

## Output rules (image-only, not a poster mockup)
- Produce a **single flat image** intended for printing (like a clean diagram export).
- Do **not** render a physical poster, mockup, frame, wall, desk, paper, folds, lighting, or shadows.
- Retro flat design only: no gradients, no glow, no texture, no paper grain, no 3D.

## Workflow

### Step 1: Style Selection (required — must be immediate; always ask on each run)
Use **exactly one** AskQuestion call. After the user answers, proceed immediately.

AskQuestion:
- **Question:** Which subway-map style should I emulate?
- **Options:**
  - A) Retro NYC (Vignelli-esque Manhattan poster)
  - B) Madrid (thin + minimal)
  - C) Washington DC (thick, bright colors, bold)
  - D) London Underground (classic Tube)
  - E) Surprise me

Hard rules:
- Do **not** acknowledge the choice with extra commentary.
- If user picks **E** or provides no selection, choose a random option from **A–D** and proceed immediately.
- If user picks **F**, use their typed style direction and proceed immediately.

### Step 2: Full Repo Inspection (required on first run; optional on repeat runs)
- **First run in this chat:** inspect the ENTIRE repository BEFORE generating the prompt.
- **Repeat run in this chat:** reuse derived structure from conversation context (do not re-derive) unless user explicitly says the repo changed.

When inspecting (first run), derive from the real repo (whole project, not just `src/`):
- **Line set:** real folder/domain groupings across the repo (top-level folders + major subdomains where needed).
- **Station set:** real files OR real modules/folders depending on scope.
- **Edge set:** real import/export relationships between stations (or aggregated relationships for module scope).
- **Entrypoints:** real entry files across apps/packages/CLIs/scripts as terminals.
- **Hubs:** real high fan-in/fan-out nodes as transfers.

Allowed exclusions (noise only):
- `.git/`, `node_modules/`, build output (`dist/`, `build/`, `out/`), caches, vendored deps.

Hard rules:
- Never fabricate names. If you cannot confirm it exists, do not include it.
- Do not add extra lines to “fill out” the map.

### Step 3: Scope Selection (only if required — single shot)
Only run this if scope is genuinely unclear after inspection (or if user requests a different scope).

If needed, use **exactly one** AskQuestion call:
- **Question:** Should stations represent files (detailed) or modules/folders (cleaner for big repos)?

Defaults (use `scope_hint` if provided):
- Large repos / monorepos → **modules**
- Small repos → **files**
- When uncertain → **modules** (readability wins)

Hard rule:
- Do not ask any questions after Step 3.

### Step 4: Generate Image (must execute)
1) Build FINAL_PROMPT_TEXT using:
   - Chosen style pack
   - Chosen scope
   - Derived real line set + real station set + real edge set (whole repo; reused on repeat runs)
2) Immediately generate the image by calling:

**GenerateImage(prompt: FINAL_PROMPT_TEXT)**

Hard rules:
- Generate exactly one image file.
- Do not output only the prompt.
- Do not add extra narration after generating.

## Prompt Structure (FINAL_PROMPT_TEXT)
Make FINAL_PROMPT_TEXT concise (3–10 lines) and include explicit “flat image” + “whole repo” + “no hallucination” constraints:

1) Opening line:
> Draw a {STYLE NAME} subway-map diagram of my WHOLE REPO dependency graph as a single flat image (no poster mockup), crisp vector-like, print-ready.

2) Grounding constraint line:
> IMPORTANT: Use ONLY real folders/files/modules and real imports/exports from the repo; do not add or rename anything; do not fabricate extra lines to fill space.

3) Lines definition line:
> Lines are colored by these real repo folders/domains (full coverage): {DERIVED_LINE_GROUPS}; include a legend listing ONLY these.

4) Stations + edges line:
> Stations are {files|modules} from the repo; tracks represent ONLY real imports/exports; terminals are real entrypoints; transfers are real hubs.

5) Readability line:
> Truncate paths; keep labels non-overlapping; you may simplify only by merging/removing minor leaf nodes, never by adding nodes/lines/edges.

6) Style keywords line:
> {STYLE KEYWORDS}

7) Constraints + background + title line:
> Retro flat only — no gradients, no shadows, no glow, no texture, no 3D, no paper grain. Background pure white. Add a creative subway-map title based on {repo_name if provided} (no fixed title).

## Style Packs

### A) Retro NYC (1970s Manhattan) (DEFAULT)
**STYLE NAME:** Retro NYC / Vignelli Manhattan (1970s)

**STYLE KEYWORDS:**
Inspired by the **1970s NYC Subway Map / Manhattan diagram (Vignelli)**.
- Modernist museum poster layout: BIG bold header, wide white margins, graphic and sparse.
- EXTRA-WIDE banded ribbon routes built from multiple parallel stripes; mostly vertical trunks + horizontal crossbars.
- Stations are small solid black dots in rhythmic sequences; transfers are larger bold circular nodes.
- Aggressively schematic: crisp 90° turns with occasional 45°; almost no curves; clean layering at crossings.
- Modernist sans-serif typography; labels compact and secondary; strong hierarchy.
- Flat, saturated transit colors (orange/yellow/green/blue/purple) with uniform fills; black used sparingly for dots; lots of white space.

### B) Madrid (thin + minimal)
**STYLE NAME:** Madrid Metro (minimal)

**STYLE KEYWORDS:**
Inspired by the **Madrid Metro map**.
- Calm, airy technical diagram with lots of whitespace; subtle hierarchy.
- THIN, crisp line strokes; simple angles; minimal visual bulk.
- Stations are small WHITE circles with COLORED outlines; transfers slightly emphasized but still minimal.
- Restrained typography; tidy label placement; minimal ornamentation.
- Clean flat colors with restrained saturation; white background dominates; color differentiates lines without shouting.

### C) Washington DC (thick, bright colors, bold)
**STYLE NAME:** Washington DC Metro (bold)

**STYLE KEYWORDS:**
Inspired by the **Washington DC Metro (WMATA) map**.
- Very bold, high-contrast: thick routes dominate; map reads instantly.
- VERY THICK saturated colored bands with DARK/BLACK OUTLINES (cased lines).
- Transfer stations are LARGE CONCENTRIC circles (target-like); regular stations prominent and high-contrast.
- Bright system palette (vivid red/deep blue/strong orange/bold yellow/bright green) with crisp black casing.
- Optional subtle pale flat accents for water/parks, but keep the schematic dominant and clean.

### D) London Underground (classic Tube)
**STYLE NAME:** London Underground (classic Tube)

**STYLE KEYWORDS:**
Inspired by the **London Underground (Tube) map**.
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
- Cover the whole repo: represent real top-level folders and major domains across the project (not only `src/`).
- Use ONLY real folders/domains and real file/module names from the repo; never invent.
- Do not add decorative branches, stations, labels, or lines.
- Truncate paths (`src/ui/components/Button.tsx` → `ui/Button` or `Button`)
- Keep labels legible; avoid overlaps and text-on-line collisions
- Prefer line continuity and clear spacing; minimize crossings
- Prefer “poster truth” over perfect fidelity by REMOVING/AGGREGATING only (never adding)
- Background must be **pure white** (no beige/off-white)
- Include a legend listing ONLY the real folders/domains used as lines
