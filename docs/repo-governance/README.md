# Repo governance

How `vanalen-metrics-hub` is protected, and why. The JSON files in this folder are the exact
ruleset payloads applied through the GitHub API. If the rules change, update the JSON here in
the same PR.

| File | Ruleset | Targets |
|---|---|---|
| `protect-dev.json` | `protect-dev` | `refs/heads/dev` |
| `protect-main.json` | `protect-main` | `refs/heads/main` |

## Branch model

- `dev` is the default branch. All feature PRs target `dev` and are **squash merged**.
- `main` is demo-ready only. It is updated by a `dev` → `main` PR using a **merge commit**.
- Nobody pushes directly to either branch, including the org owner. Both rulesets have an
  empty bypass list.

## Repository merge settings

| Setting | Value | Why |
|---|---|---|
| `allow_squash_merge` | `true` | One commit per feature on `dev`. |
| `allow_merge_commit` | `true` | Used only for `dev` → `main` releases. |
| `allow_rebase_merge` | `false` | Rebase merges rewrite commits and drop their signatures. |
| `allow_update_branch` | `true` | Shows the "Update branch" button when a PR is behind `dev`. |
| `delete_branch_on_merge` | `true` | Merged work branches are cleaned up automatically. |

## Rules

| Rule | `dev` | `main` | Why |
|---|---|---|---|
| `deletion` | yes | yes | The branch cannot be deleted. |
| `non_fast_forward` | yes | yes | No force pushes, so history is never rewritten. |
| `required_linear_history` | yes | no | `dev` stays a straight line of squashed commits. `main` needs merge commits for releases. |
| `pull_request` | yes | yes | Every change goes through a PR. |
| `required_status_checks` | yes | yes | The `checks` and `secret-scan` jobs from `.github/workflows/ci.yml` must pass. |

### Pull request settings (both branches)

- `required_approving_review_count: 1`: one teammate must approve.
- `dismiss_stale_reviews_on_push: true`: new commits clear earlier approvals, so what gets
  merged is what was reviewed.
- `required_review_thread_resolution: true`: every review comment thread must be resolved.
- `require_code_owner_review: false`: the tech lead is the only code owner and cannot approve
  their own PRs, so requiring code owner review would block them.
- `require_last_push_approval: false`: GitHub's default, left unchanged.

### Status checks: strict on `dev`, not on `main`

`strict_required_status_checks_policy` requires the PR branch to contain the latest commit of
the target branch before it can merge.

- On `dev` it is `true`: a feature branch must be up to date with `dev`, so checks ran against
  what will actually land.
- On `main` it is `false`. Each release adds a merge commit to `main` that `dev` can never
  contain, because `dev` forbids merge commits. With the strict policy on, the second release
  would be permanently blocked.

## Commit signing

Commits should be signed so GitHub shows them as "Verified". Signing proves a commit came from
the person it names. It is not enforced by a ruleset today, because squash merges are signed by
GitHub rather than by the author.

### Set up SSH signing

1. Create a key if you don't have one, and set a passphrase when asked:

   ```bash
   ssh-keygen -t ed25519 -C "you@example.com"
   ssh-add --apple-use-keychain ~/.ssh/id_ed25519   # macOS; plain `ssh-add` elsewhere
   ```

2. Tell git to sign with it. Run this inside the repo:

   ```bash
   git config user.name "Your Name"
   git config user.email "you@example.com"
   git config gpg.format ssh
   git config user.signingkey ~/.ssh/id_ed25519.pub
   git config commit.gpgsign true
   git config tag.gpgsign true
   ```

   The email must be a verified email on your GitHub account.

3. Upload the public key to GitHub as a **signing** key. This is separate from an
   authentication key, even if it is the same key:

   ```bash
   gh auth refresh -h github.com -s admin:ssh_signing_key
   gh ssh-key add ~/.ssh/id_ed25519.pub --type signing --title "My signing key"
   ```

4. Optional: let git verify signatures locally.

   ```bash
   echo "you@example.com $(cut -d' ' -f1,2 ~/.ssh/id_ed25519.pub)" > .git/allowed_signers
   git config gpg.ssh.allowedSignersFile "$(pwd)/.git/allowed_signers"
   ```

5. Check it works. `G` means a good signature:

   ```bash
   git commit --allow-empty -m "test: signing"
   git log -1 --format='%an <%ae> | %G?'
   git reset --hard HEAD~1
   ```

## Re-applying the rulesets

Requires admin access. Look up the ruleset ID, then update it in place:

```bash
REPO=vanalen-metrics-hub/vanalen-metrics-hub
gh api repos/$REPO/rulesets --jq '.[] | "\(.id) \(.name)"'
gh api -X PUT repos/$REPO/rulesets/<id> --input docs/repo-governance/protect-dev.json
```

Use `-X POST repos/$REPO/rulesets` only if the ruleset does not exist yet.
