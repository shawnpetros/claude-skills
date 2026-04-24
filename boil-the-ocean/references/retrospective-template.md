# Retrospective Template

Write at end of run, before closing the session. Location: `notes/RETROSPECTIVE.md`.

```markdown
# <Project> V<n> Retrospective

Run date: <YYYY-MM-DD>
Coordinator: <model>
Branch: v<n>-<slug>
Final commit: <sha>

## Wave-by-wave metrics

| Wave | Duration (wall) | Agents | Model(s) | Tokens (est) | Tests after | Commit |
|---|---|---|---|---|---|---|
| W0 | ... | 0 | - | ... | N | <sha> |
| W1 | ... | A1,A2 | sonnet | ... | N | <sha> |
| ... | | | | | | |

## Individual agent readout

| Agent | Duration | Tokens | Tests | Outcome | Notes |
|---|---|---|---|---|---|
| A1 | ... | ... | ... | accepted | ... |

## Behavioral observations

(What did the agents do that was surprising, good, bad, or instructive?)

- Coordination bugs
- Self-committing agents
- Helpful-but-wrong cross-scope edits
- Spec divergences discovered only via live tests
- ...

## Failures

For each failure, even if recovered:
- **Symptom:**
- **Root cause:**
- **Recovery path:**
- **Prevention:**

Required: if a 529 cascade happened, document it. If compaction was forced,
document it. If an agent had to be re-dispatched, document it.

## What I'd do the same

(Load-bearing patterns worth keeping.)

## What I'd change

(Ranked by ROI.)

## Skill updates

(What changes to `~/.claude/skills/boil-the-ocean/skill.md` or its references
does this run suggest? Edit the skill directly before closing the session.)
```

## Writing discipline

- Evidence over feelings. Numbers, commits, agent names, exact symptoms.
- If a failure recovered, still document it. The point is the next run.
- Update the skill itself with any new failure mode or prevention tactic. This
  is the feedback loop that turns one rewrite into a methodology.
