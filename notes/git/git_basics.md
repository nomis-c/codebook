# Git Basics

Task-based quick reference. `<commit>` can be any of the ways listed under [Ways to refer to a commit](#ways-to-refer-to-a-commit).

## Getting Started

```bash
git init                  # start a new repo
git clone <url>           # clone an existing repo
```

## Prepare to Commit

```bash
git add <file>            # stage an untracked file or unstaged changes
git add .                 # stage all untracked files and unstaged changes
git add -p                # choose which parts of a file to stage
git mv <old> <new>        # move/rename a file
git rm <file>             # delete a file
git rm --cached <file>    # forget a file without deleting it
git status                # check what you added
```

Unstage:

```bash
git reset <file>                # unstage one file
git restore --staged <file>     # same thing

git reset                       # unstage everything
git restore --staged .          # same thing
```

## Make Commits

```bash
git commit                # commit, opens editor for the message
git commit -m 'message'   # commit with inline message
git commit -am 'message'  # commit all changes to tracked files
```

## Move Between Branches

```bash
git switch <name>                       # switch branches
git checkout <name>                     # same thing

git switch -c <name>                    # create a branch
git checkout -b <name>                  # same thing

git branch                              # list branches
git branch --sort=-committerdate        # list by most recent commit

git branch -d <name>                    # delete a branch
git branch -D <name>                    # force delete a branch
```

## Diff Staged/Unstaged Changes

```bash
git diff HEAD             # all staged and unstaged changes
git diff --staged         # only staged changes
git diff                  # only unstaged changes
```

## Diff Commits

```bash
git show <commit>                 # diff between a commit and its parent
git diff <commit> <commit>        # diff two commits
git diff <commit> <file>          # diff one file since a commit

git diff <commit> --stat          # summary of a diff
git show <commit> --stat
```

## Ways to refer to a commit

| Reference         | Example            |
|-------------------|--------------------|
| a branch          | `main`             |
| a tag             | `v0.1`             |
| a commit ID       | `3e887ab`          |
| a remote branch   | `origin/main`      |
| current commit    | `HEAD`             |
| 3 commits ago     | `HEAD^^^` or `HEAD~3` |

## Discard Your Changes

```bash
git restore <file>                            # delete unstaged changes to one file
git checkout <file>                           # same thing

git restore --staged --worktree <file>        # delete staged + unstaged changes to one file
git checkout HEAD <file>                      # same thing

git reset --hard                              # delete all staged + unstaged changes
git restore --staged --worktree .             # same thing

git clean                                     # delete untracked files
git stash                                     # stash all staged + unstaged changes
```

## Edit History

```bash
git reset HEAD^                   # "undo" the last commit, keep working directory as is
git commit --amend                # change the last commit message (or add a forgotten file)
```

Squash the last 5 commits into one:

```bash
git rebase -i HEAD~5
# then change "pick" to "fixup" for every commit to combine with the previous one
```

Undo a failed rebase:

```bash
git reflog BRANCHNAME
# find the right commit ID in the reflog, then:
git reset --hard <commit>
```

## Code Archaeology

```bash
git log main                      # a branch's history
git log --graph main
git log --oneline

git log <file>                    # every commit that modified a file
git log --follow <file>           # ...including before it was renamed
git log -G banana                 # every commit that added or removed some text

git blame <file>                  # who last changed each line
```

## Combine Diverged Branches

```bash
# rebase
git switch banana
git rebase main

# merge
git switch main
git merge banana

# squash merge
git switch main
git merge --squash banana
git commit

# fast-forward (bring a branch up to date with another)
git switch main
git merge banana

# copy one commit onto the current branch
git cherry-pick <commit>
```

## Restore an Old File

```bash
git checkout <commit> <file>              # get a file's version from another commit
git restore <file> --source <commit>      # same thing
```

## Remotes, Push, Pull

```bash
git remote add <name> <url>               # add a remote

git push origin main                      # push main to origin
git push                                  # push current branch to its tracking branch
git push -u origin <name>                 # push a branch for the first time
git push --force-with-lease               # force push (safer than --force)
git push --tags                           # push tags

git fetch origin main                     # fetch without changing local branches
git pull --rebase                         # fetch, then rebase current branch
git pull origin main                      # fetch, then merge into current branch
git pull                                  # same, using the tracking branch
```

## Configure Git

```bash
git config user.name 'Your Name'          # set a config option
git config --global ...                   # set it globally
git config alias.st status                # add an alias
man git-config                            # all possible options
```

## Important Files

| File             | Purpose                  |
|------------------|--------------------------|
| `.git/config`    | local git config         |
| `~/.gitconfig`   | global git config        |
| `.gitignore`     | files to ignore          |