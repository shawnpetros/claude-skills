# claude-skills

Opinionated [Claude Code](https://claude.com/claude-code) skills that earn their keep.

Process skills, not prompts. These are the ones I actually ship with... written, iterated, and debugged on real work. No chain-of-thought cosplay, no "you are a helpful assistant," no corporate hedging.

## What's in here

| Skill | What it does |
|-------|--------------|
| [`review-pr`](./review-pr/skill.md) | Reviews a PR with taste. Fetches the diff, finds what shines, flags what breaks, posts inline comments and a summary in your voice. Delta-aware re-reviews so you can ask "did they fix it?" and get a tight answer instead of a wall of text. |

More coming. I'm auditing my private stack and promoting the general-purpose stuff as it's ready.

## Install

Claude Code skills live in `~/.claude/skills/`. Two options:

**Symlink (recommended)** so you can `git pull` updates:

```bash
git clone https://github.com/shawnpetros/claude-skills.git ~/projects/claude-skills
ln -s ~/projects/claude-skills/review-pr ~/.claude/skills/review-pr
```

**Copy** if you want to fork the behavior:

```bash
cp -r ~/projects/claude-skills/review-pr ~/.claude/skills/review-pr
```

Restart Claude Code. Invoke with `/review-pr <PR URL>`.

## The rules these skills follow

Every skill in here enforces the same anti-slop discipline:

- **No AI-slop vocabulary.** "Leverage", "robust", "seamless", "utilize", "delve" are banned.
- **No em dashes.** The character ( — ) is contraband. Hyphens ( - ) for structural pivots in bullets. Ellipses (...) for conversational pauses. Em dashes scream LLM.
- **Opinions, not hedges.** "This breaks" beats "I think perhaps we might consider that this could potentially break."
- **Brevity by default.** If the answer fits in one sentence, one sentence is what you get. Length is earned, not assumed.
- **Genuine reactions, not performative ones.** "This is clean as hell" > "Amazing work!!!"

These are baked into every skill body. It's why the output doesn't read like every other LLM wrapper.

## How these are written

Each skill is a single `skill.md` with YAML frontmatter (`name`, `description`) plus a body that reads like instructions to a senior teammate who already knows what they're doing. Rigid skills like `review-pr` have a step-by-step flow with examples of good and bad output. Flexible skills will ship with principles instead.

The frontmatter `description` is load-bearing. Claude Code decides whether to invoke a skill based on it, so it's written like a trigger specification: activation phrases, what the skill does, what it doesn't do.

## Why public?

I run a private skills stack for my own business work: outreach, lead intel, content pipeline, brand voice, daily briefings. That stays private because it's stitched into trackers, credentials, and personal context that doesn't travel.

But process skills (review a PR, wrap up a session, explain a project, plan a multi-wave build) are useful to anyone running Claude Code seriously. Those get promoted here as I audit them.

Garry Tan shipped [gstack](https://github.com/garrytan/gstack) publicly. Same energy. Steal what works.

## License

MIT. Use them, fork them, adapt them, rename them. No warranty, no attribution required, though a star is appreciated.

---

Built by [Shawn Petros](https://petrosindustries.com) with Argyle at the keyboard.
