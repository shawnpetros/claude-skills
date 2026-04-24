---
name: boil-the-ocean
description: Use when tackling a multi-day rewrite, ambitious end-to-end build, or any project that benefits from parallel subagent orchestration with adversarial review. Activates on "boil the ocean", "V3 rewrite", "let's rebuild X", "parallel agents", "multi-wave", or when the user explicitly authorizes large-scope autonomous work. Codifies wave structure, concurrency caps, live-integration discipline, telemetry, and retrospective. NOT for client work or production systems with real blast radius, use a surgical-precision pattern there.
---

# Boil the Ocean: Multi-Wave Rewrite Harness

The operational how-to for ambitious end-to-end builds with parallel subagents. Philosophy: intelligence is cheap, the constraint is no longer scope. Use it. Don't split load-bearing work into arbitrary phases that rarely ship. Ship the whole thing in one concentrated run.

This skill codifies the discipline that keeps a multi-wave run from setting itself on fire.

## When to activate

- User says "boil the ocean", "let's rewrite X", "V<n> rewrite", "rebuild this from scratch"
- Scope spans 2+ days of work AND 4+ independent modules
- User explicitly authorizes autonomous multi-hour work ("let it cook", "dispatch agents", "make it so")
- A plan requires more than 6 subagent dispatches

## When NOT to activate

- Client work or production systems (surgical precision applies instead)
- Single-crate or single-file changes
- Bug fixes with known root cause
- Anything where the user wants to stay in the loop turn-by-turn

## Non-negotiable preconditions

Before dispatching any subagent:

- [ ] **Worktree isolated.** `git worktree add ~/projects/<name>-v<n> <branch>`. Never do this on the main branch in the primary workspace.
- [ ] **Plan committed to disk.** `notes/PLAN.md` with numbered waves. Plans in chat don't survive compaction.
- [ ] **Verification gates defined up front.** Compile + lint + test commands, written into the plan. No subagent completes without hitting them.
- [ ] **Telemetry file created.** `notes/telemetry.jsonl` per `references/telemetry-schema.md`. Appended on dispatch, completion, commit, failure.
- [ ] **Commit cadence.** One commit per wave. Never bundle waves.
- [ ] **Rollback plan.** Each wave must be revertable via `git reset --hard <prev-wave-commit>` without leaving stranded work.

## Wave structure

A standard run has 6-8 waves. Do not invent more unless scope demands it.

| Wave | Purpose | Subagents | Model |
|---|---|---|---|
| W0 | Scaffold | 0 (coordinator) | sonnet/opus |
| W1 | Parallel build of independent modules | 2 (cap) | sonnet |
| W2 | Integration layer | 2 (cap) | sonnet |
| W3 | Feature promotion from later phases | 1-2 | sonnet |
| W4 | **Adversarial review** | 1 | **opus** |
| W5 | Fixups from W4 findings | 2 (cap) | sonnet |
| W6 | Data migration / cutover | 1 | sonnet |
| W7+ | Polish, docs, handoff | coordinator | sonnet |

Each wave has exactly one commit. Each agent within a wave writes exactly one deliverable note at `notes/A<N>-<scope>-complete.md`.

## Concurrency rules, NON-NEGOTIABLE

