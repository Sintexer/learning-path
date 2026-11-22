# Learning Workflow

Use this workflow when you study a new topic and add notes to this vault.

## 1. Capture Before You Organize

1. Add raw topics, book terms, or production questions to [[00 - info stack]].
2. Do not create empty `0B` stub notes for topics you have not studied.
3. Use unresolved wikilinks (for example `[[Fencing Token]]` or `[[Write Skew]]`) inside existing notes to mark knowledge gaps.

## 2. Create One Atomic Note Per Concept

1. Pick one task from [[00 - info stack]] or one unresolved link from your graph.
2. Place the note in its domain folder so `flashcards-obsidian` assigns the correct Anki deck.
3. Keep one concept per note. Split a note when it grows beyond 3–4 screens or covers multiple independent mechanisms.

Use this minimal structure:

```markdown
One-paragraph definition in your own words.

## How it works

Short explanation, example, or invariant.

> [!warning] Failure modes & trade-offs
> List real production risks, limits, or common traps.

## Related

- [[Parent or sibling concept]]
```

## 3. Add Cross-Links Naturally

1. Link existing notes inline with pipe aliases (`[[Target Note|natural phrase]]`) so sentences stay readable.
2. Add 2–4 links under `## Related` to connect the note to its parent MOC and adjacent concepts.
3. Update the folder MOC (`🗂️ ...`) when you add a major topic.

## 4. Write 2–5 High-Signal Anki Cards

Follow the rules in [[Anki flashcards]]:

- Use `Question::Answer` for single facts.
- Use `Term:::Definition` only when recall in both directions has value.
- Use `==cloze==` for formulas, invariants, and trade-offs.
- Use `## Question? #card` for failure modes, mechanisms, and comparisons.
- Use `Rule. #card-reminder` for Senior/Staff design heuristics.

Do not place card markers inside code blocks, math blocks, or blockquotes. Do not edit `^q-xxxx` anchors or `flashcards:` frontmatter by hand.

## 5. Sync and Close the Loop

1. Open Anki and run **Flashcards: Update Anki from current note**.
2. Check off the completed item in [[00 - info stack]] and replace the plain text with a link to your new note.
3. Review Anki daily. If a card feels vague or too long during review, edit the source note and resync.

## 6. Weekly Maintenance (10 Minutes)

1. Open Graph View and inspect the most-linked unresolved nodes. Promote the top 1–2 into [[00 - info stack]] Tier 1.
2. Revisit one older note you used at work this week. Add one production lesson or failure mode.
3. Commit your vault changes when a study session ends.
