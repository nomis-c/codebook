# git cherry-pick: Apply a Specific Commit to Current Branch

## Basic usage
```bash
git cherry-pick <commit-hash>
```
- Applies changes from a specific commit onto your current branch
- Commit hash can be found with `git log`

### Example
```bash
# fixed a bug on main, but also need it on your dev branch
git checkout dev
git cherry-pick a1b2c3d

# Before: dev branch is missing the bugfix
# After:  dev branch has the bugfix applied as a new commit
```

## Cherry-pick multiple commits
```bash
git cherry-pick a1b2c3d e4f5g6h
```

## Cherry-pick a range of commits
```bash
git cherry-pick a1b2c3d..e4f5g6h
```
- Applies all commits between the two hashes (exclusive of first, inclusive of last)

## Apply without committing immediately
```bash
git cherry-pick --no-commit <commit-hash>
```
- Applies changes to working directory but lets you review before committing

## If conflicts occur
```bash
# After resolving conflicts manually:
git cherry-pick --continue

# Or abort entirely:
git cherry-pick --abort
```

# Notes
- cherry-pick creates a **new commit** with a different hash — it's a copy, not a move
- Use `git log --oneline` to find commit hashes quickly
- Best for picking isolated, self-contained commits — messy if the commit depends on others
- Order of commits in the command matters — they are applied left to right, not chronologically
  - `git cherry-pick a1b2c3d e4f5g6h` applies `a1b2c3d` first, then `e4f5g6h` regardless of original order