# git-wink

git-wink is a script that abuses git-filter to exclude arbitrary diffs of tracked files from staging

## Motivation

When searching for information about "ignoring differences in files tracked by git," you often come across examples using `git update-index --skip-worktree` or `--assume-unchanged`. Both of these are performance-oriented flags [^sw] [^au] and are not features intended for "ignoring differences."

However, a feature specifically for "ignoring differences" (rather than files) does not currently exist in git itself, and since I could not find any tools for this purpose, I created this script.

[^sw]: https://git-scm.com/docs/git-update-index#_skip_worktree_bit 
[^au]: https://git-scm.com/docs/git-update-index#_using_assume_unchanged_bit

## Important Notice

- The author does not use this script in production; it was written as a proof of concept to demonstrate "ignoring specific diffs of arbitrary files." Instead of relying on this approach, always consider submitting proper change requests to the project. For example, you might propose environment-variable-based configurations to handle developer-specific environment differences.

- This script is implemented by hacking git's filter functionality. For more details about the filter functionality, please refer to the following documentation:
    - [Git - Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)
    - [Git - gitattributes Documentation](https://git-scm.com/docs/gitattributes#_filter)

## Setup

1. Make the script executable and add it to your PATH:

```bash
chmod +x git-wink
# Add to PATH, e.g., copy to /usr/local/bin/ or add script directory to PATH
```

2. Add git filter configuration

You can configure it with the following command:

```sh
git wink setup-config
```

Or configure manually:

```bash
git config filter.wink.clean "git-wink filter-clean '%f'"
git config filter.wink.smudge "git-wink filter-smudge '%f'"
```

## Usage

```bash
# Make any changes
vim some/file.txt

# Diff will be displayed
git diff

# Make the current unstaged changes ignored
git wink add some/file.txt

# Diff will no longer be displayed
git diff

# nothing to commit, working tree clean
git status

# If you want to temporarily disable the filter, use glance
git wink glance status

# `wink diff` is an alias for git glance diff
git wink diff

# You can merge other changes to some/file.txt. If conflicts occur, an error will be raised
git merge some-file-other-change

# List ignored files
git wink ls-files

# Reset ignored diffs. When resolving conflicts, please reset once first
git wink reset some/file.txt

```

## Warning

Using this script creates hard-to-notice "おま環" (works-on-my-machine issues). When something is wrong locally, first use `git wink reset` to display all diffs and carefully check if it's not your own issue.

Remember that this script exists only as a workaround, and making proper contributions to the main project is far smarter.
