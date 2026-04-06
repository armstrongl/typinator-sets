# typinator-sets

Private repository of [Typinator](https://www.ergonis.com/typinator) text expansion sets. Links into the [dotfiles](https://github.com/armstrongl/dotfiles) repo as a git submodule at `configs/typinator/share/Published/`.

## What this is

Typinator is a macOS text expansion app. It uses **sets** (`.typubset` files) to group related expansions — typo corrections, code snippets, emoji shortcuts, regex replacements, etc. This repo tracks the **Published** sets: the ones available across all Typinator-synced machines.

Typinator also supports `.tyset` directories, which are bundle-style sets containing an `Index` file and individual entry files (`X*` prefixed). These function identically to `.typubset` files but use a directory structure internally.

## Repository layout

```
Published/
  .gitignore              # Excludes machine-local and private sets
  README.md               # This file
  ai.typubset             # AI tool abbreviations
  ai-prompts.typubset     # Reusable AI prompt snippets
  brand-names.typubset    # Proper casing for brand names (GitHub, macOS, etc.)
  case-corrections.typubset
  claude-code.typubset    # Claude Code-specific expansions
  cli.typubset            # CLI tool shorthands
  double-cap-exceptions.typubset
  filenames.typubset      # Common filename expansions
  gaming.typubset
  github.typubset         # GitHub-specific expansions
  jira.typubset           # Jira ticket/workflow expansions
  logs.typubset           # Log entry formatting
  markdown.typubset       # Markdown syntax shortcuts
  obsidian.typubset       # Obsidian-specific expansions
  regex.typubset          # Regex pattern expansions
  regex-date-steps.typubset
  regex-default.typubset
  regex-double-caps.typubset
  shorthands.typubset     # General abbreviations
  symbols.typubset        # Unicode symbols and special characters
  tailscale-docs-repo.typubset
  tailscale-style.typubset
  typos.typubset          # Common typo auto-corrections
  typos-misspellings.typubset
  utils.typubset          # Utility expansions
  work-typos.typubset     # Work-specific typo corrections
  drafts-and-logs.tyset/  # Bundle-format set for drafts and logs
```

## Ignored files

The `.gitignore` excludes the following files from version control (they remain on disk):

| Pattern | Reason |
|---------|--------|
| `default.typubset` | Machine-local Typinator defaults |
| `dotfiles.typubset` | Dotfiles-repo-specific expansions (paths vary per machine) |
| `emails.typubset` | Contains personal email addresses |
| `personal.typubset` | Personal/private expansions |
| `sieve.typubset` | Email sieve filter rules (environment-specific) |
| `_*.typubset` | Convention for local-only/draft sets (prefix with underscore) |
| `*.DS_Store` | macOS Finder metadata |

To add a new local-only set, prefix the filename with `_` (e.g., `_scratch.typubset`) and the gitignore pattern catches it automatically.

## How it connects to dotfiles

This repo is a **git submodule** inside the dotfiles repository:

```
# In dotfiles/.gitmodules
[submodule "configs/typinator/share/Published"]
    path = configs/typinator/share/Published
    url = git@github.com:armstrongl/typinator-sets.git
```

The dotfiles repo pins a specific commit of this submodule. When sets change, the next sync updates the submodule ref in dotfiles.

### Dotfiles Typinator directory structure

```
configs/typinator/
  archived/       # Old/retired sets
  assets/         # Typinator-related assets
  backups/        # Set backups
  personal/       # Personal (non-shared) sets and includes
  share/
    Published/    # <-- THIS REPO (submodule)
    Sets/         # Other shared sets
  sync/           # Typinator sync configuration
  work/           # Work-specific sets
```

## Automatic syncing

The dotfiles repo uses `git-sync-auto` (at `scripts/shell/git-sync-auto/git-sync-auto.sh`) for automated commits. As of v4.4.0, it **syncs submodules first** before committing the parent repo:

1. Iterates all submodules listed in `.gitmodules`
2. For each dirty submodule: stages all changes, commits, and pushes
3. Then commits the parent dotfiles repo (which picks up the updated submodule refs)

This means: edit a Typinator set, run `git-sync-auto` in the dotfiles repo, and both the submodule and the parent get synced in one command.

### Manual sync

If you need to sync this repo independently:

```bash
cd ~/.dotfiles/configs/typinator/share/Published
git add -A
git commit -m "update sets"
git push
```

Then update the submodule ref in dotfiles:

```bash
cd ~/.dotfiles
git add configs/typinator/share/Published
git commit -m "sync(configs): update typinator-sets submodule"
git push
```

## Setting up on a new machine

### 1. Clone dotfiles with submodules

```bash
# If cloning dotfiles fresh:
git clone --recurse-submodules git@github.com:armstrongl/dotfiles.git ~/.dotfiles

# If dotfiles already cloned without submodules:
cd ~/.dotfiles
git submodule update --init --recursive
```

### 2. Verify the submodule

```bash
cd ~/.dotfiles/configs/typinator/share/Published
git status          # Should show clean working tree
git remote -v       # Should point to armstrongl/typinator-sets
git branch          # Should be on main
```

If the submodule is in detached HEAD state (common after `submodule update`):

```bash
git checkout main
git pull origin main
```

### 3. Point Typinator to the sets

In Typinator preferences, add `~/.dotfiles/configs/typinator/share/Published/` as a set folder, or symlink individual sets into Typinator's default set location. The exact method depends on your Typinator configuration and whether you use Typinator's built-in sync or manage sets manually through dotfiles.

### 4. Restore ignored files

The clone won't include gitignored files (`default.typubset`, `emails.typubset`, etc.). Recreate them manually or restore from a backup. They contain machine-specific paths or personal data, which is why they stay out of version control.

## Common operations

### Add a new shared set

1. Create the `.typubset` file in this directory (Typinator does this when you create a new set in the Published group)
2. `git-sync-auto` picks it up on the next run, or commit manually

### Add a new local-only set

Prefix the filename with `_`:

```bash
# Typinator will create the file; just rename it
mv new-set.typubset _new-set.typubset
```

Or add the exact filename to `.gitignore` if the underscore prefix isn't practical.

### Remove a set from tracking (keep on disk)

```bash
# Add to .gitignore
echo "setname.typubset" >> .gitignore

# Remove from git index only (file stays on disk)
git rm --cached setname.typubset

# Commit
git add .gitignore
git commit -m "ignore setname.typubset"
git push
```

### Purge a set from history entirely

If you accidentally committed a set containing sensitive data:

```bash
# If the repo has few commits, amend the relevant commit and force-push.
# For deeper history, use git filter-repo:
pip install git-filter-repo
git filter-repo --path setname.typubset --invert-paths
git push --force
```

## File format notes

- `.typubset` files use Apple's binary property list format. They aren't human-readable in a text editor, but `plutil -p file.typubset` dumps the contents.
- `.tyset` directories contain an `Index` plist and individual entry files prefixed with `X` followed by a hex ID.
- Typinator manages these files directly. Use the Typinator app to create, modify, and organize sets — don't edit the binary plists manually.
- File sizes vary significantly: typo correction sets (`typos-misspellings.typubset`) can exceed 1MB, while simple sets run 1-2KB.
