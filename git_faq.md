# Table of Contents
- [Git: Force Push After Local Reset](#git-force-push-after-local-reset)
- [Git Reset: Understanding `--hard` vs. `--soft`](#git-reset-understanding---hard-vs---soft)
- [Git Branch Merging Scenarios](#git-branch-merging-scenarios)
   - [Scenario 1: Fast-Forward Merge](#scenario-1-fast-forward-merge)
   - [Scenario 2: Three-Way Merge (Recursive Merge)](#scenario-2-three-way-merge-recursive-merge)
   - [Scenario 3: Squash Merge](#scenario-3-squash-merge)
   - [Scenario 4: Rebasing followed by Fast-Forward Merge](#scenario-4-rebasing-followed-by-fast-forward-merge)
---


# Git: Force Push After Local Reset

**Scenario:** You reset your local branch (`feat_borhan_dcama_pipeline`) back 5 commits, but those 5 commits were already pushed to the remote (`origin`). `git status` shows you are "behind" because the remote branch still includes those commits.

**Problem:** You want the remote branch to also go back 5 commits, effectively removing the pushed history. A normal `git push` fails because it's not a fast-forward.

**Solution:** You must **force push** to update the remote branch to match your local branch's older state.

```bash
git push origin feat_borhan_dcama_pipeline --force-with-lease
```

**⚠️ WARNING:** Force pushing rewrites the remote history. This **will cause problems** for anyone else who has pulled those commits. Use with extreme caution, preferably only on your own branches or after coordinating with your team. `--force-with-lease` is slightly safer than `--force`.

---

# Git Reset: Understanding --hard vs. --soft

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

# Git Branch Merging Scenarios

Merging branches is a fundamental operation in Git, combining the history and changes from one branch into another. The outcome of a merge depends on the history of the branches involved and the strategy used.

Here are common scenarios and the resulting history:

## Scenario 1: Fast-Forward Merge

This happens when the target branch (e.g., `main`) has not diverged from the source branch (e.g., `feature`). The entire history of the source branch is a direct continuation of the target branch's history.

* **Setup:**

    * Branch `main` is at commit `C`.

    * Branch `feature` was created from `main` at commit `C`.

    * Commits `D` and `E` were added only to `feature`.

    * No new commits have been added to `main` since `C`.

    ```
    A -- B -- C (main)
             \
              D -- E (feature)

    ```

* **Action:** Merge `feature` into `main`.

    ```
    # On main branch
    git merge feature

    ```

* **Outcome:** Git simply moves the `main` branch pointer forward to the tip of the `feature` branch (`E`). No new merge commit is created. The history remains linear.

    ```
    A -- B -- C -- D -- E (main, feature)

    ```

* **When it happens:** When the branch you are merging *into* has not had any new commits added since the branch you are merging *from* was created or last updated from the target.

* **Pros:** Simple, linear history.

* **Cons:** Doesn't explicitly show where the feature branch work occurred as a separate line of development.

## Scenario 2: Three-Way Merge (Recursive Merge)

This is the most common scenario when both branches have diverged, meaning new commits have been added to both the target branch and the source branch independently since the source branch was created or last updated. Git needs a common ancestor commit (the "base") and the tips of both branches to figure out how to combine the changes.

* **Setup:**

    * Branch `main` is at commit `C`.

    * Branch `feature` was created from `main` at commit `C`.

    * Commits `D` and `E` were added to `feature`.

    * Commits `F` and `G` were added to `main`.

    ```
    A -- B -- C -- F -- G (main)
             \
              D -- E (feature)

    ```

    The common ancestor (base) is commit `C`.

* **Action:** Merge `feature` into `main`.

    ```
    # On main branch
    git merge feature

    ```

* **Outcome:** Git performs a three-way merge, combining the changes from `E` (relative to `C`) and `G` (relative to `C`). A new **merge commit** (`M`) is created on the `main` branch. This merge commit has two parent commits: the tip of `main` (`G`) and the tip of `feature` (`E`).

    ```
    A -- B -- C -- F -- G -- M (main)
             \             /
              D -- E ----- (feature)

    ```

* **When it happens:** When both branches have unique commits since their last common ancestor.

* **Pros:** Preserves the history of both branches, clearly showing where the merge occurred and the parallel development.

* **Cons:** Can result in a more complex, non-linear history graph ("diamond shape"). May require resolving merge conflicts if the same parts of files were changed differently in both branches.

## Scenario 3: Squash Merge

Squash merging takes all the commits from the source branch and combines them into a *single* new commit on the target branch. The individual commits from the source branch are not preserved in the target branch's history.

* **Setup:** Same as the Three-Way Merge scenario:

    ```
    A -- B -- C -- F -- G (main)
             \
              D -- E (feature)

    ```

* **Action:** Squash merge `feature` into `main`.

    ```
    # On main branch
    git merge --squash feature
    # Git applies the changes, but doesn't commit. You need to commit manually.
    git commit -m "Squashed feature branch work"

    ```

* **Outcome:** A single new commit (`S`) is created on `main`. This commit contains all the changes from commits `D` and `E` combined. The history of `feature` (commits `D` and `E`) is not included in `main`'s history.

    ```
    A -- B -- C -- F -- G -- S (main)
             \
              D -- E (feature - history not on main)

    ```

* **When it happens:** Often used when merging feature branches that have many small or messy commits, and you want to keep the target branch's history clean and concise.

* **Pros:** Keeps the target branch history clean and linear. Useful for merging feature branches with many intermediate commits.

* **Cons:** Loses the individual commit history from the source branch. Can make debugging or reverting specific changes from the original feature branch harder.

## Scenario 4: Rebasing followed by Fast-Forward Merge

Rebasing rewrites the commit history of the source branch (`feature`) so that it appears to branch off the *current* tip of the target branch (`main`), rather than its original base. After rebasing, a fast-forward merge is usually possible.

* **Setup:** Same as the Three-Way Merge scenario:

    ```
    A -- B -- C -- F -- G (main)
             \
              D -- E (feature)

    ```

* **Action:** Rebase `feature` onto `main`, then merge `feature` into `main`.

    ```bash
    # On feature branch
    git rebase main
    # ... Git reapplies D and E on top of G, creating new commits D' and E' ...

    # Now switch back to main
    git checkout main

    # Merge feature (which is now ahead of main)
    git merge feature

    ```

* **Outcome:** The `feature` branch history is rewritten. New commits (`D'` and `E'`) are created with the same changes as `D` and `E` but with `G` as their parent. The `main` branch is then fast-forwarded to `E'`.

    ```
    A -- B -- C -- F -- G -- D' -- E' (main, feature)

    ```

* **When it happens:** Used to create a linear history, making the feature branch appear as if it was developed *after* the latest changes on the target branch.

* **Pros:** Creates a clean, linear history. Can make the project history easier to read.

* **Cons:** **Rewrites history!** Never rebase commits that have already been pushed to a shared remote repository unless you are prepared for the consequences (requires force pushing and will cause issues for collaborators). Can involve resolving conflicts during the rebase process.

Choosing the right merge strategy depends on your team's workflow, the desired history structure, and whether the commits have been shared.

