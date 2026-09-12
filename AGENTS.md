# ME4305 Repository Guidance

These instructions apply to the entire repository. More specific `AGENTS.md` files, if added later, override this file within their directories.

## Project Purpose

This repository supports ME4305, Mechatronics II, at Cal Poly San Luis Obispo. ME4305 follows ME3305 and precedes ME5305. Its central artifact is an Obsidian-compatible collection of course notes for primarily fourth-year mechanical engineering students concentrating in mechatronics.

Treat mechatronics as an integrated engineering practice. Connect mechanics, electronics, programming, sensing, actuation, controls, timing, and system limitations instead of presenting them as isolated subjects. The Romi differential-drive robot is a recurring instructional anchor, not the identity of the course.

## Audience and Complexity

Most students have some preliminary programming experience, but do not assume that they know Python. The majority should have encountered MATLAB and S12X assembly in prior coursework. Treat that background as useful exposure rather than guaranteed mastery, and introduce Python-specific behavior and terminology before relying on it.

- Aim for **medium-low to medium complexity** in both prose and code.
- Assume students recognize basic ideas such as variables, conditionals, loops, functions, and arrays, but may need help transferring those ideas between MATLAB, assembly, and Python.
- Introduce unfamiliar concepts in a concrete progression: define the term, give a usable mental model, show a small example, and explain the practical consequence.
- Use MATLAB or assembly comparisons selectively when they clarify an important distinction. Do not require students to translate every explanation through another language.
- Prefer precise, approachable explanations over simplified slogans that create misconceptions. Reduce complexity through structure, examples, and plain language rather than by sacrificing technical correctness.
- Include more advanced detail when it prevents a common bug, supports an implementation students will encounter, or provides a worthwhile opportunity for growth. Keep the core idea visible when doing so.

Use `notes/References/reference_mutability.md` as an approximate upper-bound benchmark for depth and conceptual difficulty. Its deepest material may slightly exceed what some students understand on a first reading, which is acceptable when the main explanation remains accessible and the additional detail encourages growth. Most notes should remain at or below this level unless the subject genuinely requires greater depth.

## Authorship and Authority

The notes are the instructor's intellectual work. Act primarily as a technical editor, formatter, organizer, and implementation assistant—not as the primary author.

- Preserve the instructor's voice, teaching sequence, examples, equations, and technical judgment.
- Do not expand lecture material into an AI-authored textbook chapter.
- Do not invent course facts, policies, technical claims, examples, or cross-course relationships.
- Do not silently correct suspected technical errors. Flag them with `Technical Note:` or ask the instructor.
- Treat generated or substantially revised notes as drafts for instructor review.

For source conversion and conflicting information, use this priority:

1. Explicit instructor instructions.
2. Handwritten lecture notes.
3. Annotated lecture slides.
4. Existing digital notes.
5. Previous conversations and historical summaries.

Legacy ME405 material is historical source material, not current ME4305 policy. Preserve its provenance when reused, but use ME4305 as the current course identity.

## Repository Layout

- `notes/` is the Obsidian vault and contains student-facing notes and published assets.
- `notes/Topics/`, `notes/References/`, `notes/Case Studies/`, and `notes/Perspectives/` contain the four supported note families.
- `notes/Images/` contains note-facing image assets.
- `figures/` contains editable or publication-source figure files.
- `.github/workflows/pdf-to-svg.yml` converts source PDFs under `figures/` into SVGs under `notes/Images/`.
- `build/` and `docs/` are generated output and are ignored by Git.
- Root guidance belongs in root Markdown files, not inside the Obsidian vault.

Do not edit `notes/.obsidian/workspace.json` or other personal workspace state unless explicitly requested. Preserve unrelated user changes in the working tree.

## Note Types

Choose a note type from its purpose and reading pattern before choosing its outline.

### Topic

A lecture-facing engineering narrative organized in the order students encounter ideas. It should be readable from beginning to end without requiring students to follow links. Use **Topic**, not **Lecture**, as the primary instructional unit.

### Reference

A standalone, reusable explanation of definitions, assumptions, methods, derivations, models, or implementation forms. References should remain independent of a particular lecture, assignment, robot, or term except for clearly labeled examples. They should generally link only to other references and should not link back to topics.

### Case Study

A long-form worked example, design story, tuning procedure, or implementation journey. Preserve modeling choices, intermediate decisions, tradeoffs, useful rejected approaches, implementation sequence, and lessons learned. Link reusable theory to references.

### Perspective

A standalone treatment of substantial instructor judgment, intuition, misconceptions, tradeoffs, or lessons learned. Small observations belong in local insight callouts rather than a separate `insight` note family.

Topic and reference structures may use stable templates. Keep case-study and perspective structures flexible until more examples establish durable patterns.

## Linking and Reuse

Use links to add depth without fragmenting the primary reading experience.

```text
topic       -> topic, reference, case-study, perspective
reference   -> reference
case-study  -> topic, reference, case-study, perspective
perspective -> topic when context matters, reference, case-study, perspective
```

When a topic contains substantial reusable theory, consider extracting a concise reference and linking to it. Do not create large reference pages or empty stub forests automatically. Mark an uncertain candidate for instructor review.

## Writing and Technical Style

