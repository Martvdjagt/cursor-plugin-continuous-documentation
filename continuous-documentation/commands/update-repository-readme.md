---
name: update-repository-readme
description: >-
  Sync repository readme.md from git changes and Cursor agent transcripts using
  the incremental index. Run manually anytime; same workflow the stop hook suggests.
---

# Update repository README

Run this workflow from the **workspace root**. Treat the **continuous-documentation** skill as the source of truth for README shape, tone, inclusion bar, exclusions, slop filter, and index file format — apply it to every edit.

## Inputs

| Source | Role |
|--------|------|
| **Git history** | `git log` and `git diff` from the last indexed commit (see index) through HEAD — what changed. |
| **Transcripts** | `~/.cursor/projects/<workspace-slug>/agent-transcripts/` — why it changed; only files new or changed since last indexed mtime. |
| **`readme.md`** | Current doc to update (repository root for microservices, project-level for monoliths). |
| **Incremental index** | `.cursor/hooks/state/continuous-documentation-index.json` — last processed commit SHA and per-transcript mtimes; create or update at end of run. |

## Workflow

1. **Read** the existing `readme.md`.
2. **Determine project type** (Service, Monolith, UI, NuGet/NPM Package) from repository structure. Apply the matching README structure from the skill.
3. **Load** the incremental index if present: `.cursor/hooks/state/continuous-documentation-index.json`
4. **Gather what changed** (the What):
   - Run `git log --oneline` from the last indexed commit to HEAD.
   - For commits that touch documentation-relevant areas (new projects, changed domain/application layers, added endpoints, altered configuration), run `git diff` to understand the scope.
   - Identify: new capabilities, changed behavior, removed functionality, new integration points, architectural shifts, non-trivial business logic changes.
5. **Gather why it changed** (the Why):
   - Discover transcript files under `~/.cursor/projects/<workspace-slug>/agent-transcripts/`.
   - Process only new or changed transcripts (mtime newer than indexed in the incremental file).
   - From each transcript, extract stated reasoning: user-provided context, rejected alternatives, constraints, trade-offs, problem statements that prompted the change.
   - Correlate transcript intent with the git changes from step 4. Match by timeframe and subject matter.
6. **Merge What + Why** into README updates:
   - For each documentation-worthy change, write what it is and why it was done.
   - If the transcript provides no intent for a change, document the what only. Do not invent a why.
   - If the transcript reveals intent but the code change is trivial, skip it — intent without a meaningful what is noise.
7. **Update `readme.md`**:
   - Modify existing sections in place when content has changed.
   - Add new sections only when the structure calls for them.
   - Remove content that is no longer accurate.
   - Preserve the section structure for the detected project type.
8. **Run the slop filter** from the skill on every sentence.
9. **Write back** the incremental index:
   - Store latest commit SHA processed.
   - Store latest transcript mtimes.
   - Remove entries for transcripts that no longer exist.

If no meaningful documentation updates are warranted under the skill’s inclusion bar, respond exactly: `No documentation updates needed.`
