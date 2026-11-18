# Git: Interview Study Guide

A concept-by-concept explanation of all 40 Git topics from the study list. Each entry says what it is, how it works, and what interviewers tend to probe.

## Part 1: Setup and Everyday Commands

### 1. git init

Creates a new, empty repository in the current folder by making a hidden `.git` directory that stores all history, objects and config. Run it once at the start of a project. Interview angle: know that `.git` *is* the repository; deleting it removes all history.

### 2. git clone

Copies an existing remote repository (full history, all branches) to your machine and automatically sets up `origin` as the remote. `git clone <url>` also checks out the default branch. Use `--depth 1` for a shallow clone with only the latest commit, which is faster for CI.

### 3. git status

Shows the state of your working directory: which files are untracked, modified (unstaged), or staged, and which branch you are on relative to its upstream. It is the first command to run whenever you are unsure what state you are in.

### 4. git add

Moves changes from the working directory into the **staging area** (index), choosing what goes into the next commit. `git add -p` lets you stage individual hunks, which is how you make clean, focused commits.

### 5. git commit

Saves a snapshot of the staged changes as a permanent point in history, with a message, author and a SHA-1 hash. Commits are snapshots, not diffs. Good practice: small, atomic commits with imperative messages ("Fix login redirect").

### 6. git push

Uploads your local commits to a remote branch (`git push origin main`). It is rejected if the remote has commits you lack (non-fast-forward). `-u` sets the upstream so later pushes need no arguments. `--force-with-lease` is the safer form of force-push.

### 7. git pull

Shorthand for `git fetch` followed by `git merge` (or `git rebase` with `--rebase`). It updates your current branch with remote changes. Interview favorite: explain that pull is two operations, which is why fetch is often safer.

### 8. git fetch

Downloads new commits and refs from the remote into remote-tracking branches (like `origin/main`) **without** touching your working files or local branches. Lets you inspect what changed before integrating.

## Part 2: Branching and Integrating

### 9. git branch

Lists, creates or deletes branches. A branch is just a lightweight movable pointer to a commit, which is why branching in Git is nearly free. `git branch feature-x` creates one without switching to it.

**Common types of branches.** Teams usually prefix branch names with the type of work, so purpose and ownership are clear at a glance (`type/short-description`). Branches fall into two groups: long-lived ones (`main`, `develop`) and short-lived working branches created from them and deleted after merging.

| Type | Prefix | Purpose | Branches from, merges into | Example |
| --- | --- | --- | --- | --- |
| Feature | `feature/` | New functionality or enhancement | `main` or `develop`, back to the same | `git switch -c feature/user-login` |
| Bugfix | `bugfix/` | Fix a non-urgent bug found during development or testing | `main` or `develop`, back to the same | `git switch -c bugfix/cart-total-rounding` |
| Hotfix | `hotfix/` | Urgent production fix | `main`, merged into `main` and `develop` | `git switch -c hotfix/payment-crash` |
| Release | `release/` | Stabilize a version: final testing, version bump, docs | `develop`, merged into `main` and `develop` | `git switch -c release/2.4.0` |
| Debug | `debug/` | Investigate a problem: extra logging, experiments, reproducing an issue (often discarded) | Any branch | `git switch -c debug/memory-leak-worker` |
| Security | `security/` | Patch a vulnerability or harden code (dependency upgrades, auth fixes), often reviewed privately first | `main`, merged into `main` | `git switch -c security/fix-xss-comments` |
| Refactor | `refactor/` | Restructure code without changing behavior | `main` or `develop`, back to the same | `git switch -c refactor/split-auth-module` |
| Docs | `docs/` | Documentation-only changes | `main` or `develop`, back to the same | `git switch -c docs/api-readme` |
| Test | `test/` | Add or improve tests | `main` or `develop`, back to the same | `git switch -c test/checkout-integration` |
| Chore | `chore/` | Maintenance: dependency bumps, tooling, config, cleanup | `main` or `develop`, back to the same | `git switch -c chore/upgrade-eslint` |
| Experiment | `experiment/` or `spike/` | Prototype or proof of concept; may never merge | `main` | `git switch -c experiment/graphql-gateway` |
| CI/Build | `ci/` or `build/` | Pipeline and build-system changes | `main` | `git switch -c ci/add-cache-step` |

