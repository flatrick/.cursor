# cursor-home

Safe-to-share baseline for a Cursor home directory (user-level rules, skills, and docs).

## What this repo tracks

- `rules/` (user-level Cursor rules, e.g. `ai-command-safety.mdc`)
- `skills/` (your personal domain skills: languages, DBs, CI, review, QA, security)
- `commands/` (custom slash commands: `.md` files; each file = one `/command` in chat)
- `agents/` (user-level subagent definitions: `.md` with YAML frontmatter; available in all projects)
- `docs/` (optional workflow guidelines and references)
- `README.md`, `.editorconfig`, `.gitattributes`, `.gitignore`

If you have `skills-cursor/` or `plans/` on disk, they are local-only (not in the allowlist).

User-level rules apply in all workspaces unless a project defines its own `.cursor/rules/`. Project rules take precedence in that project.

## Commands and subagents (Cursor-unique)

- **Commands**: Add `.md` files to `~/.cursor/commands/`. Each filename becomes a slash command (e.g. `review-code.md` → `/review-code`). The file content is the prompt Cursor uses when you run that command. They appear when you type `/` in chat.
- **Subagents**: Add `.md` files to `~/.cursor/agents/` with YAML frontmatter (`name`, `description`, optional `model`). The body is the system prompt. The main Agent can delegate to these for context isolation or parallel work. For project-only agents, use `.cursor/agents/` in a repo. See the **create-subagent** skill (in `skills-cursor/`, if present locally) for full format and examples.
- **Runtime state**: Cursor writes background subagent output and transcripts to `~/.cursor/subagents/`. That directory is typically not committed; we track `agents/` (definitions) and `commands/` (definitions), not `subagents/` (runtime).

## What this repo does not track

The `.gitignore` uses an allowlist: everything at top level is ignored except the paths listed in "What this repo tracks." Not tracked:

- Cursor IDE state (extensions, snapshots, ai-tracking, etc.)
- The entire `projects/` tree
- `subagents/` (runtime state: background subagent output and transcripts)
- `plugins/` (plugin cache)
- `skills-cursor/`, `plans/`, and any other path not in the allowlist

## Bootstrap on a new machine

1. Back up existing `~/.cursor` if present.
2. Clone this repo into `~/.cursor` (or copy tracked paths into an existing `.cursor`).
3. Ensure `rules/` and `skills/` are present and that Cursor is pointed at this directory for user config. You may add or copy `skills-cursor/` locally if desired.
4. Open Cursor and confirm user rules and skills are visible (e.g. in Settings → Rules, and in the skills list).

## Operations (day-2)

### New machine setup verification checklist

After bootstrap, verify:

- **Rules loaded**
  - Confirm `rules/ai-command-safety.mdc` (or your guardrail rule) is present and applied globally.
  - Open a project and confirm the rule is active (e.g. risky commands are refused with the expected phrase).
- **Skills visible**
  - Ensure `skills/` exists and contains the expected `SKILL.md` files. If you use `skills-cursor/`, it is not in the repo—ensure that directory exists locally.
  - In Cursor, confirm skills are available when starting a chat or agent.
- **Sample prompt works**
  - Run a small prompt (e.g. ask for a summary of this README) and confirm the agent respects guardrails and can use skills.

### Upgrade path (safe pulls)

When updating from upstream:

1. Inspect local changes: `git status`
2. If you have local edits, commit them on a branch or stash: `git stash push -u -m "pre-upgrade $(date +%F)"`
3. Fetch and fast-forward: `git fetch origin` then `git pull --ff-only`
4. Re-apply local changes if stashed: `git stash pop`
5. Resolve conflicts with priority: keep upstream changes in shared rules and skills; re-apply any machine-specific tweaks manually.

### Disaster recovery (restore + re-auth)

If the working copy is corrupted or misconfigured:

1. Preserve anything you need: `cp -a ~/.cursor ~/.cursor.backup.$(date +%F-%H%M%S)`
2. Re-clone a clean copy of this repo into `~/.cursor` (or re-copy only the tracked paths).
3. Restore any local-only edits from backup.
4. Re-open Cursor and confirm rules/skills and auth (Cursor manages auth separately; no auth files are in this repo).

### Troubleshooting

| Symptom | Likely cause | What to check/fix |
| --- | --- | --- |
| Rules not applied | Wrong config path or rule file missing | Verify Cursor uses `~/.cursor` for user config and `rules/*.mdc` exist. |
| Skills not listed | Skills dir not present or not in allowlist | Check `skills/` exists and is tracked. `skills-cursor/` is not in the repo; if you use it, ensure the directory exists locally. |
| Guardrails not firing | Rule disabled or overridden | Confirm `alwaysApply: true` in the rule frontmatter; check for project-level overrides. |

## Safety checks before sharing publicly

- Run `git status` and `git diff --cached` before each commit.
- Never force-add ignored files that may contain secrets or local state.
- If a secret is ever committed, rotate it immediately.
