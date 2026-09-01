---
name: closing-ceremony
description: End-of-task ritual that hardens a branch before it ships — thermonuclear code-quality review, apply fixes, simplify pass, targeted thermonuclear re-review of touched files, live end-to-end browser verification for UI changes, push and open the PR in the browser, then ask about ticket status/comment updates. Trigger on "closing ceremony", "run the closing ceremony", or /closing-ceremony. Not auto-triggered by casual mentions of review/simplify/PR — only by explicit invocation.
disable-model-invocation: true
---

# Closing Ceremony

A fixed ritual for finishing a branch: review hard, clean up, review hard again, then ship the PR. Every step operates on the branch's full diff against its base branch, not just the latest commit.

Run the steps below **in order**. Do not skip a step or reorder it, and do not silently soften the thermonuclear standard.

## Step 0 — Establish scope

1. `git status` and `git branch --show-current`. If there are uncommitted changes, ask the user whether to include them in the ceremony or stash them — don't guess.
2. Determine the base branch (what this branch was cut from — usually `main`/`master`/`develop`). If ambiguous, ask.
3. Confirm the current branch isn't `main`/`master` itself. If it is, stop and tell the user — the ceremony operates on a feature branch, not the trunk.
4. Note the diff scope as `git diff <base>...HEAD` — this is what every review/simplify step below reviews, not just uncommitted changes.

## Step 1 — Thermonuclear review (round 1)

Spawn **one** forked subagent for the whole review/fix arc (Steps 1 and 3 both happen in this same fork — do not spawn a fresh agent per round, it would reload the thermo-nuclear standard and re-read the whole diff from scratch each time). Brief the fork with: the `thermo-nuclear-code-quality-review` skill name to invoke, the Step 0 diff scope, and instructions to fix structural/high-conviction findings itself in the same pass rather than reporting findings back for the orchestrator to re-dispatch — collect-then-fix as one motion, not two round-trips.

- Findings ranked by the skill's stated priority order (structural regressions first, cosmetic nits last); fix the high-conviction ones inline, skip manufacturing busywork out of low-value nits.
- If a finding implies a large architectural change the user should weigh in on, the fork should surface that to you to relay rather than unilaterally restructuring.

## Step 2 — Simplify

Invoke the `simplify` skill (`Skill({ skill: "simplify" })`) against the same diff scope (branch vs. base, including the Step 1 fixes). It reviews for reuse/simplification/efficiency and applies its own fixes — let it do that. Note exactly which files it touched; Step 3 needs that list.

## Step 3 — Thermonuclear review (round 2, targeted)

Resume the **same fork** from Step 1 (it already holds the thermo-nuclear standard and round-1 context in cache) and ask it to re-review only the files that could have regressed: the files it fixed in Step 1, plus the files Step 2's simplify pass touched. This preserves the same regression guarantee as a full re-review — nothing outside those files changed since round 1 cleared it — at a fraction of the tokens.

- Fall back to a full diff re-review only for small branches (a handful of files) where the narrowing wouldn't save anything meaningful.
- Fix any remaining structural findings.
- If this round turns up nothing new, say so plainly and move on — don't invent findings to justify the step.

## Step 4 — Sanity check

Run this project's build/typecheck/test commands if they're discoverable (package.json scripts, Makefile, etc.) and report only the pass/fail result — no narration of what you looked for. If none are discoverable, say so in one line rather than skipping silently.

**Gate:** if anything fails here, stop the ceremony. Do not proceed into Step 5's live testing or Step 6's commit with known-broken code. Show the user the failure and ask whether to fix it now and resume, or abort the ceremony — don't unilaterally decide either way.

## Step 5 — Live browser verification (UI/frontend changes only)

If the diff touches UI/frontend code (components, pages, styles, client-side routes/handlers), spawn a **separate** agent — not the review fork — at medium effort/model tier to verify the change actually works live, per this project's rule that type-checking and tests verify correctness but not feature correctness. First identify the impacted feature(s)/area(s) from the diff itself (which pages, components, or user-facing flows the changed files belong to) — the test must be scoped to that, not a generic smoke test of the app. Brief the agent to:

