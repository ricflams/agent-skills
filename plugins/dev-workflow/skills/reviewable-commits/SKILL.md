---
name: reviewable-commits
description: Split work into reviewable, functional commits and commit them one chunk at a time with the user's approval. Use whenever the user is ready to start committing code.
when_to_use: The user signals that the work is ready to be committed, for example "let's commit", "commit this", "ready to commit", "time to commit", "commit the changes", "make the commits", "split this into commits" or "prepare the commits for the PR". Also use it when the user asks for commit messages for pending changes. Load it before running any git add or git commit for the user's work.
argument-hint: "[jira-key]"
---

# Reviewable commits

Turn a body of work into a sequence of commits that a reviewer, and the author later on, can read
one at a time, and never commit anything the user has not seen and approved.

## Principles

- **One idea per commit.** A refactor, a behaviour change, a bug fix, a docs-only change and a
  whitespace-only change are separate commits.
- **Every commit stands on its own.** It builds, its tests pass, and it carries its own tests and
  docs, so the history can be bisected.
- **Refactors before the changes that need them**, so the behaviour change stays small.
- **The user approves every commit.** Nothing is committed, pushed or rewritten without it.

## Procedure

### 1. Propose the chunks

Read the full diff (`git status`, `git diff`, untracked files) and propose an ordered list of
commits. For each one give a headline and a short list of what it contains. If a commit needs an
interim state that never existed in the working tree (for example a behaviour that a later commit
replaces), say so.

Wait for the user to agree with the plan or change it before touching anything.

#### How to cut the commits

Order the commits so each one is easy to accept on its own, and the risky change is as small as
possible when it lands:

1. **Mechanical changes:** whitespace, formatting, renames. These are reviewed by skimming. Their
   commit message states that the commit is mechanical only, with no functional changes, for
   example a final bullet `- Mechanical change only: no functional changes`.
2. **Housekeeping:** `.gitignore`, build and tooling settings.
3. **Independent corrections:** fixes or doc corrections that would be right even without this work.
4. **Preparatory refactors:** reshape the code the change needs, with no behavior change.
5. **The core behavior change in its smallest form**, even if that means an interim state that a
   later commit replaces.
6. **The main feature or contract** that builds on it.
7. **Separable behaviors on top:** additions a reviewer could accept or reject on their own.

Cut between two changes when any of these is true:

- A reviewer could approve one and question the other.
- One could be reverted without the other.
- One is mechanical and the other needs careful reading.
- They belong to different issues or stories.

Keep changes together when one is incomplete or breaks the build without the other. Tests and docs
travel with the change they cover.

*Example:* tabs → `.gitignore` → UTF-8 docs fix → extract `PartitionPaths` → regex on the old
endpoint → new POST contract → 404 for a missing partition → typed 400 for bad patterns (work for a
second issue, still under the branch's key).

### 2. Take a safety copy (when splitting uncommitted work)

Before reshaping the working tree, record the final state on a local backup branch without touching
the working tree or the real index:

```bash
export GIT_INDEX_FILE="$(mktemp)"
git add -A
tree=$(git write-tree)
commit=$(git commit-tree "$tree" -p HEAD -m "backup: final state")
git update-ref refs/heads/backup/<branch>-final "$commit"
unset GIT_INDEX_FILE
```

### 3. Build each chunk

For each commit in order:

1. Bring the working tree to that commit's state. Take final content from the backup branch
   file by file (`git show backup/…:path`), and edit only the hunks that belong to this commit when
   a file spans several commits.
2. Build and run the tests. A failing chunk is fixed before it is shown.
3. Stage exactly that commit's changes, and nothing else.
4. Show the user:
   - the staged files (`git diff --cached --stat`);
   - the proposed commit message;
   - the build and test results.
5. **Stop and wait for approval.** Apply change requests, re-stage and show the chunk again.
6. After approval, commit and move to the next chunk.

### 4. Verify the result

- `git diff backup/<branch>-final` is empty, or every difference is a change the user asked for
  during review. Report which.
- Run the repo's format or lint check if it has one.
- Delete the backup branch once the user is satisfied.

## Commit message format

```
<KEY>: <headline>

- <point>
- <point>

Co-Authored-By: <model name> <noreply@anthropic.com>
```

- **KEY prefix:** if the skill argument gives a key, use it. Otherwise look at the branch name
  (`git branch --show-current`), ignoring a leading path such as `feature/` or `bugfix/`. If it starts
  with what looks like a Jira issue id, followed by `-` or the end of the name, that id in uppercase
  is the key: `A11Y-12587-regex-search`, `feature/A11Y-12587-regex-search` and
  `a11y-12587-regex-search` all give `A11Y-12587: `. The id pattern is
  `^[A-Za-z][A-Za-z0-9]*-[0-9]+(-|$)`, matched case-insensitively. If the name doesn't start with one,
  there is no prefix: the headline starts the message. Don't ask, and don't take an id from the
  middle of the name.
- **One key for the whole branch:** every commit on the branch gets the same prefix, or none, even a
  commit that implements another issue.
- **Headline:** imperative, specific, about 72 characters at most. It says what the commit achieves,
  not which files it touches.
- **Body:** a few itemized points, short and to the point: what changed and why. Not a narrative,
  and not a file list. A mechanical commit says so explicitly: "Mechanical change only: no
  functional changes".
- **Co-author trailer:** name the model that is actually writing the commit, for example
  `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`. Never copy a model name from an earlier
  commit or from this file. If the environment gives an exact attribution line, use that line.

## Rules

- Never push, force-push, or reword or rebase pushed commits unless the user explicitly asks. If a
  request implies it, say so and ask first.
- Never skip hooks (`--no-verify`) or signing.
- Never delete untracked files with a broad loop or `git clean`. Remove only explicit paths you have
  checked. Ignored files (IDE state, `*.user`) are not backed up by step 2.
- Never stage unrelated files, generated output or secrets.
- If the user stops the process midway, leave the working tree and the backup branch as they are
  and report what is committed and what is not.
