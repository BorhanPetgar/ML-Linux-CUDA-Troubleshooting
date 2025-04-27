# Git: Force Push After Local Reset

**Scenario:** You reset your local branch (`feat_borhan_dcama_pipeline`) back 5 commits, but those 5 commits were already pushed to the remote (`origin`). `git status` shows you are "behind" because the remote branch still includes those commits.

**Problem:** You want the remote branch to also go back 5 commits, effectively removing the pushed history. A normal `git push` fails because it's not a fast-forward.

**Solution:** You must **force push** to update the remote branch to match your local branch's older state.

```bash
git push origin feat_borhan_dcama_pipeline --force-with-lease
```

**⚠️ WARNING:** Force pushing rewrites the remote history. This **will cause problems** for anyone else who has pulled those commits. Use with extreme caution, preferably only on your own branches or after coordinating with your team. `--force-with-lease` is slightly safer than `--force`.

---

# Git Reset: Understanding `--hard` vs. `--soft`

The `git reset <commit>` command is used to move your branch's HEAD pointer to a different commit. The key difference between the `--hard` and `--soft` options lies in how they affect your **staging area** (index) and your **working directory** after moving the HEAD.

Let's assume you are currently on commit `E` and you run `git reset <Commit C>` where `C` is an earlier commit in your history: `A -- B -- C -- D -- E`.

## `git reset --hard <commit>`

* **What it does:**
    * Moves the branch pointer (and HEAD) to the specified commit (`C`).
    * Resets the **staging area** (index) to match the specified commit (`C`).
    * Resets the **working directory** to match the specified commit (`C`).
* **Result:** Your branch's history is rewritten back to commit `C`. All changes introduced in commits `D` and `E`, as well as any uncommitted changes in your working directory or staging area, are **permanently discarded**.
* **Analogy:** It's like traveling back in time and completely erasing everything that happened after that point, both in history and your current workspace.
* **Use Case:** When you want to completely discard the last few commits and *all* subsequent changes in your working directory. Be very careful with this command, especially if you have uncommitted work you wanted to keep.

```bash
# Moves HEAD and branch to <commit-hash>, discards index and working directory changes
git reset --hard <commit-hash>

# Example: Discard the last 2 commits and any uncommitted changes
git reset --hard HEAD~2
```

## `git reset --soft <commit>`

* **What it does:**
    * Moves the branch pointer (and HEAD) to the specified commit (`C`).
    * **Does NOT touch** the **staging area** (index). It keeps the index as it was *before* the reset.
    * **Does NOT touch** the **working directory**. Your files remain exactly as they were.
* **Result:** Your branch's history is rewritten back to commit `C`. The changes that were introduced in commits `D` and `E` (and any existing uncommitted changes) are now sitting in your staging area, ready to be committed again as a single new commit.
* **Analogy:** It's like traveling back in time *only* in terms of history. Your current work-in-progress and staged changes remain exactly as they are, but Git now sees them as changes *relative to the older commit C*.
* **Use Case:** When you want to undo the last few commits but keep all the changes from those commits (and any uncommitted work) in your staging area so you can re-commit them, perhaps as a single, squashed commit. Useful for reshaping history.

```bash
# Moves HEAD and branch to <commit-hash>, keeps index and working directory
git reset --soft <commit-hash>

# Example: Undo the last 2 commits, but keep their changes staged
git reset --soft HEAD~2
```

## Key Differences Summarized

| Feature          | `git reset --hard <commit>`                 | `git reset --soft <commit>`                   |
| :--------------- | :------------------------------------------ | :-------------------------------------------- |
| **HEAD/Branch** | Moves to `<commit>`                         | Moves to `<commit>`                           |
| **Staging Area** | Resets to match `<commit>`                  | **Kept as is** (changes from undone commits staged) |
| **Working Dir** | Resets to match `<commit>` (discards changes) | **Kept as is** (changes remain in files)      |
| **Changes** | Discarded                                   | Kept, staged for commit                     |
| **Safety** | Less safe (destructive)                     | More safe (preserves changes)                 |

## When to Use Which

* Use `git reset --hard` when you want to **completely throw away** the last few commits and any uncommitted work, returning your branch and workspace to a clean state at a previous commit.
* Use `git reset --soft` when you want to **undo the last few commits in history**, but **keep all the changes** from those commits (plus any uncommitted work) staged so you can make a new commit incorporating all those changes. This is common for squashing commits.

Always be mindful of using `git reset` on commits that have already been pushed, as it rewrites history and can complicate collaboration, often requiring a force push (as discussed previously).
