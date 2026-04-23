# Custom Styles for drawio-skill — Design

**Date:** 2026-04-23
**Status:** Design approved, pending implementation plan
**Related issue:** [#2 Custom Styles](https://github.com/Agents365-ai/drawio-skill/issues/2)

## Problem

Diagrams produced by drawio-skill today follow a single hardcoded palette, shape vocabulary, and edge style baked into `SKILL.md`. Users with a distinct visual preference — corporate palette, hand-drawn look, dark theme, etc. — have no way to teach the agent "make diagrams look like this one" short of editing `SKILL.md` themselves and losing those edits on the next auto-update (`git pull`).

Users may arrive with either of two artifact types that encode their preference:
- a `.drawio` XML file they have hand-tuned, or
- an image (PNG/JPG) of a diagram whose look they want to reproduce.

The feature turns either into a named, persistent **style preset** that influences subsequent generations.

## Goals

1. Accept either `.drawio` XML or an image as the input to "learning" a style.
2. Let users keep multiple named presets, with at most one marked as default.
3. Survive skill updates (`git pull`) — user presets must live outside the skill checkout.
4. Fully override the built-in palette/shape/edge conventions when a preset is active; never produce an output that mixes preset colors with built-in defaults.
5. Keep the feature prompt-only — no new runtime code, no new binaries — consistent with the rest of the skill.

## Non-goals (v1)

- Preset versioning / migration UI. Schema is v1 only.
- Style diffing between two presets.
- Automatic/implicit style detection during generation ("you seem to favor blue, want me to save that?"). Learning is always explicit.
- Web UI. All interaction is chat + files.
- Sharing / export format beyond the preset JSON file itself. Users who want to share can commit the file to their own repo.
- Slash commands. All invocation is natural language.

## Decisions (locked during brainstorming)

| Decision | Choice | Rationale |
|---|---|---|
| Scope of a preset | Colors + fonts + shape vocabulary + edge style | Directly visible in both XML and images; avoids fragile "layout/semantic" inference |
| Presets per user | Named, optional default | Degrades to single-default or many-no-default depending on user habits |
| Storage | `~/.drawio-skill/styles/*.json` + shipped `<skill-dir>/styles/built-in/*.json` | Survives skill auto-update; built-ins give immediate value |
| Learn flow UX | Extract → summary + sample PNG → save on approval (uniform for XML and image) | Single codepath; sample render is cheap insurance |
| Invocation | Natural language only | The skill has zero slash commands today; stay consistent |
| Apply behavior | Full override — extrapolate within preset palette | Mixing preset + defaults produces inconsistent output nobody asked for |

## Architecture

Three new artifacts in the repo plus edits to `SKILL.md`. No new runtime code.

```
drawio-skill/
├── SKILL.md                         # edited: +step 0.5, +apply subsection, palette table wrapped
├── styles/
│   ├── built-in/                    # NEW — ships with the repo
│   │   ├── default.json             # mirrors the current SKILL.md color table
│   │   ├── corporate.json           # muted blues/greys, sharp corners
│   │   └── handdrawn.json           # warm palette, sketch=1
│   └── schema.json                  # NEW — JSON Schema for preset files
└── references/
    └── style-extraction.md          # NEW — prompts/rules + sample-render skeleton
                                     #       loaded on demand during the learn flow
```

User-side (outside the repo):

```
~/.drawio-skill/styles/<name>.json   # one file per user preset
```

**Three flows:**

1. **Learn** — `.drawio` or image + target preset name → agent runs extraction → renders a sample PNG using the candidate preset → user approves → agent writes `~/.drawio-skill/styles/<name>.json`.
2. **Generate** — existing diagram workflow gains a preamble that resolves the active preset and applies it throughout.
3. **Manage** — natural-language "list / show / default / delete / rename" handled by a small table in `SKILL.md`.

**Why this shape:**
- `references/style-extraction.md` keeps extraction prompts out of the always-loaded `SKILL.md`.
- Built-in presets travel with the skill version and answer issue #2 even for users who never "teach" a style.
- User presets live outside the checkout so auto-update never touches them.

## Preset JSON schema (v1)

One file per preset. Field names match drawio style keys where possible so the agent can drop values into `style=` strings without translation.

```json
{
  "$schema": "../schema.json",
  "name": "corporate",
  "version": 1,
  "default": false,
  "source": { "type": "xml", "path": "~/work/sample.drawio", "extracted_at": "2026-04-23" },
  "confidence": "high",

  "palette": {
    "primary":   { "fillColor": "#dae8fc", "strokeColor": "#6c8ebf" },
    "success":   { "fillColor": "#d5e8d4", "strokeColor": "#82b366" },
    "warning":   { "fillColor": "#fff2cc", "strokeColor": "#d6b656" },
    "accent":    { "fillColor": "#ffe6cc", "strokeColor": "#d79b00" },
    "danger":    { "fillColor": "#f8cecc", "strokeColor": "#b85450" },
    "neutral":   { "fillColor": "#f5f5f5", "strokeColor": "#666666" },
    "secondary": { "fillColor": "#e1d5e7", "strokeColor": "#9673a6" }
  },

  "roles": {
    "service":  "primary",
    "database": "success",
    "queue":    "warning",
    "gateway":  "accent",
    "error":    "danger",
    "external": "neutral",
    "security": "secondary"
  },

  "shapes": {
    "service":   "rounded=1",
    "database":  "shape=cylinder3",
    "queue":     "rounded=1",
    "decision":  "rhombus",
    "external":  "rounded=1;dashed=1",
    "container": "swimlane;startSize=30"
  },

  "font": {
    "fontFamily": "Helvetica",
    "fontSize": 12,
    "titleFontSize": 14,
    "titleBold": true
  },

  "edges": {
    "style": "edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1",
    "arrow": "endArrow=classic;endFill=1",
    "dashedFor": ["optional", "secondary"]
  },

  "extras": {
    "sketch": false,
    "globalStrokeWidth": 1
  }
}
```

**Fields:**
- **`palette`** — seven fixed-name slots; each slot is a `(fillColor, strokeColor)` pair. Slot names mirror the seven rows of the current `SKILL.md` color table.
- **`roles`** — semantic role → palette slot. Drives color selection during generation.
- **`shapes`** — semantic role → drawio `shape=` / style-keyword prefix.
- **`font`** — single font family + size hierarchy.
- **`edges`** — default edge style string, arrow style, and a `dashedFor` list of label/role tokens that should render dashed.
- **`extras`** — global toggles (`sketch=1` for hand-drawn look; overall `strokeWidth`).
- **`source`** — provenance (file path + type + date).
- **`confidence`** — `"low" | "medium" | "high"`. Set by the extractor; shown to the user at approval time.
- **`version: 1`** — for future schema migration.

`styles/schema.json` is a JSON Schema that validates these fields structurally (required keys, hex format, enum for `confidence`). Not a full type system — only catches hand-edit breakage.

## XML extraction pipeline

Input: `.drawio` file path. Output: candidate preset JSON. Deterministic, no LLM inference.

Steps the agent follows (from `references/style-extraction.md`):

1. **Parse XML.** Read file, collect every `<mxCell>` with a `style=` attribute, split into vertices (`vertex="1"`) and edges (`edge="1"`).
2. **Tokenize each `style=` string** on `;` into key/value pairs or keyword-only tokens (e.g., `rhombus`, `ellipse`, `rounded=1`).
3. **Extract palette.** Collect every `(fillColor, strokeColor)` pair across vertices, count frequency, take top ≤7 pairs. Slot assignment happens in step 4; leftovers fill empty slots in descending-frequency order.
4. **Extract shape vocabulary + role mapping.** For each vertex determine a "shape class" by precedence `cylinder3 > ellipse > rhombus > swimlane > rounded=1 > rounded=0`. Infer semantic role from shape class + label text:
   - cylinder → `database`
   - rhombus → `decision`
   - `dashed=1` + neutral color → `external`
   - swimlane → `container`
   - label matches `/queue|bus|kafka|rabbit/i` → `queue`
   - label matches `/gateway|api|lb|load/i` → `gateway`
   - label matches `/auth|login|jwt|oauth/i` → `security`
   - label matches `/error|fail|alert/i` → `error`
   - everything else → `service`
   
   The most frequent `(role, color-pair)` mapping wins; the pair goes into the role's canonical palette slot (service→primary, database→success, queue→warning, gateway→accent, error→danger, external→neutral, security→secondary) and `roles[role]` is set to that slot name.
5. **Extract fonts.** Modal `fontFamily`, `fontSize`, `fontStyle` across vertices. If a subset consistently uses a larger size + bold, that size becomes `titleFontSize` with `titleBold=true`.
6. **Extract edge defaults.** Modal edge style string, stripped of per-edge coordinate keys (`entryX/entryY/exitX/exitY/entryDx/entryDy/exitDx/exitDy`) which are layout-specific, not style. Record arrow style from `endArrow`/`endFill`. If `dashed=1` edges share a labeling pattern, populate `edges.dashedFor` with matching tokens.
7. **Extract extras.** `sketch=1` present anywhere → `extras.sketch=true`. Modal `strokeWidth` → `globalStrokeWidth`.
8. **Set provenance.** `source.type="xml"`, `source.path=<input>`, `extracted_at=<today>`, `confidence="high"`.

**Edge cases:**
- Source has <3 distinct color pairs → unfilled palette slots left `null`; generation later synthesizes by hue rotation from `primary`. `confidence` drops to `"medium"`.
- Source has >7 color pairs → keep top 7 by frequency; summary warns that some were dropped.
- Source uses non-standard `shape=` keywords (e.g., `shape=mxgraph.aws4.*`) → record the literal string under the closest role and note it in the summary.
- Labels are non-English → the English keyword regexes in step 4 will mostly miss; everything collapses to `service`. The palette, shape vocabulary, font, and edge style are still captured correctly (they don't depend on label text). Confidence stays `"high"`; summary notes that semantic roles beyond service/database/decision/container/external were not inferred.

## Image extraction pipeline

Input: image path (PNG/JPG/any vision-readable format). Output: candidate preset JSON. Inference-based; `confidence: "medium"` at best.

Steps the agent follows:

1. **Read the image** via the agent's built-in vision capability (same mechanism `SKILL.md` step 5 already uses for self-check).
2. **Extract palette by visual inspection.** Identify distinct fill-color regions. For each:
   - `fillColor` — dominant shape-body color, quantized to drawio-typical pastel (round each channel, harmonize lightness ~85%). Map to a named slot by hue: blue → `primary`, green → `success`, yellow → `warning`, orange → `accent`, red/pink → `danger`, grey → `neutral`, purple → `secondary`. Two colors in the same hue family: keep the more frequent one; the other spills to the nearest empty slot.
   - `strokeColor` — matching border. If unreadable, derive from fill by darkening ~25%.
3. **Extract shape vocabulary.** Classify each visible shape by silhouette:
   - rounded rectangle → `rounded=1`
   - sharp rectangle → `rounded=0`
   - circle/oval → `ellipse`
   - diamond → `rhombus`
   - cylinder (rect with curved top/bottom) → `shape=cylinder3`
   - titled container (header bar + nested children) → `swimlane;startSize=30`
   - dashed-bordered rectangle → `rounded=1;dashed=1`
   
   Role assignment uses the **same label-text + shape rules as the XML path** (step 4 above). Visible labels are read via vision.
4. **Extract fonts.** Best-effort. Serif vs. sans vs. monospace: default `"Helvetica"`, override to `"Georgia"` or `"Courier New"` if clearly serif/mono. Size by relative appearance: small → 11, medium → 12, large → 14. Distinctly larger/bolder titles → `titleFontSize` + `titleBold`.
5. **Extract edge defaults.** Orthogonal right-angle arrows → `edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1`. Curved → append `curved=1`. Filled triangle heads → `endArrow=classic;endFill=1`. Open V heads → `endArrow=open;endFill=0`. Dashed arrows near labels like "optional"/"async"/"fallback" → populate `edges.dashedFor`.
6. **Extract extras.** Visibly hand-drawn / rough look → `extras.sketch=true`. Heavy strokes → `globalStrokeWidth=2`.
7. **Set provenance.** `source.type="image"`, `source.path=<input>`, `extracted_at=<today>`, `confidence="medium"` baseline. Downgrade to `"low"` if <3 shapes identifiable; upgrade to `"high"` only on very clean images with ≥5 confidently mapped roles.

**Vision unavailable:** same rule as the skill's existing self-check — stop, tell the user "image-based learning needs a vision-enabled model; re-run on Sonnet/Opus, or give me the `.drawio` file instead."

## Learn flow (end-to-end)

1. User says something like "learn my style from `~/work/sample.drawio` as `corporate`". Agent matches intent from verbs ("learn", "save as style") + a path + a name.
2. Agent dispatches to XML or image path by file extension.
3. Extraction produces a candidate preset (not yet saved).
4. Agent writes the candidate to `/tmp/drawio-preset-<name>.json`.
5. Agent generates a **fixed sample diagram** — seven shapes, one per role (`service`, `database`, `queue`, `gateway`, `external`, `security`, `error`), connected by six edges with at least one dashed so `edges.dashedFor` is exercised. The XML skeleton lives in `references/style-extraction.md` and is parameterized by the candidate preset.
6. Agent exports to PNG with the existing `draw.io -x` command: `./preset-<name>-sample.png` in the user's working directory.
7. Reply to the user contains: (a) preset summary table, (b) sample PNG path + inline view, (c) provenance + confidence line.
8. User responds:
   - "save" / "looks good" → write candidate to `~/.drawio-skill/styles/<name>.json`, delete tempfile and sample PNG.
   - "change X to Y" → apply in-memory edits to the candidate, re-render, re-ask.
   - "cancel" → delete tempfile and sample PNG, no save.

## Apply flow — SKILL.md edits

**Edit 1: New step 0.5 in the Workflow section, right after auto-update:**

> **0.5 Resolve active preset.**
> - Scan the user's message for a named preset reference ("use my `<name>` style", "with `<name>`", "in `<name>` mode"). If found → active preset = `<name>`.
> - Else, check `~/.drawio-skill/styles/` for a file with `"default": true`. If found → active preset = that one.
> - Else → no preset; use the built-in color/shape/edge conventions as today.
> - Load the preset JSON from `~/.drawio-skill/styles/<name>.json`, falling back to `<skill-dir>/styles/built-in/<name>.json`. If the named preset exists in neither, tell the user and list available names; do not silently fall back.

**Edit 2: New subsection "Applying a preset" inserted immediately before the existing "Color palette" table:**

> When a preset is active, it fully replaces the built-in palette, shape keywords, edge defaults, and font for this diagram. Rules:
> - **Colors** — for each role the diagram needs, look up `roles[role]` → slot name → `palette[slot]`. If `roles[role]` is unset, pick the canonical slot by role family (service→primary, data→success, etc.); if that slot is also empty, pick the most-populated slot. Never mix in a color outside the preset palette.
> - **Shapes** — use `shapes[role]` literally as the style-keyword prefix (e.g., `shapes.database = "shape=cylinder3"` → vertex style starts with `shape=cylinder3;whiteSpace=wrap;html=1;...`).
> - **Edges** — use `edges.style` as the base edge string (per-edge routing keys like `entryX/exitX` are still added by the usual routing rules). Append `edges.arrow`. If a logical flow matches a token in `edges.dashedFor`, append `;dashed=1`.
> - **Font** — add `fontFamily=<...>;fontSize=<...>` to every vertex style. Titles and container headers additionally get `fontSize=<titleFontSize>;fontStyle=1` when `titleBold=true`.
> - **Extras** — if `extras.sketch=true`, append `sketch=1` to every vertex and edge style. If `extras.globalStrokeWidth > 1`, append `strokeWidth=<n>` everywhere.
> - **Telling the user** — the first line of the generation response mentions which preset is active: *"Using preset `corporate` (confidence: high)."*

**Edit 3: Wrap the existing "Color palette" table in a conditional:**

> **No preset active:** use the table below (current content unchanged).
> **Preset active:** ignore the table; use the preset palette as described above.

**Interaction with diagram-type presets (ERD / UML / Sequence / …):** user-style preset is **additive on top of** the diagram-type preset. Diagram-type presets set structural style keywords (e.g., ERD tables using `shape=table;...container=1;childLayout=tableLayout;...`) — those are preserved. The user preset then layers color, font, edge style, and extras on top. When a diagram-type preset hardcodes a color that conflicts with the user preset, the user preset wins.

## Management operations (natural language)

A small table in `SKILL.md` listing the canonical phrasings and the file-level effect of each:

| User says | Agent does |
|---|---|
| "list my styles" / "what styles do I have" | Read `~/.drawio-skill/styles/` + `<skill-dir>/styles/built-in/`, print table: name, source type, confidence, default flag |
| "show my `<name>` style" | Print the preset JSON + a one-line summary |
| "make `<name>` the default" | If `<name>` is a user preset: set `default: true` and clear `default` on any other user preset that had it. If `<name>` is a built-in: copy the built-in JSON to `~/.drawio-skill/styles/<name>.json` first, then set `default: true` on the copy (leaves the shipped built-in untouched). |
| "delete `<name>`" | Confirm, then `rm ~/.drawio-skill/styles/<name>.json`. Refuse to delete files under `<skill-dir>/styles/built-in/` — suggest shadowing with a user preset of the same name instead. |
| "rename `<a>` to `<b>`" | `mv` file, update `name` field inside |
| "learn my style from `<path>` as `<name>`" | Dispatch to the learn flow (XML or image path by file extension) |

## Error handling

Each failure produces a clear user-facing message; never a silent bad preset.

| Failure | Behavior |
|---|---|
| Source file path doesn't exist | Stop; report path not found |
| XML parse fails (malformed `.drawio`) | Stop; report the parse error; suggest opening in drawio desktop to repair |
| Image vision unavailable | Stop on image path; tell user to re-run on a vision-capable model or supply XML |
| Extraction produces 0 vertices | Stop; refuse to save — no preset can be learned from nothing |
| Extraction produces <3 distinct color pairs | Continue, but mark `confidence: "low"` and warn in the summary |
| Preset name collides with existing user preset | Ask: overwrite, or pick a new name |
| Preset name collides with a built-in preset | Write user preset to user dir; shadows the built-in on lookup. Warn once. |
| Sample render fails (CLI missing / export error) | Still show summary table; note "could not render sample — saving on your OK anyway". Don't block. |
| Preset fails schema validation at load time | During generation: warn, fall back to built-in for this diagram. During learn: refuse to save. |
| `~/.drawio-skill/styles/` doesn't exist on first use | Create silently on first save |

## Verification checklist

The skill has no unit test harness (it's a prompt). Verification is scenario-based — each scenario must pass before the feature is considered done.

1. **XML learn round-trip.** Feed `assets/demo-layered.drawio`, ask to learn as `demo`. Expected: preset captures the seven color slots from that file, `confidence: "high"`, sample PNG visually matches source colors.
2. **Image learn round-trip.** Feed `assets/demo-layered.png`, ask to learn as `demo-img`. Expected: same ballpark as #1 with `confidence: "medium"`, colors harmonized to drawio-standard pastels.
3. **Apply — explicit.** With `demo` saved, ask for an architecture diagram "using my `demo` style". Expected: output uses preset palette/shapes/edges; first line of reply mentions the active preset.
4. **Apply — default fallback.** Mark `demo` default, ask for a diagram without mentioning style. Expected: still uses `demo`.
5. **No preset.** With no presets saved, ask for a diagram. Expected: current `SKILL.md` behavior, unchanged.
6. **Missing preset.** Ask for a diagram "using my `nonexistent` style". Expected: agent lists available presets, does not silently fall back.
7. **Sparse preset.** Hand-author a preset with only 2 palette slots filled, ask for a diagram that needs 5 roles. Expected: agent stays inside the 2 colors, synthesizes by hue rotation from `primary`, does not reach for built-in defaults.
8. **Built-in shadowing.** Save user preset named `corporate`, verify it wins over `styles/built-in/corporate.json`.
9. **Diagram-type layering.** Ask for an ERD using the `demo` preset. Expected: ERD structural styles preserved (tables as containers with rows); user preset colors/fonts applied on top.

## Open questions (for implementation time)

- Exact contents of the three shipped built-in presets (`default`, `corporate`, `handdrawn`) — palette/font values TBD at implementation; the schema is fixed.
- Exact English phrasings the SKILL.md management table lists as canonical triggers — minor wording, tune during implementation.
