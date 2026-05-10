# git stash: Temporarily Save Uncommitted Changes

## Save current changes
```bash
git stash
```
- Saves all uncommitted changes and reverts to last commit
- Useful when you need to switch context without committing

## List all stashes
```bash
git stash list
```
- Output: `stash@{0}: WIP on main: abc1234 your last commit message`

## Apply most recent stash
```bash
git stash pop
```
- Restores changes and removes stash from list

## Apply without removing from list
```bash
git stash apply
```
- Restores changes but keeps stash saved

## Apply a specific stash
```bash
git stash pop stash@{2}
```

## Delete a stash
```bash
git stash drop stash@{0}
```

## Clear all stashes
```bash
git stash clear
```

## Notes
- `pop` = apply + delete from list → use when you're done with the stash
- `apply` = apply + keep in list → use when you might need it again
- Untracked files (new files) are not stashed by default, add `-u` to include them:
```bash
git stash -u
```
- Stashes are numbered `stash@{0}` to `stash@{n}` — `{0}` is always the most recent
- Numbering shifts automatically when you add/remove stashes
- Stashes persist as long as the repository exists — no automatic expiry
- stashes are local only