# TortoiseGit

## Introduction

TortoiseGit is a Windows shell extension that provides a graphical interface for Git operations directly in File Explorer. It is designed for users who understand Git concepts but prefer repository tasks to be integrated with normal Windows file management. Commands are exposed through context menus whose contents depend on whether the selected path belongs to a Git working tree. Additional commands are available through the extended context menu by holding Shift while right-clicking.

The application delegates Git operations to a configured `git.exe`, while auxiliary components provide status caching, diff and merge interfaces, image comparison, blame views, and command dialogs. Explorer icon overlays indicate states such as clean, modified, conflicted, staged, added, deleted, ignored, or unversioned. Because Windows limits the number of overlay handlers available system-wide, other shell extensions can affect which TortoiseGit overlays are displayed. Pressing F5 refreshes overlays and also refreshes file lists or history views in many TortoiseGit dialogs.

TortoiseGit supports repository creation and cloning, commits, branches, tags, remotes, submodules, merge and rebase workflows, stashes, bisect operations, history inspection, patch handling, and conflict resolution. Network access can use SSH or HTTP/HTTPS. SSH can be configured with TortoiseGitPlink and Pageant or with OpenSSH when compatibility with `~/.ssh/config` is required.

For IT teams, the main benefit is controlled access to Git operations through explicit dialogs while retaining standard Git repository semantics. This is useful on Windows engineering workstations where Git tasks coexist with IDEs, deployment tools, and file-oriented administration workflows.

## Working Tree and Commit Workflow

A practical TortoiseGit workflow starts by separating the working tree, index, and committed history. Use **Check for Modifications** before committing to inspect modified, added, deleted, missing, ignored, and unversioned paths. The dialog can also expose files marked with Git's `assume-valid` or `skip-worktree` flags, which is useful when diagnosing why an expected local change is not appearing normally. A diff against the base revision should be reviewed before staging changes.

The Commit dialog lists changed paths under the selected location; **Whole Project** expands the scan to the complete working tree. Checking an unversioned file stages it for the commit, while unchecking a changed path excludes it. Files from other Explorer windows can be dragged into the same Commit dialog when they belong to the same working tree, allowing an atomic commit without scanning from the repository root.

When only part of a modified file belongs in the commit, **Restore after commit** can preserve the complete file, let the user temporarily remove unrelated edits, commit the reduced version, and then restore the remaining local changes. TortoiseGitMerge can assist by marking change blocks for inclusion.

Double-clicking a modified file opens the configured diff tool. Commit messages can use recent-message history, snippets, filename and symbol auto-completion, and an optional `Signed-off-by` line. **Commit & Push** is convenient only when the branch and remote-tracking relationship are already correct; otherwise, review the push parameters separately.

## Branches and Remote Synchronization

The Switch/Checkout dialog changes the working tree to a selected branch, tag, or commit and can create a new branch during the operation. When checking out a remote branch, enabling tracking associates the local branch with its remote-tracking branch so later pull, push, and synchronization dialogs can preselect the expected destination. Checking out a tag or arbitrary commit without creating a branch produces a detached `HEAD`; create a branch before making work that must be retained as a normal development line. Avoid **Overwrite working tree changes** unless uncommitted edits are intentionally disposable.

Fetch and pull are distinct operations. Fetch updates remote-tracking state without integrating it into the current branch, making it suitable for inspection before integration. Pull also performs integration. Fetch options can control tag retrieval and pruning of remote-tracking branches that disappeared remotely.

Push should be treated as a publication step. **Set upstream/track remote branch** establishes the relationship used by later operations. For rewritten history, prefer the safer force mode corresponding to `--force-with-lease`; it rejects the push when the remote branch no longer matches the state previously observed locally. The unrestricted force mode can overwrite changes that were never fetched and should be reserved for tightly controlled cases.

Repositories containing submodules require additional checks. Push can verify that submodule commits recorded by the superproject exist remotely or push missing submodule commits on demand. After upstream changes to `.gitmodules`, run submodule synchronization before updating submodules so local remote settings match the superproject configuration.

## Conflict Resolution and Recovery

Merges should normally start from a clean working tree. TortoiseGit can merge a branch, tag, or selected commit and supports squash, no-fast-forward, and no-commit behavior. A squash integrates file changes without recording a merge parent, while no-fast-forward creates a merge commit even when a fast-forward is possible. Cherry-pick applies selected commits to the current `HEAD`; rebase reapplies commits in sequence and rewrites history, so it should be used carefully on commits that may already be shared.

When a textual conflict occurs, **Edit Conflicts** can open the configured merge tool and create temporary BASE, LOCAL, and REMOTE variants. BASE represents the common ancestor. For merge, pull, stash, and cherry-pick operations, LOCAL is normally the current `HEAD` side and REMOTE is the incoming side. During rebase the interpretation changes: LOCAL represents the commit being replayed, while REMOTE represents the branch onto which it is being replayed. This distinction prevents incorrect conflict choices.

After editing, **Resolved** does not determine correctness; it stages the selected file to tell Git that the conflict has been handled. Rebase and cherry-pick conflicts must then be continued from their respective dialogs rather than treated as ordinary commits.

For recovery, reset modes have different effects. Soft moves `HEAD` while retaining the index and working tree; mixed also resets the index; hard resets both and discards tracked local changes without using the Recycle Bin. Use stash for temporary context switches, and use bisect to locate a regression by repeatedly marking automatically selected revisions as good or bad.

## Configuration, Hooks, and Automation

TortoiseGit depends on a configured Git executable and exposes Git settings through hierarchical scopes. Repository-local values are stored in `.git/config`; project-distributed values can be kept in `.tgitconfig`; user and system scopes apply broadly. Higher-precedence values override lower ones. When a value should come from another scope, use inheritance rather than storing an empty value. Line-ending controls such as `core.autocrlf` and `core.safecrlf` need explicit team policy because inconsistent settings can create large, misleading diffs.

Remote configuration can define separate fetch and push addresses, a PuTTY key, tag-fetch behavior, pruning, and a preferred push remote. Separate fetch and push addresses are useful when read access and authenticated write access use different transports. Diff and merge programs can be configured globally or by file extension.

Client-side hooks can run at start-commit, pre-commit, post-commit, pre-push, post-push, and pre-rebase stages. Hooks receive operation-specific parameters such as a path-list file, commit-message file, error file, and working-tree root. A pre-commit hook can validate exactly what the user selected, while start-commit and pre-commit hooks can modify the UTF-8 commit-message file before the dialog workflow continues.

For interactive automation, `TortoiseGitProc.exe` accepts commands such as `/command:commit`, `/command:log`, `/command:push`, and `/command:rebase` with `/path`. Multiple paths can be separated by `*`, and `/closeonend:2` closes a progress dialog automatically when no error occurred. For unattended repository logic, use Git CLI instead of GUI automation.

`GitWCRev.exe` is useful in build pipelines: it can read the current commit and working-tree state, reject builds with modifications or unversioned files, and substitute revision metadata into generated build files.
