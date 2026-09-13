---
name: inspo
description: Full inspiration-to-prototype workflow for design and frontend tasks. Searches 4,235 structurally tagged real-world screenshots in the local library, selects references that are provably divergent across 9 structural axes, generates 3–5 high-fidelity clickable HTML prototypes with wired navigation, presents them in an interactive canvas switcher, refines the chosen one in-place, then writes a DESIGN.md handoff spec. Runs automatically on any design or frontend brief. Trigger words: "inspo", "check the library", "find references", "grab inspiration", "build N pages", or any design brief.
---

# inspo — intake → references → divergence → prototypes → canvas → refine → handoff

> **Source of the method:** `PROTOCOL.md` at the library root is vendor-neutral and
> authoritative. This skill is the Claude-side wrapper. If the two ever disagree,
> PROTOCOL.md wins — and fix the drift rather than editing around it. An earlier copy of
> this protocol lived only in the installed `~/.claude/` copy and was destroyed by a
> `cadre sync`; that is why the method now lives with the corpus, not with the client.
>
> Prefer the MCP server when it is registered (`select_divergent_references`,
> `get_reference_image`, ...) — same logic, fewer shell calls:
> `claude mcp add inspo -- python3 "$LIB/_library/mcp_server.py"`

```
LIB="/Users/dhaban/Documents/Design Work/WEB_DESIGN_INSPIRATION_pinterest_2026-08-26 2"
PROTO="./_prototypes"
```

The library is **read-only**. Reference by absolute path; never copy files into the project;
never write tool output into `$LIB` (always pass an absolute `--output` under `$PROTO`).

## Search tools

```bash
# Structural divergence — THE tool for picking concepts that genuinely differ.
# Plain python3, no venv. Excludes the untrusted tag tier by default.
python3 "$LIB/_library/structure_search.py" --k 5 \
  --categories Portfolio Agency-Studio \
  --output "$(pwd)/_prototypes/divergent_selection.json"

# Keyword — fast, misses ~⅓ of the library (no title/description on those)
python3 "$LIB/_library/search.py" --category Portfolio --query "editorial minimal" --limit 10

# Visual/semantic — reaches what keyword can't; best for look-and-feel
"$LIB/_library/.venv/bin/python" "$LIB/_library/semantic_search.py" \
  --like "dark editorial portfolio black type" --limit 10

# Taxonomy — industry/page-purpose/pattern. CLIP-derived tags are candidates to
# view, not facts (raw confidence ~0.17–0.30). Add --require-text-match for the
# trustworthy tier only.
python3 "$LIB/_library/taxonomy_search.py" --industry NGO --pattern Donation --limit 10
```

### The 9 structural axes

Every one of the 4,235 images carries one value per axis plus a `signature_move`
(`_library/structure.py` holds the closed vocabularies):

`layout_system` · `spatial_character` · `type_treatment` · `behaviour` ·
`content_hierarchy` · `navigation_model` · `imagery_role` · `colour_character` ·
`data_presentation`

`structure_search.py` measures divergence as plain categorical Hamming distance —
how many of the nine axes two references disagree on — and walks a deterministic
greedy farthest-point selection. It prints `min pairwise N/9`: **the guarantee that
your concepts are actually different.** Treat below 5/9 as too similar; re-run with a
wider `--categories` or a larger pool.

**Tag trust:** `--include-unverified` exists. Do not use it. That tier scored 51–56%
in two blind audits and contains records bound to the wrong image entirely.

---

## STEP 1 — INTAKE

Before searching anything, understand the project.

**First:** scan the project directory for existing context files — `README.md`, `brief.md`,
`ANCHOR.md`, `package.json` (description field), any `content/` or `copy/` folders.
Extract what you can. Only ask for what's missing.

**If this is a redesign:** audit the existing site before asking anything:
- Read the existing CSS/HTML — extract current colors, fonts, spacing, structure
- Screenshot or view the live/local site if possible
- Run semantic search on any existing screenshot found:
```bash
  "$LIB/_library/.venv/bin/python" "$LIB/_library/semantic_search.py" \
    --similar-to "./screenshot-current.png" --limit 10
```
- Then ask redesign-specific intake instead of new-site intake:
  1. What's broken? (specific complaints — not "it feels outdated")
  2. What's working and must be preserved?
  3. What triggered this redesign?
  4. Same audience and tone, or shifting?
  5. Hard constraints — must keep certain sections, colors, content structure?

Add a "Current" card to the canvas switcher as the zero-th prototype — the existing
design as a baseline to compare against. DESIGN.md gains a `## What Changed` section:
per decision, what existed, what replaced it, why.

**If this is new — ask in a single message, all at once:**

