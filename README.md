# cursor-home

Safe-to-share baseline for a Cursor home directory (user-level rules, skills, and docs).

## What this repo tracks

- `rules/` (user-level Cursor rules, e.g. `ai-command-safety.mdc`)
- `skills/` (your personal domain skills: languages, DBs, CI, review, QA, security)
- `skills-cursor/` (Cursor meta-skills: create-rule, create-skill, update-cursor-settings, etc.)
- `commands/` (custom slash commands: `.md` files; each file = one `/command` in chat)
- `agents/` (user-level subagent definitions: `.md` with YAML frontmatter; available in all projects)
- `plans/` (plan files for sync, safety, and other improvements)
- `docs/` (optional workflow guidelines and references)
- `README.md`, `.editorconfig`, `.gitattributes`, `.gitignore`

User-level rules apply in all workspaces unless a project defines its own `.cursor/rules/`. Project rules take precedence in that project.

## Commands and subagents (Cursor-unique)

- **Commands**: Add `.md` files to `~/.cursor/commands/`. Each filename becomes a slash command (e.g. `review-code.md` → `/review-code`). The file content is the prompt Cursor uses when you run that command. They appear when you type `/` in chat.
- **Subagents**: Add `.md` files to `~/.cursor/agents/` with YAML frontmatter (`name`, `description`, optional `model`). The body is the system prompt. The main Agent can delegate to these for context isolation or parallel work. For project-only agents, use `.cursor/agents/` in a repo. See the **create-subagent** skill (in `skills-cursor/`) for full format and examples.
- **Runtime state**: Cursor writes background subagent output and transcripts to `~/.cursor/subagents/`. That directory is typically not committed; we track `agents/` (definitions) and `commands/` (definitions), not `subagents/` (runtime).

## What this repo does not track

The `.gitignore` blocks local state, IDE data, and most of `projects/`:

- Cursor IDE state (extensions, snapshots, ai-tracking, etc.)
- Most of `projects/` except allowlisted subdirs: `mcps/`, `agent-transcripts/`, `terminals/`, `agent-notes/`, `agent-tools/`
- `subagents/` (runtime state: background subagent output and transcripts; usually left untracked)
- `plugins/` (plugin cache; un-ignored but often empty)

## Bootstrap on a new machine

1. Back up existing `~/.cursor` if present.
2. Clone this repo into `~/.cursor` (or copy tracked paths into an existing `.cursor`).
3. Ensure `rules/`, `skills/`, and `skills-cursor/` are present and that Cursor is pointed at this directory for user config.
4. Open Cursor and confirm user rules and skills are visible (e.g. in Settings → Rules, and in the skills list).

## Operations (day-2)

### New machine setup verification checklist

After bootstrap, verify:

- **Rules loaded**
  - Confirm `rules/ai-command-safety.mdc` (or your guardrail rule) is present and applied globally.
  - Open a project and confirm the rule is active (e.g. risky commands are refused with the expected phrase).
- **Skills visible**
  - Ensure `skills/` and `skills-cursor/` contain the expected `SKILL.md` files.
  - In Cursor, confirm skills are available when starting a chat or agent.
- **Sample prompt works**
  - Run a small prompt (e.g. ask for a summary of this README) and confirm the agent respects guardrails and can use skills.

### Upgrade path (safe pulls)

When updating from upstream:

1. Inspect local changes: `git status`
2. If you have local edits, commit them on a branch or stash: `git stash push -u -m "pre-upgrade $(date +%F)"`
3. Fetch and fast-forward: `git fetch origin` then `git pull --ff-only`
4. Re-apply local changes if stashed: `git stash pop`
5. Resolve conflicts with priority: keep upstream changes in shared rules/skills/plans; re-apply any machine-specific tweaks manually.

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
| Skills not listed | Skills dir not present or not un-ignored | Check `skills/` and `skills-cursor/` exist and are tracked (see `.gitignore`). |
| Guardrails not firing | Rule disabled or overridden | Confirm `alwaysApply: true` in the rule frontmatter; check for project-level overrides. |

## Safety checks before sharing publicly

- Run `git status` and `git diff --cached` before each commit.
- Never force-add ignored files that may contain secrets or local state.
- If a secret is ever committed, rotate it immediately.
