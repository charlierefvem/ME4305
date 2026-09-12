# ME4305 Note Style Guide

This guide describes the durable editorial conventions for the ME4305 Obsidian notes. It complements the enforceable repository instructions in [AGENTS.md](AGENTS.md).

## Editorial Posture

The notes support the instructor's teaching rather than replace it. Preserve the instructor's voice and technical choices. Improve organization, readability, formatting, consistency, and implementation quality without introducing unsupported course content.

A generated or substantially revised note is a penultimate draft. Use `TODO (Instructor Review):`, `Editorial Note:`, or `Technical Note:` when uncertainty cannot be resolved from authoritative sources.

## Note Families

### Topics

Topics follow the student-facing lecture sequence and should read coherently from start to finish. A topic may use sections such as Motivation, Core Idea, Technical Development, nearby Examples, Implementation Notes, Practical Notes, Summary, Related Topics, and Candidate Static Notes. Use only the sections that fit the material.

Topics may link to any note family, but links should add depth rather than outsource essential lecture flow.

### References

References support arbitrary access. Define their scope and assumptions clearly, develop the concept, place examples near the relevant explanation, and keep course-specific or robot-specific context in labeled examples. References should generally link only to other references.

### Case Studies

Case studies preserve the path from an engineering problem to an outcome. Include relevant modeling choices, intermediate decisions, tradeoffs, rejected approaches, tuning or implementation sequence, validation, and lessons learned. Link reusable theory instead of reproducing an entire reference treatment.

### Perspectives

Perspectives collect substantial instructor interpretation: design judgment, recurring misconceptions, intuition, tradeoffs, and lessons that deserve a standalone reading experience. A small local observation belongs in an insight callout instead.

## Frontmatter

Use Obsidian-compatible YAML frontmatter. Include these fields when appropriate:

```yaml
---
title: Descriptive Note Title
type: topic
tags:
  - example-tag
source:
  course: ME4305
status: draft
---
```

Valid note-family values are `topic`, `reference`, `case-study`, and `perspective`. Preserve more detailed provenance when adapting legacy ME405 material.

## Headings and Narrative

- Use a descriptive title in frontmatter; begin note content at level-two headings unless the local note convention requires otherwise.
- Prefer short sections that each advance one idea.
- Put examples immediately after the concept they demonstrate.
- Integrate equations into explanatory prose and define symbols, units, assumptions, and sign conventions.
- Explain why a method is useful, where it fails, and what physical or computational limitation matters most.
- Keep lecture-facing topics self-contained enough to read without following every link.
- Avoid empty template sections and excessive heading depth.

## Links

Use Obsidian wiki-links where they improve navigation.

```text
topic       -> topic, reference, case-study, perspective
reference   -> reference
case-study  -> topic, reference, case-study, perspective
perspective -> topic when context matters, reference, case-study, perspective
```

Do not create empty reference stubs merely because a phrase might someday deserve a page. Mark an uncertain candidate for instructor review.

## Equations and Code

- Use LaTeX for mathematical notation and `$$ ... $$` for display math.
- Preserve derivation order and connect each result to physical meaning or implementation.
- Prefer executable code and typed source when available.
- Preserve language identifiers on fenced code blocks.
- Ordinary code fences are the default for short snippets, one-line expressions, API calls, syntax fragments, and intermediate steps.
- Use numbered listings only when the code is a substantial unit worth identifying and referring to.

## Callouts

The source CSS files in `notes/.obsidian/snippets/` contain the authoritative rendering examples.

### Notes and Insights

Use a note for supporting context and an insight for a conceptual observation or design principle.

```markdown
> [!note]
> This sentence begins normally; it does not repeat the callout label.

> [!insight]
> The physical interpretation matters more than the algebraic form alone.
```

Do not write `**Note:**`, `**Insight:**`, `Note:`, or `Insight:` inside the callout body. When converting an existing labeled paragraph, remove the complete label—including bold markers and colon—and repair the first sentence's capitalization.

### Algorithms

Use an algorithm callout when a sequence of steps or pseudocode should be recognized as a procedure.

```markdown
> [!algorithm] Rollover compensation
> 1. Read the current counter value.
> 2. Compute the difference from the previous value.
> 3. Correct the difference when rollover is detected.
```

Do not use a numbered code listing merely to style an ASCII diagram or pseudocode block.

### Code and File Listings

Reserve numbered listings for complete implementations, examples longer than a few lines, or code important enough to cite elsewhere in the note.

````markdown
> [!block_listing] Reading accelerometer registers over I2C
> ```python
> # Substantial example here
> ```

> [!file_listing] motor_driver.py
> ```python
> # Complete or meaningful file excerpt here
> ```
````

Every listing needs a meaningful description or filename. Avoid numbered listings for:

- one-line expressions or method calls;
- syntax templates;
- brief illustrative fragments;
- intermediate steps discussed only in the adjacent paragraph;
- ASCII diagrams or other preformatted prose.

Use an ordinary fenced `text` block for diagrams or styled text:

````markdown
```text
sensor -> observer -> controller
```
````

### Program Output

Use an output callout for console or program output. It can follow either a numbered listing or an ordinary code block.

````markdown
> [!output]
> ```text
> Measurement complete
> ```
````

### Figures

Keep detailed accessibility information in alt text and make the visible caption concise.

```markdown
> [!figure]
> ![Detailed description of the figure's components and relationships.](example.svg)
> Short visible description of the figure.

Following prose begins after a blank line.
```

Rules:

- Use exactly one concise caption paragraph, ideally one short sentence.
- Keep the caption on one quoted Markdown line.
- Leave a blank line after the complete callout.
- Do not retain generic placeholders such as `Figure Caption` when the alt text or context supports a real caption.

### Tables

Place a concise caption in the table callout and leave a blank line after it.

```markdown
> [!table]
> Common timer functions and their purposes.
>
> | Function | Purpose |
> | --- | --- |
> | `ticks_us()` | Read the microsecond tick counter. |

Following prose begins after a blank line.
```

Use one short caption sentence on one quoted line. Do not repeat the caption outside the callout.

## Figures and Assets

- Use meaningful alt text that describes the important content and relationships.
- Keep visible captions shorter than alt text.
- During source conversion, use a clear image placeholder with detailed alt text when an asset is unavailable; do not recreate the figure unless explicitly requested.
- Preserve editable source figures under `figures/` and note-facing assets under `notes/Images/` according to the existing conversion workflow.
- PDFs intended for SVG conversion should use outlined or flattened text.
- Preserve local naming and paths unless a task explicitly settles a new repository-wide convention.

## Technical Emphasis

Relate formal methods to systems students can build and debug. Recurring connections include:

- noisy or quantized measurements motivating filters and observers;
- saturation motivating anti-windup;
- sampled firmware motivating discrete-time control;
- non-holonomic motion motivating path and trajectory planning;
- timing, interrupts, and scheduling constraining firmware design;
- sensor, actuator, power, and communication limits shaping system behavior.

Advanced material may be mentioned when it clarifies the larger design space, but substantial Kalman filtering and advanced systems integration should not displace the main ME4305 arc.

## Final Editorial Check

Before handing off a note, verify:

- the note type and reading path are clear;
- the instructor's content and voice are preserved;
- equations, units, signs, assumptions, and code remain intact;
- examples remain near the concepts they support;
- links follow the note-family rules;
- captions are concise and callout boundaries are valid;
- short snippets are not over-promoted to numbered listings;
- note and insight bodies contain no duplicated labels or orphaned Markdown;
- uncertainties are explicit and collected for instructor review.
