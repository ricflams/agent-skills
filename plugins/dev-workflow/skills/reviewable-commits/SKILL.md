---
name: reviewable-commits
description: Split work into reviewable, functional commits and commit them one at a time, stopping after each staged chunk until the user has reviewed and approved it. Use whenever the user is ready to start committing code.
when_to_use: The user signals that the work is ready to be committed, for example "let's commit", "commit this", "ready to commit", "time to commit", "commit the changes", "make the commits", "split this into commits" or "prepare the commits for the PR". Also use it when the user asks for commit messages for pending changes. Load it before running any git add or git commit for the user's work.
argument-hint: "[jira-key]"
---

# Reviewable commits

Turn a body of work into a sequence of commits that a reviewer, and the author later on, can read
one at a time, and never commit anything the user has not seen and approved.

## Principles

- **One idea per commit.** A refactor, a behavior change, a bug fix, a docs-only change and a
  whitespace-only change are separate commits.
- **Every commit stands on its own as a reviewable unit**, and carries its own tests and docs. A
  commit doesn't have to build or pass the tests: grouping by functionality comes first. Don't
  move changes between chunks or invent interim states just to keep every commit buildable.
- **Refactors before the changes that need them**, so the behavior change stays small.
- **The user reviews and approves every commit, one at a time.** Nothing is committed, pushed or
  rewritten without it. Agreeing to the plan, choosing how to split, or saying "commit" never
  approves a particular commit. Only a "yes" to a staged chunk the user has just been shown does.

## Procedure

### 1. Propose the chunks

Read the full diff (`git status`, `git diff`, untracked files) and propose an ordered list of
chunks. For each one give:

- a headline and a short list of what it contains;
- a ballpark **T-shirt size** for the review effort: XS (a few lines), S (a small, focused change),
  M (a few files), L (many files or a substantial change), XL (a large part of the work). This is
  a rough guide, not a line count;
- its **dependencies**: the earlier chunks it builds on, or "independent";
- a note when it needs an interim state that never existed in the working tree, for example a
  behavior that a later chunk replaces.

End the list with a **misc chunk** for leftovers that don't fit well in any functional chunk, such
as stray tweaks or work in progress. Don't force them into a functional chunk. The user may leave
the misc chunk uncommitted and carry on working on it, so no other chunk may depend on it.

Then ask the user explicitly whether the order is reasonable or should change, and wait. Their
agreement approves the plan only, not committing any chunk.

#### How to cut and order the chunks

Order the chunks so each one is easy to accept on its own, and the risky change is as small as
possible when it lands. A dependent chunk comes after the chunks it builds on. An independent chunk
can go anywhere, so place it where it reads best.

1. **Mechanical changes:** whitespace, formatting, renames, moves. These are reviewed by skimming,
   so get them out of the way first when they are independent and not tied to one topic. A
   mechanical change that belongs to a topic goes with that topic's chunks instead: if chunks 4–6
   deal with topic X, a rename related to X sits alongside them, not first. Either way, the commit
   message states that the commit is mechanical only, with no functional changes, for example a
   final bullet `- Mechanical change only: no functional changes`.
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

Keep changes together when one makes no sense without the other. Tests and docs
travel with the change they cover.

