# Git Exercise - Process Documentation

---

## Setting Up Git

### Configure Git with username and email

```bash
git config --global user.name "ivogiake"
git config --global user.email "you@example.com"
```

---

## Git Commits to Commit

### Create work directory and hello subdirectory, create hello.sh

```bash
mkdir -p work/hello
echo 'echo "Hello, World"' > work/hello/hello.sh
```

### Initialize the git repository in the hello directory

```bash
git init
git branch -m main
```

### Check the status

```bash
git status
```

Output showed `hello.sh` as an untracked file. Staged and committed it:

```bash
git add hello.sh
git commit -m "Add initial hello.sh with Hello World output"
```

### Change hello.sh to accept a name argument

```bash
#!/bin/bash

echo "Hello, $1"
```

```bash
git add hello.sh
git commit -m "Update hello.sh to accept name argument"
```

### Add comment (first separate commit)

Added only the comment line `# Default is "World"` to hello.sh:

```bash
git add hello.sh
git commit -m "Add default comment to hello.sh"
```

### Add name variable and updated echo (second separate commit)

Added `name=${1:-"World"}` and updated `echo "Hello, $name"`:

```bash
git add hello.sh
git commit -m "Use name variable with World default in hello.sh"
```

Final hello.sh:

```bash
#!/bin/bash

# Default is "World"
name=${1:-"World"}
echo "Hello, $name"
```

---

## History

### Show the full history of the working directory

```bash
git log
```

### Show one-line history

```bash
git log --oneline
```

### Show last 2 entries

```bash
git log -2
```

### Show commits within the last 5 minutes

```bash
git log --since="5 minutes ago"
```

### Show logs in personalized format

```bash
git log --pretty=format:"* %h %ad | %s%d [%an]" --date=short
```

Example output:
```
* b1736c6 2026-05-08 | Use name variable with World default in hello.sh (HEAD -> main) [ivogiake]
```

---

## Check It Out

### Restore first snapshot (initial commit) and print hello.sh

```bash
git checkout 645d052
cat hello.sh
```

Output:
```
echo "Hello, World"
```

### Restore second most recent snapshot and print hello.sh

```bash
git checkout 6dc6caa
cat hello.sh
```

Output:
```
#!/bin/bash

echo "Hello, $1"
```

### Return to the latest version on main branch

```bash
git checkout main
```

---

## TAG Me

### Tag current version as v1

```bash
git tag v1
```

### Tag the previous version as v1-beta (without commit hash)

```bash
git checkout v1^
git tag v1-beta
git checkout main
```

### Navigate between tagged versions

```bash
git checkout v1-beta
git checkout v1
git checkout main
```

### List all tags

```bash
git tag
```

Output:
```
v1
v1-beta
```

---

## Changed Your Mind?

### Revert uncommitted bad comment

Added unwanted comment, then reverted before staging:

```bash
git checkout -- hello.sh
```

### Stage and clean (unstage)

Added unwanted staged comment, then cleaned:

```bash
git add hello.sh
git reset HEAD hello.sh
git checkout -- hello.sh
```

### Commit and revert

Added unwanted comment, committed it, then reverted:

```bash
git add hello.sh
git commit -m "Add unwanted committed change"
git revert HEAD --no-edit
```

### Tag as oops then remove commits after v1

```bash
git tag oops
git reset --hard v1
```

### Show logs with deleted commits (oops tag visible)

```bash
git log --all --oneline --decorate
```

Output showed:
```
5fb4142 (tag: oops) Revert "Add unwanted committed change"
3797108 Add unwanted committed change
b1736c6 (HEAD -> main, tag: v1) Use name variable with World default in hello.sh
...
```

### Clean unreferenced commits

```bash
git tag -d oops
git gc --prune=now --aggressive
```

After cleanup, `git log --all --oneline --decorate` no longer shows the oops commits.

### Add author comment and commit

```bash
git add hello.sh
git commit -m "Add author comment to hello.sh"
```

