# ME4305 Note Workflows

These workflows translate [AGENTS.md](AGENTS.md) and [NOTE_STYLE_GUIDE.md](NOTE_STYLE_GUIDE.md) into repeatable editing steps.

## Revise an Existing Note

1. Read the whole note before editing isolated sections.
2. Identify its note family and intended student reading path.
3. Inspect directly linked notes and nearby notes of the same family for local conventions.
4. Separate structural or formatting work from technical rewriting.
5. Preserve equations, derivations, code, examples, figures, units, signs, and implementation details.
6. Improve section order and transitions only as needed for the stated task.
7. Add callouts and links selectively according to the style guide.
8. Mark unresolved technical or editorial uncertainty explicitly.
9. Review the complete diff and report instructor-review items.

When a task is specifically limited to formatting or callout adoption, avoid opportunistic prose edits.

## Convert Source Material into a Note

Use the following source priority:

1. Explicit instructor instructions.
2. Handwritten lecture notes.
3. Annotated lecture slides.
4. Existing digital notes.
5. Historical summaries or conversations.

Then:

1. Record provenance in frontmatter or a clearly labeled source note.
2. Choose `topic`, `reference`, `case-study`, or `perspective` from purpose—not from the source file's format.
3. Preserve the source's teaching sequence and technical artifacts.
4. Turn fragments into clear prose without expanding the material into a textbook chapter.
5. Reconstruct incomplete code only when intent is clear; flag meaningful corrections.
6. Add detailed figure alt text and a concise visible caption.
7. Use instructor-review markers instead of inventing missing facts.

## Create a New Note

1. Confirm that a new page is justified and not merely an empty future reference.
2. Select the note family and filename using current local conventions.
3. Add frontmatter with title, type, tags, source, and status as appropriate.
4. Write the smallest coherent structure that supports the material.
5. Connect formal results to physical behavior and implementation.
6. Add only links that improve the reading experience.
7. Treat the result as a draft for instructor review.

## Adopt or Audit Callouts

1. Read the three relevant CSS snippets before changing syntax:
   - `notes/.obsidian/snippets/minimal_callout.css`
   - `notes/.obsidian/snippets/listing_callout.css`
   - `notes/.obsidian/snippets/caption_callout.css`
2. Wrap figures and tables and add concise, one-line captions.
3. Leave a blank line after each complete figure or table callout.
4. Convert explicitly marked supporting remarks or design principles to note or insight callouts where appropriate.
5. Remove the full redundant body label, including any surrounding `**` and colon, and restore sentence capitalization.
6. Keep short code and preformatted text as ordinary fences.
7. Promote only substantial code to `block_listing` or `file_listing` and supply a meaningful title.
8. Use `output` for console output and `algorithm` for procedures or pseudocode.
9. Check for malformed or nested callouts, unbalanced fences, placeholder captions, and accidental content changes.

Useful targeted searches include:

```powershell
rg -n '^> \[!' notes
rg -n '^```|^!\[|^\|' notes/Topics notes/References 'notes/Case Studies' notes/Perspectives
rg -n -i '^> (\*\*)?(Note|Insight)(:\*\*|\*\*:|:)' notes
rg -n 'Caption Goes Here|Figure Caption|Table Caption Goes Here|\[REVIEW:' notes
```

Interpret search results in context: ordinary fenced code is valid, pipe characters may occur in ASCII diagrams, and hidden Obsidian comments may contain editorial scaffolding.

## Add or Update a Figure

1. Keep the editable source under `figures/`.
2. Export a PDF with text outlined or flattened when it will enter the automated pipeline.
3. Preserve the relative subdirectory so the workflow generates the corresponding SVG under `notes/Images/`.
4. Confirm the image link uses the repository's current local convention and exact path case expected by the publishing environment.
5. Add detailed alt text and a concise visible caption in a figure callout.
6. Validate the generated SVG and review the note in Obsidian or the target published rendering.

Do not bulk-rename figure sources or note-facing assets while the final path convention remains open.

## Technical Review

For equations and models, check:

- symbols and units are defined;
- sign conventions and coordinate frames are consistent;
- assumptions are stated near the derivation;
- continuous- and discrete-time forms are not mixed casually;
- the implementation matches the mathematical sequence;
- limitations such as sampling, noise, saturation, quantization, and conditioning are visible.

For code, check:

- the language fence is correct;
- indentation and imports are intact;
- the example is executable when claimed;
- hardware-specific assumptions are explicit;
- blocking, timing, allocation, and interrupt behavior are discussed when relevant;
- a numbered listing is substantial enough to deserve a number.

## Pre-Handoff Checklist

- Review `git status` and preserve unrelated changes.
- Inspect the diff for unrequested prose or technical changes.
- Check YAML frontmatter and Markdown fence balance.
- Check figure and table callout boundaries and following blank lines.
- Check caption length and remove placeholders when context is sufficient.
- Check note and insight openings for duplicated labels, orphaned `**`, residual colons, and lowercase sentence starts.
- Check links and image paths without silently settling an open naming convention.
- Run any relevant build or rendering workflow available for the task.
- Return a concise list of every `TODO (Instructor Review):`, `Editorial Note:`, `Technical Note:`, or unresolved placeholder introduced or encountered.