*Example:* tabs → `.gitignore` → UTF-8 docs fix → extract `PartitionPaths` → regex on the old
endpoint → new POST contract → 404 for a missing partition → typed 400 for bad patterns (work for a
second issue, still under the branch's key).

### 2. Take a safety copy (when splitting uncommitted work)

Skip this when the work goes into a single commit as it stands. Otherwise, before reshaping the
working tree, record the final state on a local backup branch without touching the working tree or
the real index. Run it from the repo root, and replace `<branch>` with the current branch name.

Start the temporary index from a copy of the real index, not an empty or not-yet-existing path:
`git add -A` against an empty index has no stat-cache entries to compare against, so it re-reads and
re-hashes every file in the working tree, not just the changed ones. With `core.autocrlf=true` that
re-normalizes any file whose stored blob has different line endings than the filter would currently
produce, pulling unrelated files (for example a `.sln`, or YAML checked in with real CRLF) into the
backup commit.

Bash:

```bash
gitdir="$(git rev-parse --absolute-git-dir)"
cp "$gitdir/index" "$gitdir/backup.index"
export GIT_INDEX_FILE="$gitdir/backup.index"
git add -A
tree=$(git write-tree)
commit=$(git commit-tree "$tree" -p HEAD -m "backup: final state")
git update-ref refs/heads/backup/<branch>-final "$commit"
rm "$GIT_INDEX_FILE"; unset GIT_INDEX_FILE
```

PowerShell:

```powershell
$gitdir = git rev-parse --absolute-git-dir
Copy-Item (Join-Path $gitdir "index") (Join-Path $gitdir "backup.index")
$env:GIT_INDEX_FILE = Join-Path $gitdir "backup.index"
git add -A
$tree = git write-tree
$commit = git commit-tree $tree -p HEAD -m "backup: final state"
git update-ref refs/heads/backup/<branch>-final $commit
Remove-Item $env:GIT_INDEX_FILE; Remove-Item Env:GIT_INDEX_FILE
```

Check the result with `git show --stat backup/<branch>-final` before going on.

### 3. Build each chunk

For each commit in order, and **one commit per turn**. Never script, batch or loop several commits,
even when every intermediate state has been verified.

The user may stage and commit some chunks themselves, often large ones, to get them out of the way
early. So before each chunk, check `git status` and `git log` and adjust the remaining plan to what
is actually left, instead of assuming the earlier plan still holds. Then:

1. Bring the working tree to that commit's state. When there is a backup branch, take final content
   from it file by file (`git show backup/…:path`), and edit only the hunks that belong to this
   commit when a file spans several commits.
2. Build and run the tests when that's quick. A chunk that doesn't build or pass is still fine to
   commit; report it rather than fixing it with changes from another chunk.
3. Stage exactly that commit's changes, and nothing else.
4. Show the user:
   - the staged files (`git diff --cached --stat`);
   - the proposed commit message;
   - the build and test results, if they were run.
5. **Stop and end your turn.** Wait for the user's explicit approval of this chunk. Apply change
   requests, re-stage and show the chunk again.
6. After approval, commit and move to the next chunk.

When only the misc chunk is left, ask whether to commit it or leave it uncommitted to keep working
on.

### 4. Verify the result

- When there is a backup branch: `git diff backup/<branch>-final` is empty, or every difference is a
  change the user asked for during review. Report which. A misc chunk left uncommitted is still in
  the working tree, so compare the working tree, not only `HEAD`.
- Run the repo's format or lint check if it has one.
- Delete the backup branch, if any, once the user is satisfied.

## Commit message format

```
<KEY>: [<Type>: ]<headline>

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
- **Type marker:** when a commit is entirely one specific kind of change, start the headline with
  its type, capitalized, after the key: `A11Y-12611: Fix: Offer pdftitle with string operators again`.
  The types are those of [Conventional Commits](https://www.conventionalcommits.org), which is the
  intention behind them, but only the subset that names a specific kind of change:
  - `Fix`: corrects existing behavior, not the behavior the branch is adding
  - `Refactor`: restructures code, including renames, with no change in behavior
  - `Style`: formatting and whitespace only
  - `Docs`: documentation only
  - `Test`: tests only
  - `Chore`, `Build`, `CI`: housekeeping, build or pipeline only
  - `Performance`: makes something faster with no other change in behavior (the convention's `perf`)

  There is no `Feat`: a commit without a marker is the branch's own work, which keeps the marker
  meaningful. The marker says what kind of change the commit is, and so what to check, even when it
  serves the feature: a preparatory refactor is still `Refactor`, and a commit of only the feature's
  design docs is still `Docs`. Use one marker at most; a commit that needs two should be split.
  `Refactor` and `Style` commits also end with the "Mechanical change only" bullet below.
- **Headline:** imperative, specific, and **72 characters at most for the whole first line**,
  counting the key and the type marker. This is a hard limit: GitHub cuts a longer subject off with
  "…" and moves the rest into the description, and does the same to a PR title made from the commit.
  Count the length (for example `${#headline}` in Bash) rather than estimating it, and
  shorten the wording or move detail into the body when it's over. The headline says what the commit
  achieves, not which files it touches.
- **Body:** a few itemized points, short and to the point: what changed and why. Not a narrative,
  and not a file list. A mechanical commit says so explicitly: "Mechanical change only: no
  functional changes".
- **Co-author trailer:** name the model that is actually writing the commit, for example
  `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`. Never copy a model name from an earlier
  commit or from this file. If the environment gives an exact attribution line, use that line.

## Pull request title

A PR's title carries the same key as the branch's commits: `<KEY>: <headline>`, for example
`A11Y-12611: Add the PdfText rule`, never a title GitHub derives from the branch name, such as
"A11y 12611 add pdftext rule". With no key, the headline stands alone.

- The headline says what the whole PR achieves, in the style of a commit headline. It has no type
  marker, unless every commit in the PR has the same one.
- The whole title, key included, is 72 characters at most, as for a commit headline. Count it.
- When opening a PR, always pass the title explicitly (`gh pr create --title "<KEY>: …"`): for a PR
  with several commits, GitHub otherwise derives it from the branch name.
- If an existing PR's title doesn't follow this, propose the corrected title, and change it
  (`gh pr edit <number> --title …`) only after the user agrees, since it is visible to others.

## Rules

- Never push, force-push, or reword or rebase pushed commits unless the user explicitly asks. If a
  request implies it, say so and ask first.
- Never skip hooks (`--no-verify`) or signing.
- Never delete untracked files with a broad loop or `git clean`. Remove only explicit paths you have
  checked. Ignored files (IDE state, `*.user`) are not backed up by step 2.
- Never stage unrelated files, generated output or secrets.
- If the user stops the process midway, leave the working tree and the backup branch as they are
  and report what is committed and what is not.