These exist because uncapped parallelism triggers rate-limit cascades (529s from Anthropic's capacity shedding). The rules are from real incident experience; the retrospective template captures the shape of the postmortem.

1. **Maximum 2 concurrent sonnet subagents.** Ever. Even if a wave has 4 independent modules, split into two back-to-back pairs.
2. **Maximum 1 opus call per wave.** Opus is for review only. Do not dispatch opus in parallel with anything.
3. **Quiet window after opus.** Wait 5+ minutes (or a user message) between opus completion and the next subagent dispatch. Anthropic capacity shedding compounds; dense bursts trigger 529.
4. **Context externalization between waves (the real "compaction").** Actual `/compact` is a user slash command, the model cannot fire it autonomously. What the coordinator CAN do: after each wave commits, write a 1-page `notes/wave-<N>-summary.md`, and then on the next wave, re-read ONLY that summary, not every per-agent completion note. See "Compaction options" below for the honest tradeoffs.
5. **No subagent ever retries itself in a tight loop.** If it 529s, return to the coordinator. The coordinator decides whether to retry, change approach, or ask the user.

## Subagent briefing

Use `references/subagent-brief-template.md`. Non-negotiable fields:

- **Scope:** exactly one crate/module/concern
- **Gates:** the exact commands that must pass before it reports complete
- **Deliverable note path:** `notes/A<N>-<scope>-complete.md`
- **Do-not-touch list:** files or crates outside its scope
- **Commit policy:** "do not commit; coordinator handles"

Brief like a smart colleague who walked into the room. Context, constraints, then the ask.

## Live integration discipline

Mocks are necessary but not sufficient. Every subagent whose scope touches an external system MUST include at least one live-gated test:

- Scheduling API: create a real post +30 days out, verify it appears, delete it
- Playwright/CDP: load a real page, stage actions, do NOT submit
- Messaging (Telegram/Slack/etc.): send a message to a test chat ID
- Analytics: read from a real logged-in session via CDP

Gate these behind a `live-*` feature flag (cargo feature, pytest marker, whatever your stack uses) so the default test run doesn't produce side effects. The subagent's deliverable note must include the live-test output.

Rationale from real experience: an external-API adapter passed all mock-only tests while pointing at the wrong endpoint (`api.example.com` vs `api.example.io`). Live tests caught three spec divergences that mocks could not.

## Compaction options (honest tradeoffs)

`/compact` is a USER slash command. The coordinator model cannot invoke it
autonomously. This means "proactive compaction" is never fully autonomous,
there is always a human-in-the-loop decision OR a structural workaround.
Pick one:

### Option A: Human checkpoint between waves (recommended for first runs)
Coordinator commits the wave, writes the summary, says: "W<N> landed. Context
is at ~X%. If you want a fresh window for the next wave, type `/compact` now."
User fires `/compact`, gets a clean slate, types "go W<N+1>." This is the
cheapest and most reliable path. It adds one round-trip per wave.

### Option B: Wave-as-subagent (recommended for autonomous runs)
Dispatch each wave's coordination AS ITS OWN subagent. The main session says
"run W5" in a single Agent call; the subagent dispatches A10+A11, gates them,
writes the summary, returns a 200-word result. The main session's context
barely grows because it only sees the subagent's final result, not every
intermediate turn. This is how autonomous multi-wave runs actually stay under
the context budget. The tradeoff: debugging is harder because the main session
didn't see the intermediate state. Mitigate by making the wave-subagent write
everything important to disk.

### Option C: Auto-compact threshold (reactive, not proactive)
Claude Code auto-compacts when approaching context limits. This is reactive:
it fires when the ceiling is hit, not at convenient boundaries. Relying on
this causes cascades because the coordinator is mid-task when auto-compact
becomes necessary AND capacity shedding hits simultaneously.

### Option D: Fresh session per wave
End the session after each wave. Write "start next wave" instructions to
`notes/NEXT-WAVE.md`. User opens a fresh Claude Code session, pastes the
instruction, off they go. Heaviest human involvement but zero context bloat.

### Default recommendation
If the user is around and okay with per-wave check-ins: Option A.
If the user wants to let it cook all day: Option B (wave-as-subagent).
Never rely on Option C alone.

## Inter-wave ritual

After each wave commits:

1. Append wave summary to `notes/wave-<N>-summary.md` (~20 lines: what landed, gate results, token spend, any surprises)
2. Append to `notes/telemetry.jsonl`: event `wave_complete` with commit hash, total tests, cumulative tokens
3. Update `notes/PLAN.md`: cross off the wave, add any discovered work for future waves
4. `/compact` if context is >50% full, otherwise defer to next wave's start
5. Tell the user what landed in one paragraph; ask for go-ahead on next wave unless they pre-authorized

## Retrospective

At end of run, before declaring done, produce `notes/RETROSPECTIVE.md` per `references/retrospective-template.md`. This is the load-bearing artifact. It turns one rewrite into a replayable methodology.

Include:
- Wave-by-wave timing, tokens, commit sizes
- Per-agent metrics (duration, tokens, tests added, outcome)
- Behavioral observations (coordination bugs, self-committing agents, helpful-but-wrong fixes)
- Root-cause any failures, even if they were recovered from
- Prevention recommendations for the next run, ranked by ROI

## Known failure modes

Distilled from real multi-wave runs:

| Symptom | Root cause | Prevention |
|---|---|---|
| 529 cascade requiring manual compaction | 4-wide parallel sonnet + back-to-back opus + coordinator context bloat | 2-wide cap, 5-min opus quiet window, pre-wave compaction |
| One agent's "helpful" sibling-crate fixes broke another agent's in-flight work | No explicit do-not-touch list in the brief | Every brief includes an explicit out-of-scope list |
| An agent self-committed the wave | "Incremental commits" phrased too loosely in brief | Brief says "do not commit; coordinator handles" |
| Mock-only tests passed against wrong API endpoint | No live test gate | Every adapter ships a live test |
| Coordinator context ballooned over 6 waves | Carrying every agent completion note forward | 1-page wave summaries; compact between waves |

## Reference files

- `references/wave-plan-template.md`: The PLAN.md template to commit at W0
- `references/subagent-brief-template.md`: The shape every subagent dispatch should use
- `references/telemetry-schema.md`: JSONL event schema for `notes/telemetry.jsonl`
- `references/retrospective-template.md`: The end-of-run postmortem shape
