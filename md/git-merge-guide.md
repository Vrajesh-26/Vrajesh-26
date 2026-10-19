# Git Merge + PR Guide

## Basics

- A branch is a pointer to a commit. Merging combines two pointers.
- Examples below use `main` as the target and `feature` as the branch being merged.
- Read history before merging:
  ```bash
  git log --oneline --graph --all --decorate
  ```
- Merge type depends on one thing: **did the target get new commits after you branched off?**
  - No → fast-forward is possible
  - Yes → 3-way merge (merge commit) is needed
- `refs/stash` in the graph is not a branch, it's a stash. Check with `git stash list`.

---

## The 3 merge types

| Type | Result | Use when |
|---|---|---|
| **Fast-forward** | No new commit, branch pointer just moves | Target hasn't changed since you branched |
| **Merge commit** (3-way) | New commit with 2 parents, full history kept | Both branches have new commits, want history preserved |
| **Squash** | All branch commits → 1 commit on target | Messy WIP commits, want clean `main` |

### 1. Fast-forward

```bash
git switch main
git pull
git merge feature
git push
```

Before: `main → A`, `feature → A → B`
After: `main → A → B` (straight line, no merge commit)

### 2. Merge commit (3-way)

Happens automatically when `main` has moved on. Force it even when ff is possible:

```bash
git switch main
git merge --no-ff feature
git push
```

### 3. Squash

```bash
git switch main
git merge --squash feature
git commit -m "message"
git push
git branch -D feature     # -D needed, git doesn't see it as merged
```

### Conflict? 

```bash
git merge --abort        # back out, nothing changed
```
Or fix files → `git add .` → `git commit`.

---

## Pull Request (PR) flow

Same merges, but done on GitHub with review + CI (your GitHub Actions) before merging.

```bash
git push -u origin feature
```

1. GitHub → **Compare & pull request**
2. Set **base** = branch you want to merge into (`development` → `staging` → `main` if that's your flow), **compare** = `feature`
3. Wait for CI to pass, review the diff
4. Click merge, pick a type:

| GitHub button | Equivalent |
|---|---|
| Create a merge commit | `--no-ff` (always makes a merge commit, GitHub never fast-forwards) |
| Squash and merge | `--squash` |
| Rebase and merge | replays your commits on top of base, linear history, no merge commit |

5. Clean up:

```bash
git switch main
git pull
git branch -d feature
git push origin --delete feature    # or tick "delete branch" on GitHub
```

### Same thing with GitHub CLI

```bash
gh pr create --base development --fill
gh pr merge --squash --delete-branch
```

---

## Quick pick

- Solo, small change, local → **fast-forward**
- Want PR history + review trail → **PR + merge commit**
- Many "wip / fix / fix2" commits → **PR + squash**
- Always `git pull` the target branch before merging locally.
