# Project Governance: Claude (Supervisor) & Muse (Worker)

## Roles
| Agent | Role | Owns | Never does |
|---|---|---|---|
| **Claude Code** | Architect / Supervisor | Worktree creation, task specs, final review, merge to main | Write inside a worktree while a Muse worker holds it |
| **Muse Code** | Execution Worker | Mass refactors, test additions, lint fixes inside its assigned worktree | Push to `main`, edit outside its worktree, merge its own work |

## Environment Commands
Detect the stack from the repo root, then use the matching row. If neither manifest exists, stop and ask; do not guess.

| Manifest | Build | Test | Format / Lint fix |
|---|---|---|---|
| `package.json` | `npm run build` | `npm test` | `npm run lint -- --fix` |
| `Cargo.toml` | `cargo build` | `cargo test` | `cargo fmt && cargo clippy --fix --allow-dirty` |

## Cross-Agent Workflow Rules

### 1. Single writer per branch
- One branch = one writer at a time. No simultaneous file writes on the same branch.
- Ownership is declared by the `Owner:` field in that worktree's `MUSE_TASK.md`. No file, no ownership: Muse may not start.

### 2. Worktree handoff (Claude → Muse)
1. Claude creates the worktree: `git worktree add ../wt-<task-id> -b muse/<task-id>`
2. Claude writes `MUSE_TASK.md` at the worktree root from `.governance/templates/MUSE_TASK.md`, then commits it as the first commit on the branch.
3. Claude sets `Status: ASSIGNED` and spawns the Muse worker in that worktree.
4. Claude does not write to the worktree until Muse sets `Status: DONE` or `BLOCKED`.

### 3. Execution records (Muse)
- Muse copies `.governance/templates/CRASH_CONTEXT.md` into the worktree root before its first batch.
- Muse appends one log entry **per batch** (not per session) and commits it with the batch. A crash must lose at most one batch.
- On restart, Muse reads `CRASH_CONTEXT.md` first and resumes from `Last completed batch`.

### 4. Completion gate (Muse → Claude)
Muse may set `Status: DONE` only when all of these pass in the worktree:
- [ ] Build passes
- [ ] Tests pass (no tests skipped, disabled, or deleted to reach green)
- [ ] Lint/format clean
- [ ] Every acceptance criterion in `MUSE_TASK.md` checked off
- [ ] Diff stays inside the declared `Scope`

### 5. Review and merge (Claude)
- Claude re-runs build + test, reviews the diff against `MUSE_TASK.md`, then either merges or returns it with `Status: CHANGES_REQUESTED` and notes.
- Before merge, delete `MUSE_TASK.md` and `CRASH_CONTEXT.md` from the branch; they are not shipped.
- After merge: `git worktree remove ../wt-<task-id>` and delete the branch.

## Status values
`ASSIGNED` → `IN_PROGRESS` → `DONE` | `BLOCKED` → (`CHANGES_REQUESTED` → `IN_PROGRESS`) → `MERGED`
