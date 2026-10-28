# agent-skills

Generally useful agent skills in the open [Agent Skills](https://agentskills.io) format. They are
published as a [Claude Code](https://code.claude.com/docs) plugin marketplace, and work with any
agent that reads `SKILL.md` files.

## Skills

| Skill                 | What it does                                                                           |
|-----------------------|----------------------------------------------------------------------------------------|
| `csharp-coding-style` | Applies a personal C# and .NET style guide when writing, reviewing or refactoring code |
| `reviewable-commits`  | Splits work into functional commits, each reviewed and approved before it is committed |

All skills ship in one plugin, `ricflams-skills`. Plugin skills are namespaced: run one as
`/ricflams-skills:reviewable-commits`, or let Claude load it when a request matches its description.

## Layout

```
.claude-plugin/marketplace.json        the marketplace: lists the one plugin, rooted at the repo
.claude-plugin/plugin.json             the plugin
skills/<skill>/SKILL.md
```

The plugin picks up every folder under `skills/`. Keep one plugin, so skill names never change. To keep
draft skills out of it, list the shipped skills explicitly in `plugin.json`'s `skills` array.

Keep skills portable: put everything that decides when a skill triggers in `description`, since other
agents ignore Claude Code's extra frontmatter fields such as `when_to_use`.

## Install

### Claude Code

```powershell
claude plugin marketplace add ricflams/agent-skills
claude plugin install ricflams-skills@ricflams
```

From a clone, pass its path instead: `claude plugin marketplace add .\agent-skills`.

### Other agents

Codex, Gemini CLI, Copilot, Cursor and other agents that support Agent Skills can use a skill
directly: copy its folder, for example `skills/reviewable-commits`, into the
agent's skills directory. See the agent's documentation for where that is.

## Develop

From the repo root:

```powershell
claude plugin validate .
claude --plugin-dir .
```

`--plugin-dir` loads a plugin for one session without installing it. Run `/reload-plugins` after
edits.

## Versions

`plugin.json` sets no `version`, so installs from git track the latest commit. If a version is pinned
later, users stay on it until the string changes, so bump it with every change.