1. Use the `run` skill to launch the app.
2. Run a real **end-to-end test of the impacted feature**: start from the entry point a user would actually use (nav link, button, URL) and drive the full flow through to its actual conclusion (submission, save, navigation, rendered result) — not just load the page and eyeball it.
3. Cover the obvious edge cases within that flow (empty/invalid input, error states, loading states) and check the immediately adjacent surfaces the diff could plausibly have broken (e.g. a shared component's other call sites) — don't wander into unrelated parts of the app.
4. Report back pass/fail per flow tested and anything broken — screenshots/console errors if it finds something, not a blow-by-blow of what it clicked.

Skip this step entirely (say so in one line) if the diff is backend/CLI/library-only with no UI surface — don't spin up a browser agent for a change nobody can click on.

## Step 6 — Commit

Stage and commit any fixes made in Steps 1–3 (separate, clearly-scoped commits are fine; don't amend the user's existing commits). Follow this repo's commit message conventions and the user's global git message rules (no `Co-Authored-By`/`Claude-Session` trailers).

## Step 7 — Push

`git push` the branch (set upstream with `-u` if it has none). This is a shared-visible action — if the branch already has a PR open or diverged remote history, stop and confirm with the user rather than force-pushing.

## Step 8 — Open the PR in the browser

1. Determine the git host from the remote URL (`git remote get-url origin`) — GitHub, GitLab, Bitbucket, or other. Construct the new-PR/new-MR URL for that host and branch pair (e.g. GitHub: `https://github.com/<owner>/<repo>/compare/<base>...<branch>?expand=1`; GitLab: `.../-/merge_requests/new?merge_request%5Bsource_branch%5D=<branch>&merge_request%5Btarget_branch%5D=<base>`; Bitbucket: `.../pull-requests/new?source=<branch>&dest=<base>`).
2. Draft the PR title and description from the real diff and `git log <base>..HEAD --oneline` — never invent commit content. If this repo is an AIT company repo (or the user says it is), use the `AIT-pr-generate` skill's exact template for the description, including the AI Usage Declaration section. Otherwise use a plain, concise summary + commit list.
3. Use the Chrome browser tools (load them via ToolSearch first if deferred) to navigate to the constructed URL, fill in the title and description fields.
4. **Stop before submitting.** Show the user the filled-in form (or a summary of what was filled) and ask them to confirm before you click the create/submit button — opening a PR is visible to the whole team, so this is a real confirmation, not a formality.
5. On confirmation, click submit and report back the PR URL.

## Step 9 — Ticket updates

After the PR is created (or confirmed skipped), check whether this branch is tied to a tracked ticket — a Jira-style key in the branch name (e.g. `feature/TAL-494-...`) or one the user has mentioned in this session. If none is evident, skip this step in one line rather than guessing a ticket.

If a ticket is evident, ask the user (don't assume) what they want done, covering:

- Whether to transition the ticket's status (e.g. to "In Review" / "Ready for QA"), and to which status.
- Whether to add a comment, and if so whether it should include the testing scenarios actually exercised in Step 5 (the end-to-end flows and edge cases covered) — pull the real scenarios from that step's results, don't fabricate coverage that wasn't tested.

Only act on what the user confirms — apply the status transition and/or post the comment via the available issue tracker (Jira MCP tools, etc.) and report back what was updated with a link to the ticket. If the user declines, move on without pushing further.

## Step 10 — Worktree cleanup

If this branch's work happened in a dedicated worktree (per this project's worktree convention), ask the user whether to delete it now that the PR is open — don't delete it unprompted. If they confirm, remove the worktree (e.g. `git worktree remove`) and report back. If they decline or the branch wasn't in a worktree, skip this step in one line.

## Notes

- This is a heavy, multi-step, branch-mutating ritual — only run it on explicit invocation (`/closing-ceremony` or the user clearly asking for "the closing ceremony"), never opportunistically.
- If any step finds nothing to fix, say so and move to the next step — don't pad the ceremony with manufactured findings.
- If the user interrupts mid-ceremony, stop cleanly at the current step rather than continuing through push/PR steps.
