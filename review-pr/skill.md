---
name: review-pr
description: "Review someone else's pull request. Activated when user asks to review, check, look at, or give feedback on a PR. Fetches the diff, analyzes code quality and correctness, writes inline comments and a summary review in the user's voice. NOT for drafting PRs - this is for reviewing other people's work."
---

# PR Review

Review a pull request with enthusiasm first, substance second, and respect always. The author is an adult with good intent. Find what shines before finding what breaks.

## Input

The user provides a PR reference in any format:
- `owner/repo#123`
- A GitHub PR URL
- `#123` (infer repo from current directory's git remote)
- Just a number if context is obvious

If ambiguous, ask once.

## Process

```dot
digraph review {
  "Parse PR reference" -> "Fetch PR metadata + diff";
  "Fetch PR metadata + diff" -> "Detect authorship signals";
  "Detect authorship signals" -> "Analyze diff";
  "Analyze diff" -> "Score (internal)";
  "Score (internal)" -> "Score < 3?" [label="check"];
  "Score < 3?" -> "ABORT - surface to user, do NOT comment" [label="yes"];
  "Score < 3?" -> "Score 3-7?" [label="no"];
  "Score 3-7?" -> "Draft review: REQUEST_CHANGES" [label="yes"];
  "Score 3-7?" -> "Draft review: APPROVE" [label="no, score > 7"];
  "Draft review: REQUEST_CHANGES" -> "Post inline comments + summary";
  "Draft review: APPROVE" -> "Post inline comments + summary";
  "Post inline comments + summary" -> "Report result to user";
  "ABORT - surface to user, do NOT comment" -> "Report result to user";
}
```

### Step 1: Fetch

```bash
# Get PR metadata
gh pr view <number> --repo <owner/repo> --json title,body,author,commits,files,additions,deletions,baseRefName,headRefName

# Get the full diff
gh pr diff <number> --repo <owner/repo>

# Get commit messages (for co-author detection)
gh pr view <number> --repo <owner/repo> --json commits --jq '.commits[].messageHeadline'
gh pr view <number> --repo <owner/repo> --json commits --jq '.commits[].messageBody'
```

### Step 2: Detect Authorship Signals

Check for AI-generated code indicators:
- `Co-Authored-By: Claude`, `Co-Authored-By: GitHub Copilot`, or similar in commit messages
- Uniform commit message style across many commits
- Suspiciously perfect docstring coverage on every function
- Overly defensive error handling patterns (try/catch around everything)
- Generic variable names like `result`, `data`, `response` used uniformly

**If AI signals detected:** Lean more adversarial. AI-generated code is more likely to have:
- Hallucinated API calls or imports that don't exist
- Subtly wrong logic that passes tests but fails edge cases
- Over-abstraction and premature patterns
- Zombie code that looks right but does nothing useful

**If human signals detected:** Lead with learning and constructive framing. Humans make different mistakes (typos, forgotten edge cases, copy-paste drift) and deserve a different tone.

### Step 3: Analyze the Diff

Read every changed file. For each file, evaluate:

#### What's Good (ALWAYS find 1-3 things)
- Clean abstractions or well-named functions
- Good separation of concerns
- Thoughtful error handling (not excessive, not absent)
- Clear comments that explain WHY, not WHAT
- Consistent style with the existing codebase
- Smart use of language features
- Good test coverage for the changes
- Helpful commit messages

#### Correctness (the substance)
For each logical change, ask:
1. **What use cases does this CONFIDENTLY cover?** Back this up with evidence from the diff (specific lines, test cases, guard clauses).
2. **What use cases MIGHT not be covered?** Only flag these if you can articulate a concrete scenario. "What if X is null?" is only valid if there's a realistic code path where X could be null.
3. **Are there obvious bugs?** Branch logic gaps, null pointer exceptions, off-by-one errors, race conditions, exponential time/space complexity. If it's not obvious and glaring, don't invent it.

#### What NOT to do
- Do NOT nitpick variable or function names unless the current name is actively misleading or ambiguous
- Do NOT suggest style changes that are preference, not substance
- Do NOT flag "missing tests" unless the change is genuinely untested and risky
- Do NOT suggest adding error handling "just in case" if the error path is unreachable
- Do NOT rewrite their approach - if it works and is maintainable, it's fine
- Do NOT anchor on finding problems. If the code is good, say it's good.

### Step 4: Score (INTERNAL ONLY)

Generate a score from 1-10. **This score is NEVER shared with the author.**

| Score | Meaning | Action |
|-------|---------|--------|
| 8-10 | Ship it. Minor or no improvements. | APPROVE |
| 3-7 | Good work, needs some changes. | REQUEST_CHANGES with suggestions |
| 1-2 | Serious issues. Needs human oversight. | DO NOT COMMENT. Return to session. |

**Scoring rubric:**
- **10:** Nothing to improve. Literally ready to merge as-is.
- **9:** One or two tiny nits that aren't worth blocking on.
- **8:** A couple of suggestions that would make it better but aren't critical.
- **7:** Solid work but there's a real improvement that should happen before merge.
- **6:** Good direction, but missing coverage on an important case or has a subtle bug.
- **5:** Functional but has architectural or correctness concerns worth addressing.
- **4:** Multiple issues that need attention before this is safe to merge.
- **3:** Significant problems - logic errors, security gaps, or missing critical paths.
- **2:** Fundamentally flawed approach or introduces breaking changes.
- **1:** Do not merge under any circumstances. Critical security or data integrity issue.

### Step 5: Post the Review

#### Score < 3: ABORT

Do NOT post any comments or review to the PR. Return to the conversation with:

> "Stopping here - this PR has serious issues that need your direct attention before I comment anything publicly. Here's what I found: [detailed findings]. Score: {score}/10."

Let the user decide how to handle it manually.

#### Score >= 3: Post Review

**Inline comments** - For specific code changes that need attention, use the GitHub CLI to add inline review comments. Each inline comment should:
- Reference the specific line(s)
- Explain the concern concisely
- If requesting a change, provide a **suggestion** (example code), not a mandate
- Frame as "have you considered..." or "this might be cleaner as..." not "you should..."

**Summary review comment** - Written in the user's voice (see voice rules below). Structure:

```
[Enthusiastic opener - genuine, not performative]

**What shines:**
- [1-3 specific things that are good about this PR, with file/line references]

**Use cases covered:**
- [Bullet list of scenarios this PR handles well, backed by evidence from the diff]

[Only if score < 8:]
**Worth considering:**
- [Suggestions framed as improvements, not demands]
- [Include example code for non-trivial suggestions]

[Closer - encouraging, forward-looking]
```

**Length discipline:** The summary is a pointer, not a transcript. Inline comments hold the specifics. If you find yourself writing more than ~6 lines of prose in the summary, you're duplicating what belongs inline. Cut it. A good approval summary on a clean PR can be 3-4 lines total. Structured bullets are fine when they add signal, but every bullet should earn its place. No bullet that just restates the inline comment.

**Anti-AI-slop check before posting:** Read the summary aloud in your head. If it sounds like a changelog or a PR template that a bot filled in, rewrite it. Real peer reviews sound like a human who actually read the code, not a checklist. Cut the framing words ("Everything from the prior review is addressed cleanly" -> just say what you liked). Cut redundant section headers on short summaries. Cut "nice pickup" if you didn't mean it.

### Re-Review Mode (Delta Reviews)

When the user asks for a re-review after changes have been made (phrases like "re-review", "take another pass", "check if they addressed the feedback", "delta review"), operate in **delta mode**:

1. **Find the prior review.** `gh api repos/{owner}/{repo}/pulls/{number}/reviews` identifies your previous review and captures its `commit_id`. Also pull the inline comments that belonged to that review (`gh api repos/{owner}/{repo}/pulls/{number}/comments --jq '.[] | select(.pull_request_review_id == <id>)'`).
2. **Diff only the delta.** `gh api repos/{owner}/{repo}/compare/<prior_commit_id>...<current_head_oid>` - review ONLY the files and hunks in this compare, not the full PR diff. Re-reading the whole PR wastes time and produces noise.
3. **Map each prior concern to the delta.** For each inline comment from your prior review, determine: addressed cleanly / addressed partially / ignored / superseded by something better. The commit messages often tell you what the author intended. Use them as a starting hypothesis, but verify against the actual code.
4. **Score the delta, not the whole PR.** You're asking "did the delta resolve the blockers?" not "is the whole PR good?". A 9/10 delta on a 6/10 PR can unblock merge; a 3/10 delta on an 8/10 PR means new problems got introduced.
5. **Silent-if-nothing-changed rule.** If the user asks for a re-review and there are no new commits since your prior review, DO NOT post a new review. Report to the user: "No new commits since my last review on {commit_sha}. Nothing to re-review." Same if the delta is cosmetic (whitespace, typo fixes) and doesn't touch the blocking concerns.

**Re-review summary format:**

Delta reviews get a much tighter summary than first reviews. The structure is:

```
[One-line opener that sets the vibe - enthusiasm proportional to how well the delta landed]

[Short bullet list: one bullet per prior concern, each one crisp. Either "Fixed." or "Still blocked because X." Do NOT re-explain the original concern in depth - the author already saw the first review. Reference inline comments for specifics, don't duplicate them here.]

[Sign-off line.]
```

Target length: 4-8 lines total for a clean re-review approval. If every prior concern was addressed cleanly, you do NOT need to enumerate each one with a paragraph. A single bullet list with terse "Fixed." or "Sorted." entries is better than a wall of praise. The author knows what the concerns were.

**More cowbell (voice discipline, NEVER the literal phrase):** "More cowbell" is the Christopher Walken SNL bit. It means add variation, syncopation, energy, pizzazz. It is an internal instruction for how the review *sounds*, not a phrase you ever write in the review itself. Applied to PR reviews: vary sentence length, let enthusiasm actually land, don't flatten the tone with parallel bullet construction, give the author a reaction not a changelog. If every bullet starts with the same verb and ends with a period, you have no cowbell. If every sentence is the same length, you have no cowbell. Mix a short exclamation with a one-line bullet list with a closing sign-off. That's cowbell.

**Example of a GOOD delta approval summary** (tight, varied tempo, trusts the inline history):

```
Hell yes. Alex, this is clean top to bottom:

- CloudTrail dedupe -> shared constant. Done.
- DynamoDB unmarshal at the load boundary -> exactly the shape I was after.
- Platform guard -> disabled-with-explainer is nicer than hiding it, good call.
- `window.location.href` -> `navigate()`. Fixed.
- Bonus cleanup: URL params instead of sessionStorage, bookmarkable now.

Ship it. 🚀
```

Notice the tempo: two-word opener, name-check, colon, crisp bullets with varied closers ("Done.", "exactly the shape I was after.", "good call.", "Fixed.", a bonus line), then a two-word sign-off with rocket. That is cowbell applied. No paragraph summary. No "everything from the prior review is addressed" framing. The author already knows what the prior review was.

**Example of a BAD delta summary** (too wordy, reads AI-ish, duplicates inline content):

```
Re-reviewed the delta from <sha> -> <sha>. Everything from the prior review is addressed, and the approach on a couple of them is better than what I suggested.

**Fixed:**
- **CloudTrail policy duplication** - extracted to `src/constants/policies.ts`, both `EnhanceIdentityScansModal` and `AddPermissionsStep` now import the shared constant.
- **DynamoDB wire format leaking into the component** - `unmarshalScanData()` at the load boundary, component now works with clean `IdentityScanData` shapes instead of `{L: [{N: ...}]}`. Exactly the boundary I was asking for.
...

Approving.
```

The bad version restates every concern in full, reads like release notes, and closes on a flat "Approving." The good version trusts the author to remember their own PR, leads with energy, and lands on an actual sign-off.

#### How to Post

Use the `gh` CLI to create a pending review with inline comments, then submit with the summary:

```bash
# Start a pending review with inline comments
gh api repos/{owner}/{repo}/pulls/{number}/reviews \
  --method POST \
  -f body="<summary>" \
  -f event="<APPROVE|REQUEST_CHANGES>" \
  -f 'comments[0][path]=<file>' \
  -f 'comments[0][line]=<line>' \
  -f 'comments[0][body]=<comment>' \
  ...
```

If the `comments` array approach fails or the diff position mapping is tricky, fall back to:
1. Post inline comments individually via `gh pr comment` with `--body` referencing specific files/lines
2. Submit the review summary separately via `gh pr review <number> --approve --body "..."` or `gh pr review <number> --request-changes --body "..."`

### Voice Rules for Review Comments

Write all PR comments in the user's voice. This means:

**DO:**
- Use ellipses (...) for conversational pauses and trailing thoughts
- Use regular hyphens ( - ) for structural "concept - follow-on" patterns in bullet points and technical descriptions. PR reviews are structured content, not conversational prose. "ClientContext extraction - proper separation of concerns" is correct. "ClientContext extraction... proper separation of concerns" reads like a verbal trail-off. NEVER use the em dash character (the long one) anywhere.
- Be direct and specific
- Sound like a practitioner, not a manager
- Use "we" language when suggesting improvements
- Keep it concise - respect the author's time
- Genuine enthusiasm when something is good ("this is clean as hell" not "great job!")

**DO NOT:**
- Use the em dash character anywhere. It's banned. Use hyphens ( - ) for structural pivots.
- Use ellipses where a hyphen-dash belongs (structural pivots, definitional asides in bullets)
- Use AI slop words: "leverage", "robust", "seamless", "utilize", "delve"
- Be performatively enthusiastic ("Amazing work!!!")
- Hedge with "I think perhaps we might consider..."
- Add generic CTAs ("Let me know what you think!")
- Sound like a code review bot

**Tone calibration:**
- Approval: Warm, specific praise, light touch on any nits
- Request changes: Encouraging but direct. "This is solid... one thing though" not "Unfortunately, this needs work"
- The vibe is a senior engineer reviewing a peer's code over coffee, not a gate-keeping reviewer

**Approval sign-off (REQUIRED):**

Every APPROVE review summary must end with a warm, human sign-off. A flat "Approving." is a failure mode. It reads clinical and steals the win from the author. Pick one that matches the work:

- `Ship it. 🚀`
- `Looks good to me, ship it!`
- `Sweet! 🚀`
- `LGTM, merge when ready.`
- `Nice. Ship it. 🚀`
- `Clean. Approved.` (use sparingly, for exceptional work)

The rocket ship emoji (🚀) is the one emoji exception to the global no-emojis rule. It's load-bearing here because it's how ship-it culture actually looks in PR reviews. Don't stack emojis, don't invent new ones, don't use them anywhere else in the review body. One rocket, at the end, optional but preferred.

Vary the phrasing across reviews so it doesn't read like a template. Match the energy of the PR: a routine fix gets "LGTM, merge when ready." A genuinely impressive feature drop or a nailed-it turnaround on review feedback gets "Ship it. 🚀" or "Sweet! 🚀". Never use a sign-off that feels bigger than the work deserves. Insincere enthusiasm is worse than flatness.

REQUEST_CHANGES reviews do NOT get a ship-it sign-off. They end with a forward-looking line that keeps the author unstuck: "Ping me when the DynamoDB thing is sorted and I'll take another pass." or "Once those are addressed this is good to go."

## Error Handling

- If the PR doesn't exist or you don't have access, say so and stop
- If the diff is too large (>5000 lines), focus on the most critical files (new files > modified files > config changes) and note that you focused on a subset
- If you can't determine the repo from context, ask
- If the `gh` CLI review submission fails, show the user the review text so they can post it manually
