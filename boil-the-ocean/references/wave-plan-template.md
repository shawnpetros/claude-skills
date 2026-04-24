# Wave Plan Template

Copy to `notes/PLAN.md` at the start of a rewrite. Commit it before W0.

```markdown
# <Project> V<n> Rewrite Plan

## Scope
<1-paragraph goal. What end state are we aiming for?>

## Out of scope
<Explicitly list things not in this rewrite. Future waves can pick them up.>

## Verification gates (applied by every subagent)
1. `<build command>` - clean
2. `<lint command>` - clean with strict flags
3. `<test command>` - full suite passes
4. At least one live/integration test for anything that touches an external system

## Waves

### W0: Scaffold
- [ ] Worktree at `~/projects/<name>-v<n>` on branch `v<n>-<slug>`
- [ ] Directory structure committed
- [ ] Schema / type definitions / stubs
- [ ] Scenario-as-acceptance-test skeletons (N scenarios, all `#[ignore]`)
- [ ] First commit: `v<n>: scaffold`

### W1: Parallel build (independent modules)
Subagents: max 2 concurrent, sonnet
- A1: <module 1>
- A2: <module 2>
(queue A3, A4 for the second sub-wave)

Gate: `<build command>` + `<test command>` clean

### W2: Integration layer
Subagents: max 2 concurrent, sonnet
- A5: CLI / orchestration
- A6: <integration>

### W3: Feature promotion
(Promote any Phase 2/3 features that are load-bearing per Boil the Ocean philosophy)

### W4: Adversarial review
Subagents: 1 opus reviewer
- Scope: all N modules, focus areas, scenario readiness matrix
- Output: `notes/W<n>-review-findings.md` with C/H/M/L/Info categorization

### W5: Fixups
Subagents: max 2 concurrent, sonnet
- A10: wiring fixes (Critical/High findings)
- A11: scenario completion

### W6: Data migration
Subagent: 1 sonnet
- Port from prior prototype with parity tests

### W7+: Polish
Coordinator work: docs, README, operator guide, commit, push

## Concurrency budget
Total planned agents: <N>
Sonnet minutes projected: <N x ~15>
Opus minutes projected: <N x 1>

## Rollback plan
Each wave is one commit. `git reset --hard <prev-wave-commit>` is the unwind.

## Success criteria
- [ ] All gates green
- [ ] All scenarios passing without `#[ignore]`
- [ ] Data migrated with parity tests
- [ ] At least one live integration test per external system
- [ ] `notes/RETROSPECTIVE.md` written
```
