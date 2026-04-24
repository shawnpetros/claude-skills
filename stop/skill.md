---
name: stop
description: "STOP framework: event-driven scenario gate. Enforces BDD scenarios in features.json before implementation (Gate In) and verification after implementation (Gate Out). Optional hooks provide hard enforcement; this skill provides guidance."
---

# STOP: Scenario Gate

STOP is a validation layer on top of spec-driven development. It enforces two gates:

1. **Gate In:** BDD scenarios must exist in `features.json` before implementation
2. **Gate Out:** All scenarios must be verified before claiming completion

Optional hooks in `~/.claude/settings.json` provide event-driven enforcement (see Enforcement section). This skill provides the scenario authoring and verify-loop guidance whether or not hooks are installed.

## Gate In: Scenarios Before Implementation

When transitioning from planning to implementation, write BDD scenarios to `features.json` in the project root.

### Tier Classification

- **Planned:** Went through brainstorming + planning. Extract scenarios from the spec doc.
- **Ad-hoc:** No prior spec. Write 1-3 inline scenarios from the user's request.
- **Trivial:** Single file, under ~20 lines. Write 1 scenario.
- When ambiguous, default to the higher tier.

### Writing Scenarios

Each scenario follows the schema in `references/scenario-schema.md`:

```json
{
  "id": "{phase}-{feature}-S{n}",
  "description": "Plain-English summary",
  "given": "Initial state / precondition",
  "when": "Action or trigger",
  "then": "Expected outcome, specific and verifiable",
  "passes": false
}
```

Write scenarios to the appropriate feature in `features.json`. Create the feature entry if needed. Set feature status to `in_progress`.

### Scenario Quality Check

Before proceeding to implementation, review your scenarios:
- Can each `then` clause be verified by a single command with specific expected output?
- Is there at least one error/edge case scenario (not just happy path)?
- Could a stranger verify this scenario without reading the spec?

## Gate Out: Verify After Implementation

After implementation completes, verify every unpassed scenario.

### The Verify Loop

1. Read `features.json`, collect all scenarios with `passes: false`
2. For each scenario, verify the `then` clause with concrete evidence:
   - **Tests exist:** Run them. Test output is authoritative.
   - **No tests:** Run the app, hit the endpoint, check the output. Code inspection alone is NOT sufficient.
   - **Verification impossible:** Mark as `blocked`, not `failed`.
3. Produce findings:

```json
{
  "passed": ["P1-F1-S1"],
  "failed": [{ "id": "P1-F1-S2", "reason": "...", "patch_scope": "file.ts" }],
  "blocked": []
}
```

4. **All pass:** Flip `passes: true`, set feature status to `complete`
5. **Failures:** Fix scoped to findings only, then re-verify
6. **Blocked:** Surface to user with context

### Loop Caps

- Max 3 Observe-then-Patch cycles per scenario
- Max 5 total cycles per feature
- When hit: surface to user with full findings history

## Enforcement (Optional)

For hard enforcement, install three hooks in `~/.claude/settings.json`:

- **PreToolUse(Skill):** BLOCKS implementation skills if no scenarios exist (hard gate)
- **PostToolUse(Skill):** Injects scenario extraction reminder after planning (soft gate)
- **UserPromptSubmit:** Injects verify reminder if unpassed scenarios exist (soft gate)

Example hook script location: `~/.claude/hooks/stop-gate.mjs`. Without hooks, this skill still works as guidance, you just have to invoke it manually.

## Reference Files

- `references/scenario-schema.md`: BDD scenario JSON schema, ID format, tier examples
- `references/phase-map.md`: Two-gate model overview
