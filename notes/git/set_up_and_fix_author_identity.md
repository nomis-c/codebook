# git config: Set Up and Fix Author Identity

## Basic setup
```bash
git config --global user.name <name>
git config --global user.email <email>
```
- Sets the identity git attaches to every commit you create
- `--global` applies it machine-wide, for every repo (stored in `~/.gitconfig`)
- Drop `--global` to set it for just the current repo instead (stored in `.git/config`, overrides the global value there)

### Verify what's currently set
```bash
git config --global user.name
git config --global user.email
# or dump everything at once:
git config --global --list
```

## Why configure this at all
- Every commit permanently records an author name + email in its metadata
- This is how GitHub/GitLab attribute a commit to your profile — avatar, profile link, contribution graph — but only if the email matches a **verified** email on your account
- Without an email match, the commit still shows the name/email you set, just not linked to any account
- Matters for collaboration too: teammates, `git log`, and `git blame` all rely on this to know who actually wrote what

## What happens if you forget: the "root" problem
- On a fresh machine/WSL install/VM/container, git has no identity configured yet
- If you commit *before* running `git config`, git falls back to the OS-level user account + hostname — often literally `root@DESKTOP-XXXX.localdomain` on a fresh WSL/Linux setup
- That identity gets baked into the commit permanently, at the moment it's created — it is not looked up again later
- So configuring the correct identity afterward only affects **future** commits — anything already committed keeps showing "root" (visible on GitHub as "root authored and committed")

### Example
```bash
git log -2 --format="%h %an <%ae> %s"
# fc5a2b8 root <root@DESKTOP-XXXX.localdomain> update
# eed930f root <root@DESKTOP-XXXX.localdomain> update README.md
```

## Fixing already-existing commits: rebase + amend

### 1. Start a rebase covering the affected commits
```bash
git rebase -i HEAD~2
```
- `HEAD~2` = rewrite the last 2 commits (adjust the number to however many are wrong)
- Opens an editor with a list like:
  ```
  pick fc5a2b8 update
  pick eed930f update README.md
  ```

### 2. Mark each commit to fix
- Change `pick` → `edit` on every line you want to correct
- Save and close — git stops at the first marked commit

### 3. Amend the author on each stopped commit
```bash
git commit --amend --author="nomis-c <simon.chen@gmx.at>" --no-edit
git rebase --continue
```
- `--no-edit` keeps the original commit message unchanged
- Repeat this pair once per commit marked `edit` (git stops again each time)

### 4. Verify locally
```bash
git log -2 --format="%h %an <%ae> %s"
```

### 5. Push the rewritten history
```bash
git push --force
# safer alternative:
git push --force-with-lease
```
- Needed because rebase changed the commit hashes — a normal push is rejected since it's no longer a fast-forward

## Flag reference

**`git rebase`** — replays a range of commits one by one onto a new base, creating a new commit (new hash) for each one instead of just moving pointers. Here it's not used to move commits onto another branch, but as the mechanism that lets you pause on and edit specific commits.

**`-i` (interactive)** — turns rebase from an automatic replay into an editable one: it opens a list of the commits in range and lets you choose an action per commit — `pick` (keep as-is), `edit` (stop here for changes), `squash`/`fixup` (merge into previous commit), `reword` (change message only), `drop` (delete). Without `-i`, `git rebase` just replays everything with no pause.

**`--amend`** (on `git commit`) — replaces the most recent commit (`HEAD`) with a modified version instead of creating a new one on top. Can change the message, the author, and/or the staged content. Always produces a new hash for that commit.

**`--force`** (on `git push`) — overwrites the remote branch to exactly match your local branch, even if that discards commits currently on the remote. Required any time local history was rewritten (rebase, amend, reset), since a normal push only allows fast-forward updates. `--force-with-lease` is the safer version — it fails instead of overwriting if the remote has commits you don't have locally, protecting against clobbering someone else's push.

## Notes
- Rebasing rewrites the hash of the amended commit **and every commit after it**, even unrelated ones
- Force-push is safe on a private/solo repo — risky on a shared branch others already pulled or branched from, since their history diverges and they'd need to re-clone or rebase
- The same `edit` + `--amend` mechanism is reused for: scrubbing accidentally-committed secrets from history (for large-scale history cleanup, use `git filter-repo` instead), reordering commits, or cleaning up messy commit messages
- Prevention: run `git config --global user.name/email` right after setting up a new environment (fresh WSL, VM, container), before making the first commit — or use `includeIf` in `.gitconfig` to auto-switch identity per directory (e.g. work vs. personal repos)