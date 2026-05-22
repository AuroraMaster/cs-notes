# Conventions

Small rules so the notes don't drift into mush.

## File layout

```
<section>/
  README.md            short overview + reading order for the section
  nn-kebab-name.md     individual notes, numeric prefix for sort order
```

## Note structure

Every note starts with a header block:

```markdown
# <Title>

**Summary** — one sentence, no waffle.
**Why I'm reading this** — why this topic, today.
**Status** — draft / reviewed / stale.
```

Then the body, then a `## References` section at the end.

## What goes in / what stays out

In:

- Mental models that survived after I closed the book.
- Diagrams, even ugly ASCII ones.
- Code that I actually typed and ran.
- "Things I got wrong the first time."

Out:

- Copy-paste from textbooks. If I can't restate it, I haven't learned it.
- Long preamble. Get to the point.
- Stale notes that no longer match my current understanding — rewrite them.

## Rewriting policy

When I re-read a note and disagree with my past self, I rewrite the file
in place rather than appending a "correction" section. The git history
preserves the old version.