```
1. What is this site? (name + one sentence — what it does, for whom)
2. Who is the audience? (be specific — not "businesses", but "Somali shop owners in Jigjiga")
3. Tone in 3 adjectives (not "clean and modern" — those aren't tones)
4. What sections must exist? (list them)
5. Any sites you've seen that feel right, even partially?
6. Anything this must NOT look like?
```

Save to `$PROTO/brief.json`:
```json
{
  "project": "",
  "description": "",
  "audience": "",
  "tone": [],
  "sections": [],
  "references_admired": [],
  "avoid": []
}
```

This file is the shared product problem all concepts must solve. Every direction in
STEP 3 answers the *same* brief — that is what makes them comparable rather than
merely different.

**Generate realistic copy from the intake immediately** — actual headlines, body text,
CTA labels, nav items, specific to this project. No lorem ipsum, no "Your headline here."
This copy is used in all prototypes. Bad copy kills a good design.

---

## STEP 2 — CANDIDATE SEARCH

Map the intake to library facets.

**Categories (12):**
`SaaS-Tech-Product` · `Landing-Pages` · `Ecommerce-Retail` · `Agency-Studio` ·
`Editorial-Blog` · `Food-Hospitality` · `Portfolio` · `Mobile-App` ·
`Graphic-Abstract` · `Photography` · `Branding-Identity` · `Typography`

**Source signal:** `Land-book` / `Lapa Ninja` / `Awwwards` → full-page composition.
`Dribbble` / `Behance` → isolated component ideas.

Cast every tool that applies. Target 8–15 total candidates — not per tool.

**View every candidate image file before shortlisting.** For each, output one line:
```
[absolute path] — [one factual visual observation: nav type / hero treatment / color / type / grid]
```
Never decide from titles or tags alone. A tag is a candidate to view, never a fact to
build from. If the library has no real match, say so — still proceed, but mark which
directions are invented from principles rather than drawn from references.

---

## STEP 3 — DIVERGENCE GATE

Round one is never one interface. It is **3–5 structurally distinct directions that
solve the same brief differently.**

**Run the divergence selection first** — it decides the structural spread mechanically
rather than by eye:

```bash
python3 "$LIB/_library/structure_search.py" --k 5 \
  --categories <the categories the brief maps to> \
  --output "$(pwd)/_prototypes/divergent_selection.json"
```

Read the printed report. `min pairwise` is the divergence guarantee; the per-axis
`distinct value(s)` lines tell you which axes are actually carrying the difference
and which have collapsed to one value across all five.

If a category's trusted pool is too thin, widen `--categories` to a neighbouring
category that shares the same underlying problem (a studio homepage and a portfolio
are both identity/work/contact) — and **say** you substituted, so the human can judge it.

Then **view the selected references** and let them inform, not dictate. The selection
guarantees structural spread; your judgement decides whether a direction is worth
building. Drop one and fall to 4 or 3 rather than padding with a variation.

**Each direction gets:**

1. **A name** — after its structural concept, not generic numbering.
   Good: `Left-Anchor` · `Editorial-Grid` · `Full-Bleed` · `Table-Row` · `Modular-Card`
   Bad: `Concept 1` · `Direction A` · `Option 2`

2. **A structural fingerprint** — one line, structural only (not colors, not tone):
   > "Fixed left sidebar, scrolling right content, work as ruled table rows, contact as bare form"

3. **Its axis vector** — the 9 values from `divergent_selection.json`, quoted verbatim.
   This is what makes "these are different" checkable instead of asserted.

4. **A reference cluster** — which library images informed it (2–4 absolute paths)

5. **A palette.py run** on that cluster:
   ```bash
   "$LIB/_library/.venv/bin/python" "$LIB/_library/palette.py" \
     "/abs/path/ref1.png" "/abs/path/ref2.png" --n 6
   ```

**Fingerprint uniqueness check:** no two directions may share the same `navigation_model`
**and** `content_hierarchy` **and** `layout_system`. If two do, the divergence run was too
narrow — widen the pool or drop the weaker direction.

