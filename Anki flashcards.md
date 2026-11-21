## Quick start

Write a basic card in any Markdown note:

```markdown
What is the capital of France?::Paris
```

Keep Anki open. Then run **Flashcards: Update Anki from current note** from the Obsidian command palette. You can also use the Flashcards button in the left sidebar.

The plugin adds a small `^q-xxxx` anchor and a managed `flashcards` property. They connect the Obsidian card to its Anki note. Do not edit them by hand.

## Card types

### Basic

```markdown
What does HTTP mean?::Hypertext Transfer Protocol
```

### Reversed

```markdown
TCP:::Transmission Control Protocol
```

A reversed note creates cards in both directions. Use it only when both directions are useful.

### Cloze

```markdown
The mitochondria is the ==powerhouse== of the cell.
The {1:heart} pumps blood through the {1:circulatory system}.
```

`==text==` creates numbered clozes in source order. `{N:text}` lets you choose the number. Reuse a number when several parts should be hidden together.

### Heading card

```markdown
## What is recursion? #card

A function that calls itself and has a base case.
```

A heading card can contain several paragraphs, lists, code blocks, and lower headings. The next heading at the same or a higher level ends the answer.

### Reminder

```markdown
Prefer reversible decisions when uncertainty is high. #card-reminder
```

A reminder has one piece of content and no answer. It returns through Anki's normal schedule.

### Explicit multi-line card

````markdown
```flashcard
front: What does CSS stand for?
back: Cascading Style Sheets
type: reversed
```
````

The default `type` is `basic`. Other values are `reversed`, `cloze`, and `reminder`. Reminder blocks use `content:` instead of `front:` and `back:`.

The plugin also supports card callouts, cards in lists, and atomic note cards. See the [v2 wiki](https://github.com/reuseman/flashcards-obsidian/blob/HEAD/docs/wiki.md) for their exact rules and examples.

## Markdown content

Cards can contain:

- emphasis, links, lists, quotes, and code;
- LaTeX math;
- Obsidian wikilinks, including heading and block links;
- images and audio attachments.

Card markers inside code, math, HTML comments, and ordinary blockquotes are not parsed as new cards.

## Decks, tags, and context

Set a deck for one note with frontmatter:

```yaml
---
cards-deck: Knowledge::Biology
tags:
  - biology
  - exam
---
```

By default:

- the note folder selects the deck when `cards-deck` is absent;
- the fallback deck is `Default`;
- every card gets the `obsidian` tag;
- parent headings appear as context above the review question;
- removing a linked card asks for confirmation before Anki deletes it.

You can change these defaults in the Flashcards settings.