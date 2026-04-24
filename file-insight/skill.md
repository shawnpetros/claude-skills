---
name: file-insight
description: "Promote a substantive insight, synthesis, or analysis generated in chat back into your notes so explorations compound instead of vanishing into transcript history. Lean skill, offers a single file destination and writes. Activated by: /file-insight, 'file this', 'save this analysis', 'promote this to the vault', 'this is worth keeping', or proactively offered when the assistant produces a multi-paragraph synthesis the user acknowledges as valuable."
---

# File Insight

Karpathy's Query operation has a payoff most people miss: **good answers can be filed back into the wiki as new pages.** A comparison you asked for, an analysis you synthesized, a connection you discovered ... these are valuable and shouldn't disappear into chat history. This skill is the promotion step.

Lean by design. No pipelines. No committees. Read the last substantive output, pick the right destination, write the file, update `index.md`, append to `log.md`. Done.

> **Vault setup required.** This skill writes into a personal knowledge base using the [Karpathy LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). See the [repo README](../README.md#using-these-skills-with-your-own-notes) for vault setup before using.

## When to trigger

**User-invoked:** `/file-insight`, "file this," "save this to the vault," "this is worth keeping," "promote this."

**Proactive:** After the assistant produces a multi-paragraph synthesis (comparison, architecture analysis, strategic breakdown, cross-reference discovery) AND the user acknowledges it as useful ("yeah, that's the answer," "great, makes sense," "perfect"), offer in one line:

> "Worth filing to the vault?"

No preamble, no pitch. One line. If the user says yes, proceed. If they ignore it, drop it. Don't ask twice.

## Process

### 1. Identify the insight

The "insight" is the substantive output from the most recent assistant turn. Usually one of:
- **Analysis**: comparison, critique, architecture review
- **Synthesis**: pattern across multiple sources or conversations
- **Decision rationale**: "we chose X because Y, with Z tradeoff"
- **Research finding**: unexpected connection, contradiction surfaced
- **Plan**: multi-step approach worth preserving

Skip if the output was a simple Q&A, a code diff, or procedural ("I ran X, it said Y").

### 2. Classify destination

One table, one column:

| Content shape | Destination |
|---|---|
| Reference material, external findings, framework analysis | `<vault>/50-References/<slug>.md` |
| Novel idea, thesis, angle not yet acted on | `<vault>/40-Ideas/<slug>.md` |
| Project-specific decision or milestone | `<vault>/10-Projects/<Project>.md` under `## History` |
| Person-specific observation | `<vault>/30-People/<Name>.md` |
| Life-admin insight (legal, tax, health, etc.) | `<vault>/20-Areas/<domain>/<topic>.md` |

Pick one. If genuinely ambiguous, ask the user once with two options, not five.

### 3. Write the file

Use this frontmatter:

```yaml
---
title: <Title>
type: reference | idea | history
source: chat
ingested: YYYY-MM-DD
session: <current cwd basename>
tags: [topic1, topic2]
---
```

Body rules:
- **Aggressive wikilinks.** Every `[[Person]]`, `[[Project]]`, `[[Concept]]`, `[[Tool]]` on first mention. The graph is the whole point.
- **Preserve the synthesis, not the chat.** Don't write "the user asked... and I said..." Strip the conversational framing. The file should read as a standalone note.
- **Cite the session.** Footer: `*Filed from session: <cwd basename>, <YYYY-MM-DD>*`
- **Under 400 lines.** If longer, it's a document not a note. Break it up.

### 4. Update `index.md`

Add a one-line entry to the correct section:
```markdown
- [[<slug>]] - one-line summary (filed: <date>)
```

### 5. Append to `log.md`

```
## [YYYY-MM-DD] promote | <Title>
Filed from chat to <destination>. One-sentence content note.
```

### 6. Confirm

One sentence back to the user: "Filed to `<path>`. Index and log updated." No fanfare.

## What NOT to do

- **Don't duplicate.** Check `qmd_query` (or grep) for the slug first. If a page exists, offer to append instead of overwrite.
- **Don't promote noise.** A single-paragraph observation isn't worth a file. Compress into an existing page or skip.
- **Don't edit `raw/`.** Raw source material is immutable.
- **Don't offer twice.** If the user ignores the offer, move on.
- **Don't write chat transcripts.** The file is the synthesis, not the dialogue.

## Interaction with other skills

- **`ingest`** handles new source material. `file-insight` handles self-generated content. Same write targets, different origin.
- **`wrapup`** promotes session history to project pages. `file-insight` fires mid-session on one specific insight, not the whole session.
- **`lint`** (if you use it) will find orphan pages from this skill if wikilinks don't resolve. Densify links to keep lint happy.
