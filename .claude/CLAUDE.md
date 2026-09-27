# Auto Version Control Rules - Claude AI

You are a senior software developer. These rules override your default behavior. Follow them on every action without being asked.

## Rule 0: Always Read First

Before taking any action on this project — including edits, commits, or file creation:

1. Read `.claude/CLAUDE.md` and `.claude/CODING_NOTES.md`.
2. Run `gh pr list` — if a PR exists for the current branch, run `gh pr view <number> --comments` and read **all comments** (CodeRabbit and human) before proceeding.
3. Run `gh issue list` — check for open issues relevant to the current work.
4. Do not make any edits until all outstanding findings and review comments are addressed or acknowledged.

No exceptions.

### Checking PR review status

`.claude/CODING_NOTES.md` is a standards and practices reference — a log of coding patterns and past findings, grouped by topic. It is **not** the source of truth for PR review status.

- To check if a PR review is complete or paused: **always use `gh pr view <number> --comments`**.
- CodeRabbit may auto-pause reviews after rapid commits — check for `review paused` in the summary comment.
- If paused, trigger a new run with: `gh pr comment <number> --body "@coderabbitai review"`
- If CR hits a rate limit (`Rate limit exceeded`), run `date -u` to get the current UTC time, calculate the UTC timestamp when the window clears, and state it explicitly. Re-trigger on the first user interaction at least 5 minutes after that time.
- **Sequential PR workflow:** Open one PR, wait for CR to finish and address all findings, merge, then open the next.

## Trigger Prompt

When the user says **"run auto version control"** (or any close variation), immediately run the full assessment:

1. Run `git status`, `git branch`, and `git log --oneline -10`
2. Run `gh issue list` and report any open issues
3. Report the current state: branch, uncommitted changes, recent commits, version tags
4. Flag any issues: working on main, uncommitted changes, missing .gitignore, no tags
5. Recommend next actions

## Rule 1: Git Is Mandatory

- If the project is not a git repository, run `git init` and create an initial commit before doing anything else.
- Never work directly on `master`. Always create a feature branch first then merge into `master`.
- Branch naming: `feat/description`, `fix/description`, `refactor/description`, `docs/description`, `chore/description`.
- If you are on `master` when you start, create and switch to a feature branch immediately.

## Rule 2: Conventional Commits

Every commit message must follow this format:

```
type: short description (imperative, lowercase, no period)
```

Valid types: `feat`, `fix`, `refactor`, `docs`, `test`, `style`, `perf`, `chore`, `ci`, `build`.

Examples:
- `feat: add category grouping to guide tree`
- `fix: handle missing training_meta.json gracefully`
- `docs: update README with deployment steps`

Rules:
- One logical change per commit. Do not bundle unrelated changes.
- Commit after every meaningful change, not at the end of a long session.
- If a new feature is added or changed, update `README.md` before committing.
- After every commit, check if a PR exists (`gh pr list --head <branch>`). If none exists, open one immediately via `gh pr create`.

## Rule 3: Fork From the Template — Backport Template Changes

This repository was forked from `jobdocs-plugin-template`. The following files are
**template files** owned upstream — changes that improve them must be PR'd back to
`jobdocs-plugin-template` before (or alongside) merging here:

```
.claude/CLAUDE.md
.claude/settings.json
.claude/hooks/pre_commit_sp_check.py
README.md                    (structure/sections, not plugin-specific content)
```

Plugin-specific logic, UI, and `.claude/CODING_NOTES.md` entries are **not** backported.

## Rule 4: This Is an External Plugin — Not Part of JobDocs Core

- Do **not** add this plugin's code to `JobDocs/modules/`.
- UI path resolution must use `Path(__file__).parent` — never `sys._MEIPASS`.
- Plugin-specific settings must be surfaced through the plugin's own UI (Browse button pattern).
- Dependencies go in `requirements.txt`; JobDocs installs them automatically.

## Rule 5: Semantic Versioning

Tag releases using `vMAJOR.MINOR.PATCH` from `master` only.

- **MAJOR** — breaking changes
- **MINOR** — new features
- **PATCH** — bug fixes

### Rehearse before releasing

Any workflow that publishes something (a release, a package, a deploy) must have a **rehearsal mode** that does everything except the final publish step. For GitHub Actions that means a `workflow_dispatch` trigger with an input for the version or tag, and a publish job that is skipped when it is a rehearsal. A rehearsal needs no tag. GitHub only lets you dispatch a workflow whose file is already on the default branch, but the run uses the workflow file from the `--ref` you pick, so a change to an existing publish workflow can be rehearsed from its own branch before it merges (for example `gh workflow run release.yml --ref <branch> -f tag=vX.Y.Z`, using the repo's real workflow file and input names). A brand-new publish workflow has to merge first, with its publish job already skipped on dispatch so that merging publishes nothing, and is then rehearsed from the default branch before the first tag.

Rehearse before the first real tag, and again after any change to the release pipeline:

1. Run the rehearsal and read its job summary (the release notes it would publish, the artifacts it built).
2. **Download the rehearsal's artifacts and use them the way a user will**: install the package, import the plugin, run the binary on a clean machine or in a clean container. A green rehearsal proves the pipeline ran. It does not prove the artifact works where it is consumed: a package that built and passed its rehearsal can still be refused by the tool that imports it (for example, because its target-system label does not match the computer).
3. Only then tag. The human sign-off before tagging still applies (Rule 6 in repos that have it), and the rehearsal result belongs in the summary you give the human.

Do not rehearse by tagging a throwaway version: a pushed tag is public and hard to take back. If a repo has no publish workflow yet, this rule waits until one exists.

### Automatic Version Bump Triggers

After every merge to `master`, count commits since the last `v*` tag:

```bash
last_tag=$(git describe --tags --abbrev=0 2>/dev/null)
if [ -n "$last_tag" ]; then
  git log "$last_tag"..master --oneline
else
  git log master --oneline
fi
```

- **5 or more `feat:` commits** → bump MINOR
- **5 or more `fix:` commits** → bump PATCH

Check after every merge. Do not wait for the user to ask.

## Rule 6: Pull Request Reviews

- Always open PRs via `gh pr create` — never merge directly to `master`.
- Read all CodeRabbit and human comments before making further changes.
- For each finding, regardless of source:
  1. If it matches an existing `.claude/CODING_NOTES.md` entry — fix it immediately and reference the note's topic in the commit message.
  2. If it is a new pattern — fix it, then add or amend a note under the relevant topic in `.claude/CODING_NOTES.md` before committing, following that file's style rule (clear, ≤300 characters, grouped by topic).
- Do not dismiss or ignore nitpicks — log them to `.claude/CODING_NOTES.md` even if not immediately actionable.
- Only merge a PR after all blocking comments are resolved and documentation has been updated.
