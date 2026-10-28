# ncdu — Disk Usage Analyzer & Exclude Patterns

`ncdu` is an interactive, curses-based disk usage analyzer (an enhanced `du`).
This page focuses on the **exclude** behaviour: keeping package / virtual-env
folders out of the size calculation while still showing the full tree.

## Install

```bash
# <DISTRO> — replace <PKG-MGR> with the system package manager
sudo <PKG-MGR> install ncdu
```

## Basic full-disk scan

```bash
# stay on one filesystem, scan the root
sudo ncdu -x /
```

## Exclude directories from the size calculation

Excluded entries **stay visible** in the tree (flag `<`) but **count 0 bytes**.
Parent and total sizes therefore stay correct, and `ncdu` does not even descend
into them — which also makes the scan much faster.

```bash
# one pattern per --exclude flag
ncdu -x / \
  --exclude node_modules \
  --exclude .pnpm \
  --exclude .venv \
  --exclude venv \
  --exclude __pycache__

# or read patterns from a file (one pattern per line)
ncdu -x / -X ~/.ncdu-excludes
```

### Pattern semantics (rsync style)

| Pattern     | Meaning                                              |
|-------------|------------------------------------------------------|
| `name`      | Matches on **every** level of the tree               |
| `/name`     | Anchored to the scan root (absolute path)            |
| `name/`     | Matches **directories only** (ncdu ≥ 2.2)            |
| `sub/name`  | Matches `name` only inside a `sub` directory         |

For typical dev workspaces the unanchored, directory-only form is the safest:

```bash
ncdu -x / --exclude node_modules/ --exclude .venv/ --exclude venv/
```

## Persistent configuration

Put default options in `~/.config/ncdu/config` (one option per line,
`#` starts a comment, a leading `@` suppresses errors for that line):

```
# Always exclude dependency / cache folders
--exclude node_modules/
--exclude .pnpm/
--exclude .venv/
--exclude venv/
--exclude __pycache__/

# Load extra excludes from a file, ignore if it does not exist
@--exclude-from ~/.ncdu-excludes
```

Command-line options override the config file. Use `--ignore-config` to skip it.

## File flags in the browser

| Flag | Meaning                                             |
|------|-----------------------------------------------------|
| `<`  | Excluded by an exclude pattern (shown, 0 bytes)     |
| `>`  | On another filesystem (`-x` stops here)             |
| `^`  | Excluded Linux pseudo filesystem (procfs, sysfs, …) |
| `e`  | Empty directory                                     |
| `H`  | Hard link that was already counted                  |
| `@`  | Not a file or directory (symlink, socket, …)        |
| `!`  | Error reading this directory                        |

Toggle the visibility of hidden/excluded entries with the `e` key.

## Cross-filesystem scan (`>` flag pitfall)

`-x` (`--one-file-system`) stops at filesystem boundaries. A separate `/home`
partition or a Btrfs subvolume counts as a different filesystem, so `/home`
shows up as `>` with `0.0 B` and is never scanned.

Two fixes:

```bash
# A) scan the mount point that holds your projects directly
ncdu /home -X ~/.ncdu-excludes

# B) cross filesystem boundaries, but skip pseudo filesystems + volatile dirs
ncdu --cross-file-system --exclude-kernfs \
  --exclude /tmp --exclude /run --exclude /dev \
  -X ~/.ncdu-excludes /
```

`--exclude-kernfs` covers procfs / sysfs / devpts / cgroup but **not** tmpfs,
so keep the explicit `/tmp`, `/run`, `/dev` excludes in option B.

## Bonus: cache directories

`--exclude-caches` excludes any directory that contains a valid `CACHEDIR.TAG`
file (e.g. `uv` creates one inside `.venv`), without listing it explicitly.
