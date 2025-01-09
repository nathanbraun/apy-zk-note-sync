# apy-zk-note-sync

This Python script uses the [apy](https://github.com/lervag/apy)
Anki-connecting Python library to add questions (written in
[zk](https://github.com/zk-org/zk-nvim) note files) to Anki.

## Context
I organize all my notes in markdown/vimwiki files using [zk and
Neovim](https://github.com/zk-org/zk-nvim) (more on how I do this is described
in my book [Tech Tools](https://techtoolsbook.com)).

Basically, each note is one of some "type" — a day, book, project, concept,
meeting, topic, etc — and linked together by links to topic notes.

## Anki Notes
With this setup, I've added *anki* notes, which contain Anki notes in the
markdown format readable by [apy](https://github.com/lervag/apy/):

```markdown
Here is the example Markdown input:

// example.md
model: Basic
tags: marked

# Note 1
## Front
Question?

## Back
Answer.

# Note 2
tag: silly-tag

## Front
Question?

## Back
Answer

# Note 3
model: NewModel
markdown: false (default is true)

## NewFront
FieldOne

## NewBack
FieldTwo

## FieldThree
FieldThree
```

### Cloze/Poker Example
So for example, if I were trying to memorize standard Poker opening ranges
(which depend on the cards dealt and the "position" you're sitting in at the
table — High Jack, Cutoff, etc) I could make these two notes:

Note I like to use Cloze cards. My Anki Model for Cloze just has the one field:
`text`.

```dp4s-anki.wiki
---
title: Poker HJ hands
date: 2025-01-08
synced: false
tags: [anki]
---

# Note
model: Cloze
tags: poker
deck: poker

## Text
HJ 28% RFI Range
8 players, 100 BB

RFI with A{{c1::X}}s

# Note
model: Cloze
tags: poker
deck: poker

## Text
HJ 28% RFI Range
8 players, 100 BB

RFI with K{{c1::4+}}s
```

```lx43-anki.wiki
---
title: Poker CO RFI Hands
date: 2025-01-09
synced: false
tags: [anki]
---

# Note
model: Cloze
tags: poker
deck: poker

## Text
CO 36% RFI Range
8 players, 100 BB

RFI with A{{c1::X}}s

# Note
model: Cloze
tags: poker
deck: poker

## Text
CO 36% RFI Range
8 players, 100 BB

RFI with K{{c1::X}}s
```

Each of these has two Anki cards. Note my real note has more.

What the script in this repository does is:

1) Goes through all the notes named `XXXX-anki.wiki` in my notes directory.
2) Finds the one's where the `synced` option in the YAML header is `false`.

Then for each:

3) Adds the notes/questions in the file to Anki using the `apy`
   `add_notes_from_file` method.
4) Changes the `synced` option to `true`.

## Misc
I have some snippets to easily add notes in the 

```
# Note
model: Cloze
tags: 
deck: 

## Text
```

format, as well as to easily select some text and turn it into a cloze field. I
also have some zk related bindings to quickly create (or open, or link to) Anki
specific notes. Again, see my [Tech Tools](https://techtoolsbook.com)) for
more.