**AI-default pre-flight (run for every direction).** If a direction matches one of these,
modify the direction — not just note it:
- Warm cream (#F4F1EA range) + serif display + terracotta/clay accent
- Near-black background + single acid-green or vermilion accent
- Inter or Space Grotesk as the font choice (unless the reference genuinely uses it)
- Rounded-lg applied uniformly
- Centered hero with headline + subheadline + two CTA buttons
- Gradient wash on white hero
- ALL-CAPS tracked eyebrow above every heading
- 01 / 02 / 03 numbering on non-sequential content
- Emoji section markers
- Identical shadow (rgba(0,0,0,.1)) soft card on everything

If a direction uses one of these because the reference genuinely does, keep it and
document: `[kept — reference uses it: path/to/ref.png]`

---

## STEP 4 — PROTOTYPE GENERATION

One self-contained HTML file per direction. Write to `$PROTO/[Direction-Name].html`.

### Structure
- Single scrolling page by default.
- Multi-page (separate linked HTML files) only when the brief clearly requires distinct
  application-level pages (case studies, per-product pages, logged-in dashboards).
  If multi-page: `$PROTO/[Direction-Name]/index.html`, `work.html`, etc.
- Nav wired: smooth-scroll `href="#section-id"` anchors for single-page;
  relative `href` links for multi-page.

### CSS at the top — annotated token block
```css
:root {
  /* palette.py: [ref path] */
  --color-bg:         #______;
  --color-surface:    #______;
  --color-text:       #______;
  --color-text-muted: #______;
  --color-accent:     #______;
  --color-border:     #______;

  /* typography */
  --font-display: '[Family]', sans-serif;
  --font-body:    '[Family]', sans-serif;
  --font-label:   '[Family]', monospace;

  /* spacing */
  --unit: Npx;
}
```

Every color must trace to a palette.py output line — no guessing.

### Contrast check
Before generating the HTML, verify text/background pairs pass WCAG AA (4.5:1 body,
3:1 large text). If a pair fails, adjust until it passes and note it:
`/* adjusted from #B0ADA6 → #6B6860 for WCAG AA */`

### Fonts
Load via Google Fonts CDN. Add a visible comment at the top:
```html
<!-- PREVIEW FONTS: [Family] loaded via Google Fonts CDN.
     Replace with self-hosted version in production (Ethiopian CDN latency). -->
```

### Images
Primary: library images by absolute path, for sections where the reference fits:
```html
<img src="/Users/dhaban/Documents/Design Work/.../Portfolio/342. Linear.png" alt="">
```
Secondary: for sections without a matching reference, picsum with a stable seed:
```html
<img src="https://picsum.photos/seed/[direction-name]-hero/1200/700" alt="">
```
Use the same seed throughout one prototype for visual consistency.

### Fidelity requirements
- Polished spacing — not approximate. Use the spacing unit consistently.
- Hover states on all interactive elements.
- Transitions — only what the reference actually showed, or what the structural concept
  implies. No fade-and-slide-up on every section.
- Real copy from the intake — no placeholders.
- Responsive: must work at 375px minimum, via a CSS breakpoint, not a separate design.
- Visible keyboard focus styles.

### Before writing each prototype, state aloud:
```
Direction: [Name]
Fingerprint: [one line]
Axis vector: [the 9 values]
References: [paths]
Key deviation from AI defaults: [what you changed and why]
Palette source: [palette.py outputs → CSS vars]
```

---

## STEP 5 — CANVAS: index.html

Write `$PROTO/index.html` — the interactive switcher.

```
┌─────────────────────────────────────────────────────┐
│  [Left-Anchor] [Editorial-Grid] [Full-Bleed] ...    │  ← fixed top bar, prototype chips
│                                           [M] [📋]  │  ← mobile toggle, info panel
├─────────────────────────────────────────────────────┤
│                                                     │
│                   <iframe>                          │  ← full-height, current prototype
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Collapsible side panel (right, opens on 📋):**
```
Direction:    Left-Anchor
Fingerprint:  Fixed left sidebar, scrolling content...
Axes:         layout_system: split-screen · navigation_model: sidebar-vertical · ...
References:   Portfolio/342.png · Agency-Studio/124.png
Sections:     [Hero] [Work] [About] [Contact]   ← section jump buttons

[Select this direction →]
```

**Behaviors:**
- Keyboard: `1`–`5` switch prototype, `M` toggles mobile (375px iframe), `I` toggles panel.
- Mobile toggle shrinks the iframe to 375px and centers it. Label: "Mobile preview — 375px."
- Section jump calls
  `iframe.contentWindow.document.querySelector('#section-id').scrollIntoView()`.
- "Select this direction →" posts back via
  `window.parent.postMessage({ selected: directionName }, '*')`, with a visible fallback:
  "or type: go with [Direction-Name]"
- Active chip visually distinct (filled vs outlined).
- No chrome, no commentary — the info panel is the only metadata.

---

## STEP 6 — SELECTION + REFINEMENT

**Selection** happens when the user clicks "Select" or types
`go with [Direction-Name]` / `pick [Direction-Name]`.

**Before any refinement:** copy the current prototype to
`$PROTO/versions/[Direction-Name]-v1.html`. Increment on every subsequent refinement.

**Refinement is in-place** — modify the chosen HTML directly. The user gives feedback
conversationally ("darken the background, make the display type heavier, remove the hover
underline on nav"). Apply all changes in one pass, re-save, and state what changed and
why — one line per change.

**Cross-prototype mixing:** "I like [A]'s hero and [B]'s work section" → generate
`$PROTO/[A]-x-[B].html`. Version-copy before any further refinement. This is now the
selected prototype.

**Revert:** `revert to v[N]` → copy `versions/[Name]-v[N].html` back to the active file.

---

## STEP 7 — HANDOFF: DESIGN.md

After refinement is done ("done", "looks good", "build it", "hand off"), write `DESIGN.md`
to the **project root** (not inside `_prototypes/`).

```markdown
# DESIGN.md
Project: [name]
Generated: [date]
Prototype: [Direction-Name] (from inspo skill)
Fingerprint: [structural one-liner]
Axis vector: [the 9 structural values]

## What makes this distinct
[2–3 sentences — the conceptual idea, specific to this project. Not "clean and modern."
What would make someone recognise this as THIS project.]

## Token System

### Colors
```css
--color-bg:         #______ /* palette.py: [ref path] */
--color-surface:    #______ /* palette.py: [ref path] */
--color-text:       #______ /* palette.py: [ref path] */
--color-text-muted: #______ /* palette.py: [ref path] */
--color-accent:     #______ /* palette.py: [ref path] */
--color-border:     #______ /* palette.py: [ref path] */
```

### Typography
| Role    | Family | Weight | Size | Line-height | Tracking |
|---------|--------|--------|------|-------------|----------|
| Display | ...    | ...    | ...  | ...         | ...      |
| Body    | ...    | ...    | ...  | ...         | ...      |
| Label   | ...    | ...    | ...  | ...         | ...      |

Fonts used: [list]. Self-host in production — do not rely on Google Fonts CDN
(Ethiopian latency). Download from fontsource.org or serve from `/public/fonts/`.

### Spacing
Unit: Npx · Scale: [named steps used: sm=4px, md=8px, lg=16px, xl=32px...]

### Motion
[Named transitions: property duration easing — purpose]
[If none: "No decorative motion. Hover states only."]

## Layout System
Grid: [columns] · [max-width] · [gutter]
Alignment: [left / center / justified — and where each applies]
Responsive breakpoints: [list with values]
Page architecture: [single-page / multi-page — and why]

## Page Sections
### [Section Name]
- Layout: [structure]
- Content: [what goes here]
- Component: [which component pattern handles it]
- Copy: [the actual copy from intake]

## Components
### [Component Name]
Structure: [HTML pattern — clear enough to implement]
Variants: [if any]
State: [hover / active / disabled]

## Content Hierarchy
[heading → subheading → body → CTA per section, with the actual copy text]

## Build Notes
- Font self-hosting: required (see Typography)
- Image optimization: [requirements from the prototype]

## Reference Attribution
Hero: [path] (category · source)
Work section: [path] (category · source)

## AI-Default Audit — Final State
[Per item: ☐ not present / ☑ present → kept because [ref] / ☑ present → changed to [X]]
```

---

## GUARDRAILS

- **Round one is never one interface.** 3–5 structurally distinct directions, or say why
  fewer. Never pad to hit a number.
- **Divergence is measured, not asserted.** Quote `min pairwise N/9` and each direction's
  axis vector. If two directions share `navigation_model` + `content_hierarchy` +
  `layout_system`, they are one direction.
- **Never pass `--include-unverified`** to `structure_search.py`. That tier scored 51–56%
  in blind audits and contains records bound to the wrong image.
- **A tag is a candidate to view, never a fact to build from.** View every reference before
  it informs code.
- **AI-default pre-flight is mandatory** per direction, before generating. Not a checklist
  to note and ignore.
- **Library is read-only** — reference by absolute path. Never copy files in; never write
  tool output into `$LIB` (pass an absolute `--output` under `$PROTO`).
- **No lorem ipsum, no placeholder headlines** — copy comes from the intake. If intake was
  vague, generate specific realistic copy and state what you assumed.
- **Palette must come from palette.py** — every `--color-*` cites its source image.
- **Contrast must pass WCAG AA** before generating, not after.
- **Refinement is always versioned** — never modify the active prototype without copying
  the previous state to `versions/`.
- **DESIGN.md is the build source of truth** — complete enough that a build agent with no
  context can start from it alone. If a decision isn't in DESIGN.md, it wasn't made.
- **If the library has no genuine match**, say so — still generate from principles, but mark
  those directions `[no library reference — generated from principles]` in the canvas panel
  and in DESIGN.md.
- **Skip entirely for non-visual work** — backend, API, copy-only, data tasks.
- **This runs automatically** — the trigger is the design brief, not a separate "check the
  library" instruction.
