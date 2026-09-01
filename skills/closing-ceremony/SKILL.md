---
name: closing-ceremony
description: End-of-task ritual that hardens a branch before it ships — thermonuclear code-quality review, apply fixes, simplify pass, targeted thermonuclear re-review of touched files, live end-to-end browser verification for UI changes, push and open the PR in the browser, then ask about ticket status/comment updates. Trigger on "closing ceremony", "run the closing ceremony", or /closing-ceremony. Not auto-triggered by casual mentions of review/simplify/PR — only by explicit invocation.
disable-model-invocation: true
---

# Closing Ceremony

A fixed ritual for finishing a branch: review hard, clean up, review hard again, then ship the PR. Every step operates on the branch's full diff against its base branch, not just the latest commit. Run steps in order — don't skip, reorder, or soften the thermonuclear standard.

## Step 0 — Establish scope

1. `git status` / `git branch --show-current`. Uncommitted changes → ask whether to include them or stash, don't guess.
2. Determine the base branch (ask if ambiguous). Confirm current branch isn't `main`/`master` — stop and tell the user if it is.
3. Diff scope for every later step is `git diff <base>...HEAD`, not just uncommitted changes.

## Step 1 — Thermonuclear review (round 1)

Spawn **one** forked subagent for the whole review/fix arc (this fork also does Step 3 — don't spawn a fresh agent per round, that reloads the standard and re-reads the diff from scratch). Brief it with: the `thermo-nuclear-code-quality-review` skill, the Step 0 diff scope, and instructions to fix structural/high-conviction findings itself in the same pass rather than reporting back for re-dispatch.

- Fix high-conviction findings inline (skill's priority order: structural first, cosmetic last); skip manufacturing busywork from low-value nits.
- Large architectural findings the user should weigh in on → fork surfaces them to you to relay, doesn't unilaterally restructure.

## Step 2 — Simplify

Invoke the `simplify` skill against the same diff scope (including Step 1's fixes). It reviews reuse/simplification/efficiency and applies its own fixes. Note which files it touched — Step 3 needs that list.

## Step 3 — Thermonuclear review (round 2, targeted)

Resume the **same fork** from Step 1 and have it re-review only the files it fixed plus the files Step 2 touched — same regression guarantee as a full re-review since nothing else changed, at a fraction of the tokens.

- Full diff re-review instead, only for small branches where narrowing saves nothing.
- Fix remaining structural findings; if nothing new turns up, say so and move on.

## Step 4 — Sanity check

Run this project's build/typecheck/test commands if discoverable; report only pass/fail, no narration. Say so in one line if none are discoverable.

**Gate:** anything fails → stop the ceremony, show the user, ask whether to fix now and resume or abort. Don't proceed into Steps 5–6 with known-broken code, and don't decide unilaterally.

## Step 5 — Live browser verification (UI/frontend changes only)

If the diff touches UI/frontend code, spawn a **separate** agent (not the review fork), medium effort/model tier, to verify the change works live — type-checking and tests don't verify feature correctness. Identify the impacted feature(s) from the diff first; scope the test to that, not a generic smoke test. Brief it to:

1. Use the `run` skill to launch the app.
2. Drive a real end-to-end flow: entry point a user would use through to actual conclusion (submission, save, navigation, rendered result) — not just load-and-eyeball.
3. Cover obvious edge cases in that flow (empty/invalid input, error/loading states) and adjacent surfaces the diff could plausibly break — don't wander into unrelated app areas.
4. Report pass/fail per flow, with screenshots/console errors only if something breaks.

Skip (say so in one line) if the diff is backend/CLI/library-only with no UI surface.

## Step 6 — Commit

Stage and commit Steps 1–3's fixes (separate, clearly-scoped commits are fine; don't amend the user's existing commits). Follow repo conventions and the user's global git rules (no `Co-Authored-By`/`Claude-Session` trailers).

## Step 7 — Push

`git push` (`-u` if no upstream). Shared-visible action — if a PR is already open or remote history diverged, stop and confirm rather than force-pushing.

## Step 8 — Open the PR in the browser

1. Determine the git host from `git remote get-url origin` (GitHub/GitLab/Bitbucket/other) and construct the new-PR/MR URL for that branch pair.
2. Draft title/description from the real diff and `git log <base>..HEAD --oneline` — never invent commit content. AIT company repo → use the `AIT-pr-generate` template (with AI Usage Declaration); otherwise a plain concise summary + commit list.
3. Use Chrome browser tools (ToolSearch to load if deferred) to navigate and fill in the form.
4. **Stop before submitting** — show the user the filled form/summary and get confirmation before clicking create; opening a PR is team-visible.
5. On confirmation, submit and report the PR URL.

## Step 9 — Ticket updates

After the PR exists (or is confirmedly skipped), check for a tracked ticket (Jira-style key in branch name, or one mentioned this session). None evident → skip in one line, don't guess.

If evident, ask the user what they want:
- Transition ticket status (and to which)?
- Add a comment, optionally with the real testing scenarios from Step 5 (don't fabricate coverage that wasn't tested)?

Only act on what's confirmed; report back what was updated with a ticket link. Decline → move on.

## Step 10 — Worktree cleanup

If the work happened in a dedicated worktree, ask whether to delete it now that the PR is open — don't delete unprompted. Confirm → remove and report. Decline/no worktree → skip in one line.

## Notes

- Heavy, branch-mutating ritual — only run on explicit invocation (`/closing-ceremony` or clear user request), never opportunistically.
- Nothing to fix at a step → say so and move on, don't manufacture findings.
- User interrupts mid-ceremony → stop cleanly at the current step, don't continue through push/PR.
