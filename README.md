# agent-skills

Generally useful agent skills in the open [Agent Skills](https://agentskills.io) format. They are
published as a [Claude Code](https://code.claude.com/docs) plugin marketplace, and work with any
agent that reads `SKILL.md` files.

## Plugins and skills

| Plugin         | Skill                | What it does                                                        |
|----------------|----------------------|---------------------------------------------------------------------|
| `dev-workflow` | `reviewable-commits` | Splits work into functional commits, each reviewed and approved before it is committed |

Plugin skills are namespaced: run one as `/dev-workflow:reviewable-commits`, or let Claude load it
when a request matches its description.

## Layout

```
.claude-plugin/marketplace.json        the marketplace: lists the plugins in this repo
plugins/<plugin>/.claude-plugin/plugin.json
plugins/<plugin>/skills/<skill>/SKILL.md
```

Start new skills in an existing plugin. Split a plugin when a clear group of skills emerges, so people
can install only what they need. Add the new plugin to `marketplace.json`.

Keep skills portable: put everything that decides when a skill triggers in `description`, since other
agents ignore Claude Code's extra frontmatter fields such as `when_to_use`.

## Install

### Claude Code

```powershell
claude plugin marketplace add ricflams/agent-skills
claude plugin install dev-workflow@ricflams
```

From a clone, pass its path instead: `claude plugin marketplace add .\agent-skills`.

### Other agents

Codex, Gemini CLI, Copilot, Cursor and other agents that support Agent Skills can use a skill
directly: copy its folder, for example `plugins/dev-workflow/skills/reviewable-commits`, into the
agent's skills directory. See the agent's documentation for where that is.

## Develop

From the repo root:

```powershell
claude plugin validate .
claude --plugin-dir .\plugins\dev-workflow
```

`--plugin-dir` loads a plugin for one session without installing it. Run `/reload-plugins` after
edits.

## Versions

`plugin.json` sets no `version`, so installs from git track the latest commit. If a version is pinned
later, users stay on it until the string changes, so bump it with every change.
