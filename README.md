# claude-skills

Generally useful [Claude Code](https://code.claude.com/docs) skills, published as a plugin
marketplace.

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

## Install

From a clone:

```powershell
claude plugin marketplace add C:\my\code\claude-skills
claude plugin install dev-workflow@claude-skills
```

Once the repo is on GitHub: `claude plugin marketplace add <owner>/claude-skills`.

## Develop

```powershell
claude plugin validate C:\my\code\claude-skills
claude --plugin-dir C:\my\code\claude-skills\plugins\dev-workflow
```

`--plugin-dir` loads a plugin for one session without installing it. Run `/reload-plugins` after
edits.

## Versions

`plugin.json` sets no `version`, so installs from git track the latest commit. If a version is pinned
later, users stay on it until the string changes, so bump it with every change.
