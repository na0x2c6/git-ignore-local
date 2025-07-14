# git-ignore-local

A Git filter script that allows you to maintain local-only changes to tracked files without committing them.

## ⚠️ Important Notice

**This script is half-jokingly created as a workaround that deviates from Git's intended usage patterns.** Instead of relying on this approach, always consider submitting proper change requests to the project. For example, you might propose environment-variable-based configurations to handle developer-specific environment differences.

## How It Works

This script uses Git's filter mechanism with 3-way merge to:
- Keep track of the original file content
- Maintain your local changes separately
- Automatically apply/remove local changes during checkout/staging

## Installation

1. Make the script executable and add it to your PATH:

```bash
chmod +x git-ignore-local
# Add to PATH, e.g., copy to /usr/local/bin/ or add script directory to PATH
```

2. Configure Git filters:

```bash
git config filter.ignore-diff.clean "git-ignore-local filter-clean '%f'"
git config filter.ignore-diff.smudge "git-ignore-local filter-smudge '%f'"
```

3. Add files to `.gitattributes`:

```
/path/to/file.txt filter=ignore-diff
```

## Usage

### Add a file to local ignore

```bash
git ignore-local add path/to/file.txt
```

This creates backup copies of both the original (from Git index) and your local version.

### Remove a file from local ignore

```bash
git ignore-local forget path/to/file.txt
```

### List all locally ignored files

```bash
git ignore-local ls
```

### Show diff between original and worktree files

```bash
git ignore-local diff
```

### Show help

```bash
git ignore-local help
```

## How Filters Work

- **Clean filter** (staging): Merges your local changes with the original to produce the version to be committed
- **Smudge filter** (checkout): Re-applies your local changes to the checked-out file

See [gitattributes documentation](https://git-scm.com/docs/gitattributes#_filter) for details.

## Environment Variables

- `GIT_IGNORE_LOCAL_DIR`: Directory for storing backup files (default: `$(git rev-parse --git-common-dir)/_ignore-local`)

## ⚠️ Critical Warning

**This script intentionally includes conflict markers in its output.** When staging or committing files managed by this script, always verify that no conflict markers are present in the file content. Look for patterns like:

```
<<<<<<< local
=======
>>>>>>> original
```

**Note**: After running `git ignore-local add`, if you make additional changes to the file, those new changes will be detected by Git and can be staged normally. Only the changes present at the time of running `add` are ignored.

## Limitations and Risks

1. **Merge conflicts**: The 3-way merge may produce conflict markers that need manual resolution
2. **Not for team use**: This is intended for individual developer use only
3. **Performance**: Each file operation involves multiple merge operations
4. **Complexity**: Adds a layer of complexity that can make debugging difficult

## Troubleshooting

If `git status` or `git diff` doesn't work as expected after setting up the filters, try:
```bash
git add --renormalize .
```
This forces Git to reapply the filters to all tracked files.

## Example Use Case

You have a configuration file that needs local modifications for your development environment:

```bash
# First, modify the file with your local settings
vim config/database.yml

# Add the file to local ignore (this captures the current local changes)
git ignore-local add config/database.yml

# Now the file appears unmodified to Git
git status  # Shows no changes

# Your local changes persist through checkouts and pulls
git checkout other-branch
git checkout -
# Your local database.yml changes are preserved

# Note: If you modify the file again after running 'add', 
# those new changes will show up in git status
```

---

Remember: **This is a workaround, not a solution.** Use sparingly and always look for better alternatives.