Good practice: add a ticket ID when you have one (`feature/PROJ-123-user-login`), use lowercase words separated by hyphens, keep branches small and short-lived, and delete them after merging. Prefixes are a team convention, not something Git enforces, so follow whatever your team uses. In an interview, the key distinction to explain is that `hotfix` branches from `main` (production) while `feature` and `bugfix` branch from `develop` in Git Flow, and that trunk-based teams keep every working branch very short-lived.

### 10. git switch

The modern, single-purpose command for changing branches (`git switch main`, `git switch -c new-branch` to create and switch). Introduced to split up the overloaded `git checkout`.

### 11. git checkout

The older multi-purpose command: switches branches, checks out a specific commit (detached HEAD), or restores files. Still widely used. Know that `switch` and `restore` replaced its two main jobs.

### 12. git merge

Combines another branch's history into the current one. If histories diverged, it creates a **merge commit** with two parents; if not, it can simply fast-forward. Preserves the true history but can make the graph noisy.

### 13. git rebase

Replays your commits on top of another base, rewriting them into new commits for a linear history. `git rebase main` on a feature branch moves it to start from the latest main. Golden rule: never rebase commits that others have already pulled.

### 14. git log

Shows commit history. Useful flags: `--oneline`, `--graph`, `--decorate`, `--author`, `--since`, `-p` (show patches), `--stat`. `git log --oneline --graph --all` gives a quick visual of branching.

### 15. git diff

Shows line-level differences. `git diff` compares working tree to index, `git diff --staged` compares index to last commit, and `git diff A..B` compares two commits or branches.

### 16. git stash

Temporarily shelves uncommitted changes so you can switch context with a clean tree. `git stash push -m "msg"`, `git stash list`, `git stash pop` (apply and drop) or `apply` (keep). Add `-u` to include untracked files.

### 17. git reset

Moves the current branch pointer to another commit, optionally changing the index and working tree depending on mode (see item 38). It *rewrites* history, so use it on local, unpushed work.

### 18. git revert

Creates a **new commit** that undoes an earlier commit's changes. History is preserved, so it is the safe way to undo something already pushed to a shared branch. Contrast with reset, which rewrites history.

## Part 3: Intermediate Tools

### 19. git cherry-pick

Applies the changes of one specific commit onto your current branch as a new commit (`git cherry-pick <sha>`). Typical use: pulling a hotfix from main into a release branch. Overuse leads to duplicate commits and confusing history.

### 20. git tag

Marks a specific commit with a permanent name, usually for releases (`v1.2.0`). **Lightweight** tags are just pointers; **annotated** tags (`-a`) store tagger, date and message and are preferred for releases. Tags must be pushed explicitly (`git push --tags`).

### 21. git config

Reads and sets configuration at three levels: `--system`, `--global` (per user, `~/.gitconfig`) and `--local` (per repo, highest precedence). Common settings: `user.name`, `user.email`, `core.editor`, aliases, `pull.rebase`.

### 22. git remote

Manages named connections to other repositories. `git remote -v` lists them, `git remote add upstream <url>` adds one. The fork workflow typically uses `origin` (your fork) and `upstream` (the original).

### 23. git restore

Modern command for restoring files: `git restore file` discards working-tree changes, `git restore --staged file` unstages. Be careful: discarding uncommitted changes cannot be undone.

### 24. git rm / git mv

`git rm file` deletes a file and stages the deletion (`--cached` untracks it but keeps it on disk). `git mv a b` renames and stages in one step. Git does not actually store renames; it detects them by content similarity.

### 25. git commit --amend

Replaces the most recent commit, either to fix its message or to add forgotten staged changes. It creates a new commit hash, so avoid amending after pushing to a shared branch.

### 26. git rebase -i (interactive rebase)

Lets you rewrite a series of commits: `pick`, `reword`, `edit`, `squash`, `fixup`, `drop`, and reorder lines. Used to clean up a messy feature branch into logical commits before opening a pull request, e.g. `git rebase -i HEAD~5`.