### Amend last commit to include email (without new commit)

Added email line to hello.sh, then amended:

```bash
git add hello.sh
git commit --amend --no-edit
```

---

## Move It

### Move hello.sh into lib/ directory using Git

```bash
git mv hello.sh lib/hello.sh
git commit -m "Move hello.sh into lib/ directory"
```

### Create Makefile and commit

```
TARGET="lib/hello.sh"

run:
	bash ${TARGET}
```

```bash
git add Makefile
git commit -m "Add Makefile to run lib/hello.sh"
```

---

## Blobs, Trees and Commits

### Exploring the .git/ directory

```bash
ls .git/
```

| Entry | Purpose |
|---|---|
| `objects/` | Stores all git objects (blobs, trees, commits, tags) as compressed files |
| `refs/` | Contains pointers to commit objects — branches (`refs/heads/`), tags (`refs/tags/`), and remotes (`refs/remotes/`) |
| `config` | Repository-level configuration (remotes, branches, user settings) |
| `HEAD` | Points to the currently checked-out branch or commit |
| `index` | The staging area — a binary file tracking what will go into the next commit |
| `logs/` | History of ref updates (used by `git reflog`) |
| `hooks/` | Scripts that can be triggered on git events (pre-commit, post-merge, etc.) |
| `COMMIT_EDITMSG` | Stores the message from the most recent commit |
| `packed-refs` | Packed representation of refs for performance |
| `description` | Used by git web interfaces |
| `info/` | Repository-specific exclude patterns |

### Find latest object hash and print its type and content

```bash
git log --oneline -1
# e.g. df79e58
git cat-file -t df79e58
git cat-file -p df79e58
```

Output (type: `commit`):
```
tree 4d9fa5e0ad5d8e44bdffce6e6d93627129f721fb
parent 2b3e6eb5fed64e1517bbe797ab18a6b72e87b8d0
author ivogiake <you@example.com> ...
committer ivogiake <you@example.com> ...

Add Makefile to run lib/hello.sh
```

### Dump the directory tree referenced by the commit

```bash
git cat-file -p 4d9fa5e0ad5d8e44bdffce6e6d93627129f721fb
```

Output:
```
100644 blob 407082da...  Makefile
040000 tree e29e7058...  lib
```

### Dump lib/ directory and hello.sh content

```bash
git cat-file -p e29e7058b862b39304b7902cbef4c37d4b19696d
# Output: 100644 blob d359f1b1...  hello.sh

git cat-file -p d359f1b117aae3a6a09072dd5db39f4b1d42ef0b
```

---

## Branching

### Create and switch to greet branch

```bash
git checkout -b greet
```

### Create lib/greeter.sh and commit

```bash
git add lib/greeter.sh
git commit -m "Add greeter.sh with Greeter function"
```

### Update lib/hello.sh to use Greeter and commit

```bash
git add lib/hello.sh
git commit -m "Update hello.sh to use Greeter function"
```

### Update Makefile with comment and commit

```bash
git add Makefile
git commit -m "Add comment to Makefile for updated hello.sh"
```

### Switch to main, compare branches

```bash
git checkout main
git diff main greet -- Makefile
git diff main greet -- lib/hello.sh
git diff main greet -- lib/greeter.sh
```

### Generate README.md on main and commit

```bash
git add README.md
git commit -m "Add README.md for the project"
```

### Draw commit tree diagram

```bash
git log --all --oneline --graph --decorate
```

Output:
```
* b49aee5 (HEAD -> main) Add README.md for the project
| * 86c5cb3 (greet) Add comment to Makefile for updated hello.sh
| * 0ba6b6a Update hello.sh to use Greeter function
| * 4b6f035 Add greeter.sh with Greeter function
|/
* df79e58 Add Makefile to run lib/hello.sh
...
```

---

## Conflicts, Merging and Rebasing

### Merge main into greet branch

```bash
git checkout greet
git merge main -m "Merge main into greet"
```

