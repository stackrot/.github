# .github

The PR workflow baseline shared by every `stackrot` and `HallowedRS` repo.
Every change lands through an issue and a squash-merged pull request, with
CI, a PR policy check and a Claude code review. `CONTRIBUTING.md` explains
the process.

This repo also supplies the default community health files for `stackrot`
repos that don't have their own.

## What's here

| Path | Purpose |
| --- | --- |
| `CONTRIBUTING.md` | The contribution guide each repo copies |
| `.github/pull_request_template.md` | PR template with the AI disclosure section |
| `.github/ISSUE_TEMPLATE/` | Bug, feature and chore forms |
| `.github/workflows/reusable-pr-policy.yml` | Fails a PR that doesn't close an issue or lacks the AI disclosure |
| `.github/workflows/reusable-claude-review.yml` | Runs the official `code-review` plugin and posts inline findings |
| `templates/workflows/` | The two caller workflows each repo gets |
| `scripts/baseline` | Copies the files in and applies a repo's settings |

## Setting up a repo

```sh
# Store the review token (from `claude setup-token`) as a repo secret. Do
# this first: the files PR runs the review check.
scripts/baseline secret TOKEN_FILE OWNER/REPO...

# Squash-only merges, auto-merge, delete merged branches, labels, and the
# default-branch ruleset requiring the policy and review checks. Rerun it
# with the repo's own CI check runs after the name once they've reported on
# a PR, so the ruleset requires them too.
scripts/baseline settings OWNER/REPO
scripts/baseline settings OWNER/REPO "ci / test"

# Copy the guide, templates and caller workflows into a checkout. Fill in
# the guide's Development section and land it all through a PR. The PR adds
# the callers, so they run on it.
scripts/baseline files /path/to/checkout

# For an organisation on GitHub Team: a ruleset covering every repo,
# including new ones. Every repo then needs the baseline files before any
# other PR can merge.
scripts/baseline org-ruleset HallowedRS

# Just the label set: bug, feature, chore, found-in-passing,
# needs-approval and approved. Moves enhancement and documentation onto
# feature and chore, then deletes GitHub's other default labels.
scripts/baseline labels OWNER/REPO

# List active repos missing the guide, a ruleset or the secret.
scripts/baseline audit
```

Only list checks that run on every PR. A required check that a path filter
skips never reports, and the PR can't merge.

A PR that changes `claude-review.yml` can't be reviewed by Claude, since
the action won't run a workflow that differs from the default branch's. The
check passes with a warning, so review those PRs with `/code-review`.

The review token comes from a Claude subscription and lasts a year. To
rotate it, run `claude setup-token` again and rerun `scripts/baseline
secret` for every repo.