- Keep topic pages coherent and self-contained enough to support a lecture.
- Preserve technical artifacts: equations, derivations, code, algorithms, examples, figures, warnings, implementation details, and lab-specific instructions.
- Place examples near the concepts they support.
- Use short, descriptive sections rather than forcing every note into an identical outline.
- Connect mathematics to physical behavior, measurement, actuation, computation, timing, and design tradeoffs.
- State assumptions, explain why a method exists, identify where it fails, and name the dominant limitation.
- Prefer executable code when practical. Reconstruct incomplete code only when its intent is clear, and disclose corrections that affect meaning or executability.
- Use typed source code directly when available rather than re-OCRing it.
- Preserve units, sign conventions, causality, validation, documentation, and debugging discipline.
- Treat sampling, saturation, quantization, noise, friction, stiction, deadband, sensor limits, power limits, and timing as engineering content.
- Avoid turning ME4305 into a controls-theory, robotics-algorithms, electronics, or programming course in disguise. Favor correct, transferable engineering understanding over unnecessary mathematical completeness.

Make uncertainty visible with a concise `TODO (Instructor Review):`, `Editorial Note:`, or `Technical Note:`. Accumulate unresolved review items and report them when handing work back.

## Markdown and Obsidian

- Use Obsidian-compatible Markdown and wiki-links where they improve navigation.
- Include YAML frontmatter with at least `title`, `type`, `tags`, `source`, and `status` when appropriate.
- Use LaTeX for equations and `$$ ... $$` for display math.
- Keep detailed accessibility text in Markdown image alt text.
- Preserve the repository's local naming and asset-path conventions. Do not bulk-rename notes or assets without explicit direction.
- Do not add empty template headings merely to satisfy an outline.

See [NOTE_STYLE_GUIDE.md](NOTE_STYLE_GUIDE.md) for note structures and exact callout conventions.

## Callout Rules

The active callout snippets are documented in:

- `notes/.obsidian/snippets/minimal_callout.css`
- `notes/.obsidian/snippets/listing_callout.css`
- `notes/.obsidian/snippets/caption_callout.css`

Use callouts deliberately rather than wrapping every visually distinct block.

- Use `figure` and `table` callouts for figures and Markdown tables. Give each one a concise, single-line caption and leave a blank line after the callout.
- Keep detailed description in image alt text; the visible caption should ideally be one short sentence.
- Use `note` for supporting information and `insight` for a conceptual observation or design principle.
- Begin note and insight bodies as ordinary sentences. Do not repeat `Note:`, `Insight:`, bold markers, or a colon in the body. Repair capitalization after removing a label.
- Use `algorithm` for a procedure or pseudocode whose procedural identity matters.
- Reserve `block_listing` and `file_listing` for complete implementations, examples longer than a few lines, or code important enough to reference elsewhere.
- Leave one-line expressions, API calls, syntax fragments, and intermediate steps as ordinary fenced code blocks.
- Leave ASCII diagrams and styled/preformatted prose as ordinary fenced `text` blocks.
- Use `output` for console or program output. It may follow a numbered listing or an ordinary code block.
- Give every numbered listing a meaningful description or filename.

When a task is limited to callout adoption or formatting, do not alter prose or technical content except for requested captions, listing labels, or the minimal sentence repair required after removing a redundant callout label.

## Figures and Generated Assets

Authoritative editable figure sources belong under `figures/`. The existing GitHub workflow converts PDFs to SVG files in `notes/Images/` and rejects generated SVGs that retain text elements or font-family declarations.

- Preserve editable sources and generated outputs.
- Prefer outlined or flattened text in PDFs intended for SVG conversion.
- Do not manually overwrite generated assets without understanding the conversion workflow.
- Do not assume the final asset naming and path convention is settled; follow newer local guidance when it exists.

## Working Method

For substantive note work:

1. Read the relevant source material, nearby notes, and local guidance.
2. Identify the note type and intended student reading path.
3. Preserve content and provenance while improving structure and clarity.
4. Add links and callouts only where they improve comprehension.
5. Validate Markdown structure, callout boundaries, captions, code fences, links, and asset paths.
6. Review the diff for accidental content changes and unrelated files.
7. Report unresolved instructor-review items explicitly.

See [NOTE_WORKFLOWS.md](NOTE_WORKFLOWS.md) for detailed checklists.

## Publishing Roles

- Canvas: administration and required student actions.
- GitHub Pages: durable course notes and technical explanations when the publishing workflow is reliable.
- GitHub repositories: code, libraries, and software distribution.
- Piazza or a similar forum: questions, discussion, and asynchronous help.

Do not rely on GitHub Pages embedded in a Canvas iframe as the primary reading experience.

## Open Decisions

Do not silently turn these into fixed policy:

- The final student-facing name for reusable/static reference notes.
- The final filename convention and treatment of legacy lecture numbering.
- How aggressively candidate reference stubs should be created and where editorial candidates should appear.
- The final figure and asset path convention and some SVG typography/export choices.
- The desired platform and depth for interactive code in published notes.
- Whether GitHub Pages replaces older distribution channels or runs alongside them during a transition.

Follow newer local guidance when available. Otherwise mark the uncertainty or ask the instructor when the decision materially affects the work.
