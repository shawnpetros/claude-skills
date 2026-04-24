# Scenario Schema

BDD scenario format used by the STOP framework inside `features.json`.

## Schema Definition

Each scenario is a JSON object with the following fields:

| Field         | Type    | Required | Description                                      |
|---------------|---------|----------|--------------------------------------------------|
| `id`          | string  | yes      | Unique identifier: `{phase}-{feature}-S{n}`      |
| `description` | string  | yes      | Plain-English summary of what this scenario tests |
| `given`       | string  | yes      | Initial state / precondition                      |
| `when`        | string  | yes      | Action or trigger                                 |
| `then`        | string  | yes      | Expected outcome                                  |
| `passes`      | boolean | yes      | Starts `false`. Only Observe flips to `true`.     |

## ID Format

```
{phase}-{feature}-S{n}
```

- **phase:** Phase ID from `features.json` (e.g., `P1`, `P2`)
- **feature:** Feature ID within that phase (e.g., `F1`, `F2`)
- **n:** Sequential scenario number within the feature (e.g., `1`, `2`, `3`)

Example: `P1-F1-S1` is Phase 1, Feature 1, Scenario 1.

## Examples

### Planned (multi-scenario feature)

A well-defined feature with full scenario coverage written upfront during Spec.

```json
{
  "id": "P1-F2",
  "name": "User Authentication",
  "status": "in_progress",
  "scenarios": [
    {
      "id": "P1-F2-S1",
      "description": "Valid credentials grant access",
      "given": "A registered user with correct credentials",
      "when": "They submit the login form",
      "then": "They are redirected to the dashboard with a valid session",
      "passes": false
    },
    {
      "id": "P1-F2-S2",
      "description": "Invalid password is rejected",
      "given": "A registered user with an incorrect password",
      "when": "They submit the login form",
      "then": "They see an error message and no session is created",
      "passes": false
    },
    {
      "id": "P1-F2-S3",
      "description": "Locked account after repeated failures",
      "given": "A user who has failed login 5 times in 10 minutes",
      "when": "They attempt a 6th login",
      "then": "The account is locked and a notification email is sent",
      "passes": false
    }
  ]
}
```

### Ad-hoc (1-3 scenarios)

A feature that emerged mid-session. Spec writes just enough scenarios to validate it.

```json
{
  "id": "P2-F1",
  "name": "CSV Export",
  "status": "in_progress",
  "scenarios": [
    {
      "id": "P2-F1-S1",
      "description": "Export produces valid CSV with headers",
      "given": "A dataset with 3 columns and 10 rows",
      "when": "The user clicks Export",
      "then": "A .csv file downloads with a header row and 10 data rows",
      "passes": false
    },
    {
      "id": "P2-F1-S2",
      "description": "Empty dataset exports headers only",
      "given": "A dataset with 3 columns and 0 rows",
      "when": "The user clicks Export",
      "then": "A .csv file downloads with only the header row",
      "passes": false
    }
  ]
}
```

### Trivial (one-liner)

A quick fix or micro-feature. One scenario is enough.

```json
{
  "id": "P1-F5",
  "name": "Fix timezone display",
  "status": "in_progress",
  "scenarios": [
    {
      "id": "P1-F5-S1",
      "description": "Timestamps render in the user's local timezone",
      "given": "A user in US/Pacific viewing an event created at 2024-01-15T20:00:00Z",
      "when": "The event detail page loads",
      "then": "The time displays as 12:00 PM PST",
      "passes": false
    }
  ]
}
```

## Rules

1. **`passes` starts `false`.** Always. No exceptions. Scenarios are written as failing assertions.
2. **Only Observe flips `passes` to `true`.** The Observe phase runs verification and is the sole authority on pass/fail. Task never marks scenarios as passing.
3. **Feature is complete when all its scenarios pass.** A feature's `status` moves to `complete` only when every scenario in its `scenarios` array has `passes: true`.
4. **Scenarios are append-only during a session.** New scenarios can be added (discovered during Observe/Patch), but existing scenario definitions should not be edited unless the requirement itself changed.
5. **Patch creates new scenarios for regressions.** If Observe finds a failure that wasn't covered, Patch adds a new scenario before fixing, so the fix is verified on the next Observe pass.
