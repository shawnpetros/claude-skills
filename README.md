# claude-skills

Opinionated [Claude Code](https://claude.com/claude-code) skills that earn their keep.

Process skills, not prompts. These are the ones I actually ship with... written, iterated, and debugged on real work. No chain-of-thought cosplay, no "you are a helpful assistant," no corporate hedging.

## What's in here

| Skill | What it does |
|-------|--------------|
| [`review-pr`](./review-pr/skill.md) | Reviews a PR with taste. Fetches the diff, finds what shines, flags what breaks, posts inline comments and a summary in your voice. Delta-aware re-reviews so you can ask "did they fix it?" and get a tight answer instead of a wall of text. |
| [`stop`](./stop/skill.md) | Spec-driven development gate. Forces you to write BDD scenarios in `features.json` before implementation (Gate In) and verify every one of them after (Gate Out). Optional hooks provide hard enforcement. Stops "I think it works" from ever being a complete status again. |
| [`explain-project`](./explain-project/skill.md) | Produces a 4-section comprehension artifact (What is this / Why this approach / What would break / What I learned) for any codebase. Reads the repo first, then interviews you only on the gaps. Doubles as a README, a CLAUDE.md, or a build-in-public post. |
| [`boil-the-ocean`](./boil-the-ocean/skill.md) | Multi-wave rewrite harness. For when you're doing a V3 rewrite and intelligence is cheap enough to dispatch parallel subagents. Codifies wave structure, concurrency caps, live-integration discipline, telemetry, and the retrospective. NOT for client work with real blast radius. |
| [`wrapup`](./wrapup/skill.md) | Closes out a work session cleanly. Promotes session history to long-term notes (if you run a vault), rewrites `SESSION-CONTEXT.md`, updates `features.json`, commits. Replaces the "uh, what did I do today" bloat at the top of every CLAUDE.md. |
| [`file-insight`](./file-insight/skill.md) | Promotes substantive chat-generated synthesis back into your notes. The payoff of Karpathy's Query operation: when you and the assistant produce a good analysis mid-session, file it as a page instead of losing it to transcript history. |
| [`ingest`](./ingest/skill.md) | The universal INGEST operation for your knowledge base. Takes YouTube videos, podcasts, PDFs, web articles, or existing markdown, synthesizes them into deep-dive notes, wikilinks them into your graph, updates `index.md`, appends to `log.md`. Based directly on [Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). |

More coming. I'm auditing my private stack and promoting the general-purpose stuff as it's ready.

## Install

Claude Code skills live in `~/.claude/skills/`. Two options:

**Symlink (recommended)** so you can `git pull` updates:

```bash
git clone https://github.com/shawnpetros/claude-skills.git ~/projects/claude-skills
ln -s ~/projects/claude-skills/review-pr ~/.claude/skills/review-pr
# repeat for any other skills you want
```

**Or install everything at once:**

```bash
git clone https://github.com/shawnpetros/claude-skills.git ~/projects/claude-skills
for d in ~/projects/claude-skills/*/; do
  name=$(basename "$d")
  [ -e ~/.claude/skills/"$name" ] || ln -s "$d" ~/.claude/skills/"$name"
done
```

**Copy** if you want to fork the behavior instead of following upstream:

```bash
cp -r ~/projects/claude-skills/review-pr ~/.claude/skills/review-pr
```

Restart Claude Code. Invoke with `/review-pr <PR URL>`, `/stop`, `/wrapup`, etc.

## Using these skills with your own notes

Several skills (`ingest`, `file-insight`, `wrapup`) assume you run a personal knowledge base. They implement Andrej Karpathy's [LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): the LLM is the programmer, markdown is the codebase, your vault is the IDE.

If you want the full experience:

**1. Pick a vault location.** Anywhere on disk. `~/vault/`, `~/Documents/notes/`, an existing Obsidian vault, whatever. Obsidian-compatible is ideal but not required, it's just markdown files in a directory.

**2. Seed the structure.** The skills expect a PARA-ish layout:

```bash
mkdir -p ~/vault/{00-Inbox,10-Projects,20-Areas,30-People,40-Ideas,50-References,raw}
touch ~/vault/index.md ~/vault/log.md
```

What lives where:
- `00-Inbox/` ... unsorted captures, triage later
- `10-Projects/` ... active work with a history log
- `20-Areas/` ... life admin that never "ends" (health, legal, taxes)
- `30-People/` ... one page per person, wikilinked from everywhere they're mentioned
- `40-Ideas/` ... theses and angles that haven't shipped yet
- `50-References/` ... distilled external material, deep-dives from `ingest`
- `raw/` ... immutable source material (transcripts, downloaded articles, PDFs). The LLM reads from here but never writes to it.
- `index.md` ... LLM-maintained catalog of every page with one-line summaries
- `log.md` ... append-only chronological record of vault operations (ingest, file, wrapup entries)

**3. Point Claude at it.** Add a line to your `~/.claude/CLAUDE.md`:

```
My personal knowledge base lives at `~/vault/`.
```

The skills in here use `<vault>` as a placeholder in path examples. Claude Code reads your CLAUDE.md on session start and substitutes the real path when it writes.

**4. Read Karpathy's original writeup** if you want to understand the thesis:
[gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

If you don't want a vault, the non-vault skills (`review-pr`, `stop`, `explain-project`, `boil-the-ocean`) still work in isolation. `wrapup` degrades gracefully to just updating `SESSION-CONTEXT.md` and `features.json`.

## The rules these skills follow

Every skill in here enforces the same anti-slop discipline:

- **No AI-slop vocabulary.** "Leverage", "robust", "seamless", "utilize", "delve" are banned.
- **No em dashes.** The character ( — ) is contraband. Hyphens ( - ) for structural pivots in bullets. Ellipses (...) for conversational pauses. Em dashes scream LLM.
- **Opinions, not hedges.** "This breaks" beats "I think perhaps we might consider that this could potentially break."
- **Brevity by default.** If the answer fits in one sentence, one sentence is what you get. Length is earned, not assumed.
- **Genuine reactions, not performative ones.** "This is clean as hell" > "Amazing work!!!"

These are baked into every skill body. It's why the output doesn't read like every other LLM wrapper.

## How these are written

Each skill is a single `skill.md` with YAML frontmatter (`name`, `description`) plus a body that reads like instructions to a senior teammate who already knows what they're doing. Rigid skills like `review-pr` have a step-by-step flow with examples of good and bad output. Flexible skills ship with principles instead.

The frontmatter `description` is load-bearing. Claude Code decides whether to invoke a skill based on it, so it's written like a trigger specification: activation phrases, what the skill does, what it doesn't do.

Some skills (`stop`, `boil-the-ocean`) have a `references/` directory with templates and schemas. The skill body tells Claude when to read them. That keeps the top-level skill concise while letting heavy reference material stay out of the way until needed.

## Why public?

I run a private skills stack for my own business work: outreach, lead intel, content pipeline, brand voice, daily briefings. That stays private because it's stitched into trackers, credentials, and personal context that doesn't travel.

But process skills (review a PR, wrap up a session, explain a project, plan a multi-wave build) are useful to anyone running Claude Code seriously. Those get promoted here as I audit them.

Garry Tan shipped [gstack](https://github.com/garrytan/gstack) publicly. Same energy. Steal what works.

## License

MIT. Use them, fork them, adapt them, rename them. No warranty, no attribution required, though a star is appreciated.

---

Built by [Shawn Petros](https://petrosindustries.com) with Argyle at the keyboard.