### Switch to main and make changes to hello.sh file

```bash
git checkout main
# Edited hello.sh to prompt for user input
git add lib/hello.sh
git commit -m "Update hello.sh to prompt for user name"
```

### Merging main into greet branch (Conflict)

```bash
git checkout greet
git merge main
```

Result: `CONFLICT (content): Merge conflict in lib/hello.sh`

### Resolve the conflict (manually or using merge tools)

Opened `lib/hello.sh`, removed conflict markers, accepted the changes from main:

```bash
# Edited lib/hello.sh manually to keep main's version
git add lib/hello.sh
```

### After resolving, stage the changes and commit

```bash
git commit -m "Resolve merge conflict: accept main version of hello.sh"
```

### Rebasing greet branch

First reset greet to the point before the initial merge:

```bash
git reset --hard 86c5cb3
git rebase main
```

A conflict arose during rebase; resolved it by keeping the greet version of hello.sh, then:

```bash
git add lib/hello.sh
git rebase --continue
```

### Merging greet into main

```bash
git checkout main
git merge greet -m "Merge greet branch into main"
```

This was a **fast-forward** merge because after rebasing, greet was directly ahead of main with no diverging commits.

### Fast-forwarding and differences between merging and rebasing

**Fast-forwarding**: When the target branch has no new commits since the source branch diverged, git simply moves the branch pointer forward to the tip of the source branch. No merge commit is created. This happened when merging greet into main after the rebase.

**Merging vs Rebasing**:

| | Merge | Rebase |
|---|---|---|
| History | Preserves full history with a merge commit | Rewrites history — replays commits on top of target |
| Commit graph | Non-linear (shows divergence) | Linear (appears as if no branching occurred) |
| Safety | Safe on shared branches | Avoid on public/shared branches (rewrites history) |
| Use case | Integrating completed features | Keeping a clean, linear history |

---

## Local and Remote Repositories

### Clone hello as cloned_hello (no copy command)

```bash
git clone hello cloned_hello
```

### Show logs in cloned_hello

```bash
git -C cloned_hello log --oneline
```

### Display remote name and more information

```bash
git -C cloned_hello remote
git -C cloned_hello remote -v
git -C cloned_hello remote show origin
```

### List all remote and local branches in cloned_hello

```bash
git -C cloned_hello branch -a
```

### Update README.md in original and commit

```bash
# Edited README.md in hello/
git add README.md
git commit -m "Update README.md - changed in original"
```

### Fetch changes from remote in cloned_hello and display logs

```bash
git -C cloned_hello fetch
git -C cloned_hello log --all --oneline --decorate
```

### Merge remote main into local main

```bash
git -C cloned_hello merge origin/main
```

The single command equivalent to `fetch` + `merge origin/main` is:

```bash
git pull
```

### Add local greet branch tracking remote origin/greet

```bash
git -C cloned_hello branch --track greet origin/greet
```

### Add remote and push main and greet branches

```bash
git remote add origin https://platform.zone01.gr/git/ivogiake/git
git push -u origin main
git push -u origin greet
```

---

## Bare Repositories

### What is a bare repository and why is it needed?

A **bare repository** contains only the git data (objects, refs) with no working tree. It has no checked-out files. Bare repositories are used as **shared/central remotes** — they accept pushes from multiple contributors without the risk of conflicting with a local working tree. Platforms like GitHub and Gitea host bare repositories.

### Create a bare repository hello.git from hello

```bash
git clone --bare hello hello.git
```

### Add bare hello.git as a remote named 'shared' to hello

```bash
git remote add shared /path/to/work/hello.git
```

### Update README.md and commit

```bash
# Edited README.md
git add README.md
git commit -m "Update README.md - pushed to shared"
```

### Push to shared repository

```bash
git push shared main
```

### Pull changes in cloned_hello from the shared repository

```bash
git -C cloned_hello remote add shared /path/to/work/hello.git
git -C cloned_hello pull shared main
```
