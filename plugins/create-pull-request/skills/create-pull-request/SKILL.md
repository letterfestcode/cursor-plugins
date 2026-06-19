---
name: create-pull-request
description: Create a GitHub pull request as a draft, filling the repository's PR template exactly and prefixing the title with a Linear ticket id when one is present. Use when the user asks to create/open/raise/draft a PR or pull request, "make a PR", "open a pull request", or "raise a PR for this branch". Requires the gh CLI; uses the Linear MCP to fetch ticket details when a ticket id is detected.
metadata:
  mcp-servers: linear
  version: 1.0.0
---

# Create Pull Request

Create a **draft** GitHub PR for the current branch. Two hard rules: follow the repo's PR template exactly, and prefix the title with the Linear ticket id when work maps to a ticket — otherwise use Conventional Commits format.

## Prerequisites
- `gh` CLI authenticated (`gh auth status`). If not, stop and tell the user.
- Linear MCP is optional — degrade gracefully if unavailable (see Ticket detection).
- Run from inside the target git repo.

## 1. Resolve scope
- Current branch: `git branch --show-current`.
- Base branch (hardcoded set, no remote API call): if `git show-ref --verify --quiet refs/remotes/origin/dev` succeeds, base is `dev` (gitflow/github-flow repos); otherwise base is `main`. These are the only two safe defaults.
- Changes: `git log <base>..HEAD --oneline` and `git diff <base>...HEAD --stat` to understand the actual diff (the title summary is derived from this, not from the ticket).

## 2. Pre-flight checks (stop conditions)
- **No changes vs base:** if `git log <base>..HEAD --oneline` is empty (the branch has no commits ahead of `<base>`), STOP — do not create a PR. Tell the user there is nothing to open a PR for.
- **Branch is the base:** if the current branch is `main` or `dev`, STOP — never open a PR from a base branch onto itself.
- **PR already exists:** check `gh pr view --json url,state,isDraft` (or `gh pr list --head "$(git branch --show-current)" --json url,state`). If an open PR already exists for the branch, STOP and return its URL instead of creating a new one.

## 3. Ticket detection
- Scan, case-insensitively, the user's prompt, the full branch name, and commit messages for a Linear id matching `ENG-\d+` (other team prefixes too if the repo uses them). Normalise to uppercase.
- If an id is found, you MUST call the Linear MCP issue-fetch tool (e.g. `get_issue`) before writing the PR, to get the issue URL/title for context. If the MCP is unavailable or the call fails, continue anyway.

## 4. Title
- If a ticket id was found: `ENG-1234: <summary>`.
- Otherwise, fall back to Conventional Commits format: `<type>: <summary>` (e.g. `feat: ...`, `fix: ...`, `chore: ...`, `refactor: ...`, `test: ...`, `docs: ...`). Pick the `<type>` that best fits the actual diff; add a `(<scope>)` only when it adds clarity.
- `<summary>` is an overall summary of the actual changes (from the diff/commits), approx under 80 chars (soft limit). Do NOT copy the Linear ticket title — only the id comes from the ticket.

## 5. Body — fill the template exactly
- Locate the repo's template: `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, root `PULL_REQUEST_TEMPLATE.md`, or `docs/PULL_REQUEST_TEMPLATE.md`.
- If a template exists, follow it with NO deviation:
  - Fill each section exactly as its HTML comment instructs.
  - Do NOT reformat, reorder, rename, or add sections.
  - Preserve separators (`---`) and tooling comments verbatim (e.g. the Greptile comment).
  - Remove a section ONLY when (and exactly as) the template explicitly marks it optional/removable; keep all others verbatim.
  - Replace each comment with real content: bullet points; "why over what" for the description.
  - `Ticket:` line → the Linear issue URL from the MCP when known, else `N/A`.
- If NO template exists, use the bundled default at `references/default-pr-template.md` and fill it the same way (only its `Context` and `Screenshots / videos` sections are removable).

## 6. Open the PR (draft)
- Upstream: if `git rev-parse --abbrev-ref --symbolic-full-name @{u}` fails, push first with `git push -u origin HEAD`. Never force-push; never push to `main`/`dev`.
- Then create the draft PR. Use a heredoc so the body formats correctly:
```bash
gh pr create --draft --base <base> --title "<title>" --body "$(cat <<'EOF'
<filled template body>
EOF
)"
```
Return the PR URL to the user.

## Safety
- Draft only. Never force-push, never push directly to `main`/`dev`, never amend or rewrite history.
