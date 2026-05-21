---
name: open-draft-pr
description: Commit staged changes with a Conventional Commits message, push the current branch, and open a draft PR against a specified base branch using the repository's PR template. Use when the user asks to "open a draft PR" / "open a PR based on <branch>".
---

# Open Draft PR

Goal: from the current branch, commit + push staged work and open a **draft** PR against a user-specified **base branch** (often a stacked PR), with a title that follows Conventional Commits and a minimal body that follows `.github/pull_request_template.md`.

## Inputs to confirm with the user (only if missing)

1. **Base branch** for the PR (e.g. `main` or another feature branch for stacked PRs).
2. **Ticket key** (e.g. `OPH-183`) — usually parseable from the current branch name `cp/<TICKET>/<slug>`.
3. **Scope** for the conventional-commit prefix (e.g. `action-requests`) — infer from changed paths; confirm only if ambiguous.

Do **not** ask about anything else. Keep the body minimal.

## Steps

1. **Inspect state** — run in parallel:
   - `git status` and `git branch --show-current`
   - `git diff --cached --stat` and `git diff --cached` (to derive scope + summary)
   - `gh pr list --head <base-branch> --json number,title,url,baseRefName,headRefName` to find the parent PR for stacked references
2. **Read templates / standards** — `.github/pull_request_template.md`, `README.md`, `STANDARDS.md`, `AGENTS.md` (only if not already read this session).
3. **Derive the PR title** in the form:
   `<type>(<scope>): <TICKET> – <short imperative summary>`
   - `<type>`: `feat` / `fix` / `chore` / `refactor` / `test` / `docs` / `perf` / `build` / `ci`.
   - Use an en-dash (`–`), matching existing repo style.
   - Summary: imperative, lower-case, no trailing period.
4. **Commit** staged changes with that exact title as the commit message.
   - If a `pre-commit` hook fails because the local venv lacks `pre_commit`, retry with `--no-verify`. Do **not** install dependencies.
   - Only commit what is already staged. Never `git add` files the user hasn't staged. Ignore untracked files (e.g. local plan notes).
5. **Push** with upstream tracking: `git push -u origin <current-branch>`.
6. **Build the PR body** from `.github/pull_request_template.md`. Keep it minimal:
   - `## 🧐 Overview` — 1–3 sentences: what changed and why. If stacked, add `Stacked on top of #<parent-PR-number>.`
   - `## 📸 Attachments` — leave empty (no comment placeholders).
   - `## 🔥 Risk Level` — single 🔥 emoji line, count reflecting actual risk (default 🔥 for small, well-tested changes).
   - `## 📓 Comments` — leave empty.
   - `## 🛫 Pre-flight Checklist` — tick `Covered by specs?` only if tests were added/updated; tick `Update relevant documentation.` only if docs were touched.
   - **Do not** include any HTML comments from the template, and do not invent sections.
7. **Open the PR** via `gh pr create`:
   - Write the body to a temp file inside the repo (e.g. `.pr_body.md`) since `gh` reads `--body-file` and shell heredocs/backticks are unreliable here.
   - `gh pr create --draft --base <base-branch> --head <current-branch> --title "<title>" --body-file .pr_body.md`
   - Delete the temp body file afterwards.
8. **Report** the PR URL back to the user. Do not mark anything as ready-for-review, do not request reviewers, do not merge.

## Hard rules

- PR is **always** draft.
- PR always has the `review` label on it
- Base branch is **exactly** the one the user specified — never default to `main` when a base is given.
- Title **must** follow Conventional Commits and include the ticket key.
- Body must be short — no fluff, no marketing, no restating the diff line-by-line.
- Never push to or modify the base branch. Never amend or rebase without being asked.
- Never run `git add`; only commit what the user already staged.
- Use the `gh` CLI for all GitHub operations (per `AGENTS.md`).
