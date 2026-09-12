# ME4305

Course notes, figures, and publishing support for **ME4305: Mechatronics II** at Cal Poly San Luis Obispo.

ME4305 treats mechatronics as an integrated engineering practice. The material connects mechanics, electronics, embedded programming, sensing, actuation, real-time scheduling, feedback control, and system-level design decisions. The Romi differential-drive robot provides a common physical context while the notes emphasize principles that transfer to other systems.

## Repository Structure

- `notes/` — Obsidian vault containing the course notes and note-facing assets.
  - `Topics/` — lecture-facing instructional narratives.
  - `References/` — reusable technical explanations.
  - `Case Studies/` — worked design and implementation journeys.
  - `Perspectives/` — substantial instructor judgment, intuition, and tradeoffs.
  - `Images/` — images used by the notes.
- `figures/` — editable and publication-source figure files.
- `.github/workflows/pdf-to-svg.yml` — automated conversion of figure PDFs to SVG.
- `build/` and `docs/` — generated output, excluded from version control.
- `temp/` — temporary source and migration material; do not treat it as published course content.

## Project Guidance

- [AGENTS.md](AGENTS.md) contains repository-wide instructions for automated and human-assisted editing.
- [NOTE_STYLE_GUIDE.md](NOTE_STYLE_GUIDE.md) defines note types, linking, Markdown, callouts, and editorial style.
- [NOTE_WORKFLOWS.md](NOTE_WORKFLOWS.md) provides repeatable workflows and review checklists.

The notes are the instructor's intellectual work. Automated assistance should preserve the instructor's voice, examples, technical judgment, and teaching sequence. Substantial revisions should be treated as drafts requiring instructor review.

## Figure Pipeline

Editable figure sources live under `figures/`. When a PDF there changes, the GitHub Actions workflow converts it to a corresponding SVG under `notes/Images/`, validates that text has been outlined or flattened, and commits the generated asset.

Do not overwrite generated SVGs casually or bulk-change figure paths without reviewing the workflow and current note references.

## Publishing Intent

The durable technical notes are intended for a GitHub Pages publishing workflow when that workflow is reliable. Canvas remains the place for course administration and required student actions; repositories distribute code and libraries; discussion belongs in Piazza or a similar forum.

Several naming, asset-path, and publishing decisions remain intentionally open. See [AGENTS.md](AGENTS.md#open-decisions) before making repository-wide conventions permanent.