### 27. git reflog

A local log of everywhere `HEAD` and branch tips have pointed, including commits that are no longer on any branch. It is the safety net: after a bad reset or rebase you can find the old SHA and recover with `git reset --hard <sha>`. Entries expire (about 90 days by default).

### 28. git blame

Shows who last modified each line of a file and in which commit (`git blame file`, `-L 10,20` for a range). Great for finding *why* something changed; pair it with `git show <sha>`.

### 29. git show

Displays details of any object: for a commit, its metadata plus the diff. `git show HEAD~2`, `git show v1.0`, or `git show <sha>:path/file` to view a file as it was at that commit.

### 30. git bisect

Binary-searches history to find the commit that introduced a bug. Mark a known bad and good commit (`git bisect start`, `bad`, `good <sha>`), then test each midpoint Git checks out. Finds the culprit in about log2(N) steps. `git bisect run <script>` automates it.

### 31. git clean

Deletes **untracked** files. Always preview first with `git clean -n` (dry run), then `-f` for files, `-d` for directories, `-x` to include ignored files. Destructive and unrecoverable.

### 32. git branch -d / -D

`-d` deletes a branch only if it is fully merged; `-D` forces deletion even if unmerged (work may become unreachable, though reflog can rescue it). Delete a remote branch with `git push origin --delete name`.

## Part 4: Advanced Features and Workflows

### 33. git worktree

Checks out multiple branches simultaneously in separate directories that share one repository (`git worktree add ../hotfix hotfix-branch`). Avoids stashing or re-cloning when you need to work on two branches at once.

### 34. git submodule

Embeds another repository inside yours at a pinned commit. Clone with `--recurse-submodules` or run `git submodule update --init`. Powerful for shared libraries but a frequent source of confusion, since the parent tracks a commit, not a branch.

### 35. .gitignore

A file listing patterns of files Git should not track (build output, `node_modules`, secrets, `.env`). It only affects *untracked* files; already-tracked files must be removed from the index with `git rm --cached`. Supports globs, negation (`!`) and per-directory files.

### 36. Resolving merge conflicts

Conflicts occur when both branches changed the same lines. Git marks them with `<<<<<<<`, `=======`, `>>>>>>>`. Fix: edit the file to the desired result, remove the markers, `git add` it, then `git commit` (or `git rebase --continue`). `git merge --abort` backs out. Tools like `git mergetool` help.

### 37. Fast-forward vs 3-way merge

A **fast-forward** happens when the target branch has no new commits, so Git just moves its pointer forward: no merge commit. A **3-way merge** is needed when both branches diverged; Git uses the two tips plus their common ancestor to build a merge commit. `--no-ff` forces a merge commit to preserve branch history.

### 38. Reset modes (--soft, --mixed, --hard)

- `--soft`: moves the branch pointer only; changes stay staged.
- `--mixed` (default): moves the pointer and unstages; changes stay in the working tree.
- `--hard`: moves the pointer and discards staged and working changes.

Mnemonic: soft keeps index and files, mixed keeps files only, hard keeps nothing.

### 39. Branching strategies (Git Flow, trunk-based)

**Git Flow** uses long-lived `main` and `develop` branches plus feature, release and hotfix branches: structured, suited to versioned releases, but heavy. **Trunk-based development** has everyone commit small changes to `main` (or very short-lived branches) with feature flags and strong CI: faster integration and the preference of most modern continuous-delivery teams. Also know GitHub Flow (feature branch + PR into main).

### 40. Pull requests and code review workflow

A pull request proposes merging a branch, giving a place for discussion, automated checks and approvals. Typical flow: branch, commit, push, open PR, CI runs, reviewers comment, you update, then merge (merge commit, squash or rebase). Good reviews check correctness, tests, readability and scope; good PRs are small and well described.

---

## Quick Practice Prompts

1. You committed to the wrong branch. How do you move the commit?
2. You pushed a bad commit to main. Revert or reset, and why?
3. How do you squash your last four commits into one?
4. You ran `git reset --hard` by accident. How can you recover?
5. Explain the difference between `merge` and `rebase`, and when each is appropriate.
