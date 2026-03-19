# Continuous Documentation

Cursor plugin that keeps the repository `readme.md` current by mining conversation transcripts for documentation-worthy changes, capturing intent and reasoning behind decisions.

## How it works

Three pieces work together:

| Piece | Role |
|--------|------|
| **`update-repository-readme` command** | Step-by-step workflow: git + transcripts + incremental index → update `readme.md`. Run from the command palette anytime; does not require the hook. |
| **`continuous-documentation` skill** | README structure, inclusion/exclusion, slop filter, and intent guidance. Use on its own for doc edits, or as the rule set the command must follow. |
| **`stop` hook** | Tracks conversation cadence; when thresholds pass, suggests running the command. |

Two sources of truth feed the sync:

- **Git history** — what changed (`git log`, `git diff` since the last indexed commit).
- **Conversation transcripts** — why it changed (reasoning, alternatives, constraints).

Processing is incremental — only new commits and changed transcripts are evaluated on each run.

## Prerequisites

The hook script runs with [Bun](https://bun.sh/). Ensure `bun` is available on `PATH`.

## State files

| File | Purpose |
|------|---------|
| `.cursor/hooks/state/continuous-documentation.json` | Cadence state (turn count, last run time, transcript mtime) |
| `.cursor/hooks/state/continuous-documentation-index.json` | Incremental index (last commit SHA, processed transcript mtimes) |

Both are workspace-local and excluded from version control.

## Trigger cadence

| Setting | Default | Trial mode |
|---------|---------|------------|
| Minimum turns | 10 | 3 |
| Minimum minutes | 120 | 15 |
| Trial duration | — | 24 hours |

All conditions must be met simultaneously. Transcript mtime must also advance since the previous run.

## Configuration

All settings are optional environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `CONTINUOUS_DOCUMENTATION_MIN_TURNS` | Minimum completed turns before triggering | 10 |
| `CONTINUOUS_DOCUMENTATION_MIN_MINUTES` | Minimum minutes between runs | 120 |
| `CONTINUOUS_DOCUMENTATION_TRIAL_MODE` | Enable reduced thresholds for a trial window | false |
| `CONTINUOUS_DOCUMENTATION_TRIAL_MIN_TURNS` | Minimum turns in trial mode | 3 |
| `CONTINUOUS_DOCUMENTATION_TRIAL_MIN_MINUTES` | Minimum minutes in trial mode | 15 |
| `CONTINUOUS_DOCUMENTATION_TRIAL_DURATION_MINUTES` | Trial window length in minutes | 1440 |

## License

MIT
