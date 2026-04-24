# Telemetry Schema

One JSONL file per run: `notes/telemetry.jsonl`. Append-only. One event per line.

## Events

```jsonc
// session start
{"ts":"ISO-8601","event":"session_start","actor":"<model>-coordinator","note":"..."}

// worktree created
{"ts":"...","event":"worktree_created","path":"<your-projects-path>/<name>-v<n>","branch":"v<n>-<slug>","base":"main@<sha>"}

// scaffold complete
{"ts":"...","event":"scaffold_complete","crates":["..."],"scenarios":N,"migrations":N}

// wave dispatch
{"ts":"...","event":"wave_dispatch","wave":"W<n>","agents":[{"name":"A<N>","model":"sonnet|opus","scope":"..."}]}

// agent poll mid-flight (optional, for stuck-detection)
{"ts":"...","event":"agent_status_poll","agent":"A<N>","status":"running|stuck","notes":"..."}

// agent complete
{"ts":"...","event":"agent_complete","agent":"A<N>","model":"...","duration_s":N,"tokens_estimated":N,"outcome":"accepted|accepted_with_note|rejected","deliverables":{"tests_pass":N,"tests_fail":N,"clippy":"clean|warnings"},"note":"..."}

// wave complete (commit boundary)
{"ts":"...","event":"wave_complete","commit":"<sha>","tests_pass_total":N,"duration_min":N,"agents":["A<N>","A<M>"]}

// rate-limit encountered
{"ts":"...","event":"rate_limit","kind":"529|429|other","during":"<coordinator|agent:A<N>>","duration_s":N,"recovered":true|false}

// compaction event (natural or forced)
{"ts":"...","event":"compaction","trigger":"auto|user|wave_boundary","context_before_tokens":N,"context_after_tokens":N}
```

## Why

- Replayable retrospective. The JSONL is the ground truth when the chat transcript is gone.
- Cumulative token and duration math across waves.
- Rate-limit visibility so the next retrospective has evidence, not just inference.

## Writing discipline

- Append, never rewrite. A corrupted line is recoverable; a rewritten file is not.
- Absolute timestamps in ISO-8601 UTC.
- Keep `note` fields under 200 chars. Detail goes in the per-agent deliverable note.
- Ship a line even on failure. "Agent rejected at gate 2" is a real event.
