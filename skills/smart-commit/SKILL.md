---
description: Group and commit all pending changes with contextual commit messages
user-invocable: true
---

# Smart Commit

Two agents: **`smart-committer`** (sonnet / medium) reads the diffs and proposes a grouping;
**`commit-runner`** (haiku) just runs the approved plan. The user approves/edits the proposal
between the two, then you offer to push.

**Trigger phrases:** "commit", "commit everything", "commit pending", "commit all", "smart commit"

## Step 1 — Get the proposal

Launch the `smart-committer` agent (Agent tool, `subagent_type: "smart-committer"`). Ask it to
propose a commit grouping for all pending changes **without committing**. It returns a numbered
list of proposed commits (message + files) — or "Nothing to commit" (then stop).

## Step 2 — Show the user, get approval

Relay the proposal verbatim and ask the user to approve or edit it (regroup, reword, drop, or
reorder commits). Wait for their answer. Do NOT commit anything yourself.

## Step 3 — Commit the approved plan

Launch the `commit-runner` agent (Agent tool, `subagent_type: "commit-runner"`). Give it the
final approved plan as an explicit numbered list — each commit's message and its files by path.
It stages each group and commits, then returns the hashes/messages. Relay the report.

## Step 4 — Push

Ask: "Push to remote?" If yes, launch a `commit-runner` agent in **push mode** — tell it to run
`git push` (or `git push -u origin <branch>` if the branch has no upstream). Relay its output.
If no, done.
