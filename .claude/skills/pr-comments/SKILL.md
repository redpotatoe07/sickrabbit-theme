---
name: pr-comments
description: Check the current PR for new Greptile (or human) review comments since the last push. If there are new ones, fix them, commit, push. If not, report "no new comments" and exit. Runs on the 8-minute cron scheduled by pr-commit, or invoke directly. Use when the user says "check the PR", "resolve comments", "what did the reviewer say", "fix the review", "address feedback", "are there new comments", or any variation. For opening a PR from scratch, use pr-commit.
---

# PR Comments

Check the current PR for new review comments since the last push. Fix them, commit, push. That's it.

Scheduled by `pr-commit` to run every 8 minutes. Keeps running until the PR is merged or the user cancels the cron.

## Workflow

### 1. Identify the PR

```bash
git branch --show-current
gh pr view --repo redpotatoe07/sickrabbit-theme --json number,state,url
```

- On `main`? Wrong branch. Tell the user and stop.
- PR `MERGED` or `CLOSED`? Cancel the cron (`CronDelete`) and stop.
- No PR for this branch? Stop.

### 2. Fetch comments posted since the last push

```bash
last_push=$(git log --format='%cI' -1)
gh api repos/redpotatoe07/sickrabbit-theme/pulls/<N>/comments --paginate \
  --jq ".[] | select(.created_at > \"$last_push\") | {id, path, line, body, diff_hunk, user: .user.login}"
gh api repos/redpotatoe07/sickrabbit-theme/issues/<N>/comments \
  --jq ".[] | select(.created_at > \"$last_push\") | {id, body_preview: (.body | .[0:500]), user: .user.login}"
```

If both are empty, output `no new comments` in one line and exit. Do not do anything else.

### 3. Fix the new comments

For each comment:

- Read the file first
- If it includes a ` ```suggestion ` block and the block still applies, use the suggestion
- Otherwise read the surrounding code and implement the minimal fix
- **Push back** (don't silently apply) when a comment contradicts a logged decision in `docs/decisions.md`, would break merchant admin editability, or is asking for a hardcoded value the token system is supposed to prevent. Reply on the PR thread with the reason and move on.

Batch fixes per file. No priority-ordering ceremony — just resolve what applies.

### 4. Commit and push

```bash
git add <only the files you touched>
git commit -m "$(cat <<'EOF'
fix: address PR review comments

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
git push
```

One commit for the batch is fine — these are review responses, not independent features.

### 5. Report and exit

```
Fixed N comments, pushed <sha>.
- <file>:<line> — <one-line summary>
- ...
```

The cron fires again in 8 minutes. If Greptile posts another round, the next fire handles it. If not, the next fire reports `no new comments` and exits. No state file, no fire counter, no auto-merge offer — the user merges when they're ready.

## Edge cases

- **PR merged or closed** — cancel the cron, stop.
- **On `main`** — wrong branch, stop.
- **No new comments since last push** — report one line, exit. Don't touch anything.
- **Merge conflicts on push** — report the error; the user decides how to recover.
- **Comment on a file the user has changed significantly since the review** — the suggestion may be stale. Read the file, judge, apply the fix in the current context; if the comment no longer makes sense, reply on the thread explaining why.

## Interaction with other skills

- **`pr-commit`** — opens the PR and schedules this skill's cron. Handoff is automatic.
- **`commit`** — same conventions for commit messages (prefix types, co-author line, specific file staging).
- **`issue`** — if a reviewer surfaces a genuine bug the user doesn't want to fix in this PR, log it via the `issue` skill and push back on the comment.
