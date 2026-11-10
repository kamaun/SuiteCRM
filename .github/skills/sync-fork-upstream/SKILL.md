---
name: sync-fork-upstream
description: 'Sync the kamaun/SuiteCRM fork with the canonical SuiteCRM/SuiteCRM upstream repository. Use when asked to update, sync, or bring this fork up to date with upstream.'
---

# Sync the SuiteCRM Fork with Upstream

Safely bring the `kamaun/SuiteCRM` fork up to date with `SuiteCRM/SuiteCRM`, preserving fork-specific commits and local work.

## Procedure

1. **Inspect before changing anything.**
   - Check `git status --short --branch` and the configured remotes.
   - Confirm `upstream` points to `https://github.com/SuiteCRM/SuiteCRM.git` and `origin` points to the user's fork (`https://github.com/kamaun/SuiteCRM.git`). If either is missing or points elsewhere, stop and ask before changing remote configuration.
   - Record the current branch and check for an in-progress merge, rebase, or other Git operation. If one exists, stop and ask the user to finish or abort it.
2. **Determine the target branch.**
   - Find the upstream default branch from `git ls-remote --symref upstream HEAD` (or repository metadata if that is unavailable); do not assume it is `master` or `main`.
   - By default, sync the current branch. If the user names another target, use that branch instead. Stop and ask if HEAD is detached.
   - Check whether the target exists on `origin`; report if it does not. If the current branch is not the target, ask before switching.
3. **Fetch and review.**
   - Fetch upstream: `git fetch upstream`.
   - Inspect the target branch's relationship to `upstream/<default-branch>` and review the incoming commits and diff. Do not discard fork-specific commits.
4. **Stash local changes and integrate.**
   - If there are staged, unstaged, or untracked changes, run `git stash push --include-untracked` with a descriptive message before switching branches or merging. Record the new stash's commit ID and original branch; verify it was created and the worktree is clean before continuing. Do not include ignored files or disturb existing stashes.
   - Switch to the target branch if needed.
   - Merge `upstream/<default-branch>` into the target branch to preserve existing branch history. Do not use `reset --hard`, force-push, or rebase a published fork branch.
   - If there are conflicts, stop and explain the conflicted paths. Resolve only when the intended behavior is clear; otherwise ask the user. Do not silently drop either side's changes. Keep the stash intact until the merge is completed or aborted.
5. **Restore local changes.**
   - After a successful merge, return to the original branch if a different target was selected, then pop the recorded stash using `git stash pop --index <stash-ref>` to restore staged and unstaged changes as well as untracked files. Locate the stash ref by its recorded commit ID rather than assuming it is the latest stash.
   - If the merge or branch switch fails without an in-progress merge, return to the original branch and restore the stash when safe; otherwise leave it intact and explain how to recover it. If the stash cannot be identified, or popping it fails or conflicts, stop and report the stash ID and affected paths; do not drop the stash or overwrite local changes.
6. **Validate and publish.**
   - Review the merge result and restored changes, and run the repository's relevant documented checks. Report any failures; do not claim the sync succeeded if validation or the merge failed.
   - Show the resulting target commit and, if its remote branch exists, ahead/behind status relative to `origin/<target>`.
   - Push the target branch to `origin` only after asking the user for confirmation, unless the user explicitly requested pushing in the current request. Never force-push.

## Safety

- Keep `origin` as the fork and `upstream` as the canonical repository.
- Preserve existing stashes and do not overwrite local work or alter remote configuration without consent.
- If upstream history was rewritten, the branches are unrelated, or the default branch cannot be determined, stop and explain the situation before choosing a recovery strategy.
