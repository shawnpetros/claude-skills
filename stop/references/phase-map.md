# Phase Map

Two-gate model for the STOP framework.

## Gate Overview

| Gate | When | Enforcement | What Happens |
|------|------|-------------|-------------|
| **Gate In** | After planning, before implementation | Hard (PreToolUse blocks Skill) + Soft (PostToolUse injects, UserPromptSubmit injects) | BDD scenarios written to features.json with passes: false |
| **Gate Out** | After implementation, before completion | Soft (UserPromptSubmit injects verify reminder) | Each scenario verified with evidence, passes flipped to true |

## Flow

```
brainstorming -> writing-plans -> [Gate In: scenarios] -> implementation -> [Gate Out: verify] -> done
                                      |                                       |
                                 Hook blocks if                          Hook reminds if
                                 no scenarios                          unpassed scenarios
```

## Tier Behavior

| Tier | Gate In | Gate Out |
|------|---------|---------|
| **Planned** | Extract scenarios from spec doc (full coverage) | Full verify loop with findings |
| **Ad-hoc** | Write 1-3 inline scenarios from user intent | Verify loop |
| **Trivial** | Write 1 one-liner scenario | Single verify pass |
