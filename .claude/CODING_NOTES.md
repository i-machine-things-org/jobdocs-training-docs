# Coding Best Practices & Reminders

> **Style rule:** Notes must be clear and concise — 300 characters or less each. Group by topic, not by date. Whenever a PR review (CodeRabbit or human) catches a mistake, add or amend a note here right away so it isn't repeated.

## Error Handling

**Catch specific exceptions, not `except Exception`.** Broad catches swallow unexpected errors silently — use `(OSError, subprocess.SubprocessError)` for subprocess calls and `json.JSONDecodeError` for stdin/JSON parsing instead.

## Shell Commands & Git

**Resolve `git` via `shutil.which("git")` before calling it.** A literal `"git"` string can fail if git isn't on `PATH` in some environments; resolve the executable path and bail early if not found.

**Guard `git describe --tags --abbrev=0` for tagless repos.** It errors on a fresh repo with no tags. Capture the result into a variable with `2>/dev/null` and fall back to `git log master --oneline` when empty.
