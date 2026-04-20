---
name: pr-comments
description: Check the current PR for new Greptile (or human) review comments since the last push. If there are new ones, fix them, commit, push. If not, count consecutive empty fires — after 2, tell the user the PR looks ready to ship. Runs on the 8-minute cron scheduled by pr-commit, or invoke directly. Use when the user says "check the PR", "resolve comments", "what did the reviewer say", "fix the review", "address feedback", "are there new comments", or any variation. For opening a PR from scratch, use pr-commit.
---

# PR Comments

Check the current PR for new review comments since the last push. Fix them if any. Keep a small counter so that after 2 consecutive empty fires on the same commit, we tell the user Greptile has gone quiet and prompt for merge.

Scheduled by `pr-commit` to run every 8 minutes. Runs until merge, cancel, or the quiet-threshold hits.

## State file

`.claude/.pr-comments-state.json` — gitignored. Tracks the consecutive-empty counter across cron fires.

```json
{ "pr_number": 2, "empty_count": 0, "last_push_sha": "abc1234" }
```

Rules:
- **New push since last fire** (`last_push_sha` doesn't match `git rev-parse HEAD`) → reset `empty_count` to 0.
- **Fire found new comments** → fix them, push, reset `empty_count` to 0.
- **Fire found no new comments** → increment `empty_count`.
- **`empty_count == 2`** → tell the user the PR looks ready to ship, cancel the cron, delete the state file.

## Workflow

### 1. Identify the PR

```bash
git branch --show-current
gh pr view --repo redpotatoe07/sickrabbit-theme --json number,state,url
```

- On `main`? Wrong branch. Tell the user and stop.
- PR `MERGED` or `CLOSED`? Cancel the cron, delete the state file, stop.
- No PR for this branch? Stop.

### 2. Load state and reconcile with current HEAD

```bash
head_sha=$(git rev-parse --short HEAD)
```

Read `.claude/.pr-comments-state.json` if it exists. If `last_push_sha` != `head_sha`, the user has pushed since the last fire → reset `empty_count` to 0 and update `last_push_sha` to `head_sha`. (No state file yet? Same reset — treat as `empty_count: 0`.)

### 3. Fetch comments posted since the last push

```bash
last_push=$(git log --format='%cI' -1)
gh api repos/redpotatoe07/sickrabbit-theme/pulls/<N>/comments --paginate \
  --jq ".[] | select(.created_at > \"$last_push\") | {id, path, line, body, diff_hunk, user: .user.login}"
gh api repos/redpotatoe07/sickrabbit-theme/issues/<N>/comments \
  --jq ".[] | select(.created_at > \"$last_push\") | {id, body_preview: (.body | .[0:500]), user: .user.login}"
```

**If both are empty → go to step 6 (empty-fire path).**

**If there's at least one new comment → go to step 4.**

### 4. Fix the new comments

For each:

- Read the file first
- If it includes a ` ```suggestion ` block that still applies, use the suggestion
- Otherwise read the surrounding code and implement the minimal fix
- **Push back** (don't silently apply) when a comment contradicts a logged decision in `docs/decisions.md`, would break merchant admin editability, or asks for a hardcoded value the token system is supposed to prevent. Reply on the PR thread with the reason and move on.

Batch fixes per file. No priority-ordering ceremony — just resolve what applies.

### 5. Commit and push, reset counter

```bash
git add <only the files you touched>
git commit -m "$(cat <<'EOF'
fix: address PR review comments

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
git push
```

Update `.claude/.pr-comments-state.json` with `empty_count: 0` and `last_push_sha: <new HEAD sha>`. Report what was fixed. Exit.

### 6. Empty-fire path: increment counter

Increment `empty_count` in the state file.

**If `empty_count < 2`**: write state, report `no new comments (N/2 empty)` in one line, exit. Cron fires again in 8 min.

**If `empty_count == 2`**: Greptile has been quiet for two rounds on the same commit. This is the ship signal.

1. Cancel the cron (`CronDelete` with the active job ID — get it via `CronList`).
2. Delete `.claude/.pr-comments-state.json`.
3. Tell the user:

```
Greptile has been quiet for 2 consecutive checks on PR #<N>.
Looks ready to ship.

Reminder: merging deploys to the live Shopify theme immediately.

Want me to merge? I can run:
  gh pr merge <N> --repo redpotatoe07/sickrabbit-theme --squash --delete-branch

Or say "wait" if you want another round — I'll reschedule the cron.
```

4. Stop. Do not merge until the user says so.

## Edge cases

- **PR merged or closed** — cancel the cron, delete the state file, stop.
- **On `main`** — wrong branch, stop.
- **Merge conflicts on push** — report the error; the user decides how to recover.
- **Comment on a file the user changed significantly since the review** — suggestion may be stale. Read the file, judge, apply the fix in the current context; if the comment no longer makes sense, reply on the thread explaining why.
- **State file has a stale `pr_number`** (different from the current PR) — overwrite with a fresh object; treat as `empty_count: 0`.

## Interaction with other skills

- **`pr-commit`** — opens the PR and schedules this skill's cron. Handoff is automatic.
- **`commit`** — same conventions for commit messages (prefix types, co-author line, specific file staging).
- **`issue`** — if a reviewer surfaces a genuine bug the user doesn't want to fix in this PR, log it via the `issue` skill and push back on the comment.
