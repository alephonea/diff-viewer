# diff-viewer

A single-page, local viewer for reviewing a branch or PR as a full-context diff: side-by-side, with a file tree,
changes-only or full-file view, syntax highlighting, filters, and per-file "reviewed" marks
that persist across reloads.

No build step, no dependencies beyond `bash`, `git`, Python 3, and optionally the
[`gh`](https://cli.github.com/) CLI.

## Quick start

```sh
./mkdiff -r ~/src/myproject my-branch   # writes diffs/my-branch.diff
./serve                                 # http://localhost:8765/
```

Open the URL `mkdiff` prints, or pick a diff from the list on the start page.

## mkdiff

```
mkdiff [-f] [-r repo] <branch|PR#> [out.diff]
```

Writes `git diff -U999999 base...head` into `diffs/` (or to `out.diff` if given; a relative
path is taken under `diffs/`).

- `-r repo` — the git repo to diff; defaults to the current directory.
- `-f` — overwrite an existing output without asking.
- `<PR#>` — a GitHub PR number: diffs the PR's head against its base branch (needs `gh`).
- `<branch>` — a local branch, or one on `origin`. The base is chosen as:
  1. the base of an open PR for that branch, if `gh` finds one;
  2. `master`, if the branch is already merged into it (the diff is empty);
  3. otherwise the nearest other branch: branches whose tip is an ancestor of the target
     win, then the fewest commits back to the merge base, with `master` breaking ties.

`mkdiff` runs `git fetch origin` first.

## serve

```
serve [port] [--prod-roots a,b,...]
```

Serves the viewer on `127.0.0.1` (port 8765 by default) and the diffs under `/diffs/`.
Reviewed marks are kept in `reviews.sqlite` next to the script, keyed by a hash of each
file's diff, so a regenerated diff that changes a file shows it unreviewed again.

`--prod-roots` lists the top-level directories that hold production code. Files outside
them that aren't tests are labelled `misc`. Without it, every non-test file is `prod`.

`view.html` also works from a plain static server (e.g. `python3 -m http.server`), or
opened directly with a diff dragged onto it; reviewed marks need `serve`.

## The viewer

- **Filters** by role (prod / test / misc), by language, and comment-only changes.
- **Keys:** `j`/`k` next/previous file, `n`/`p` next/previous change, `m` toggle
  collapsed/full view, `y` copy the file path, `r` toggle reviewed.
- `?diff=NAME` loads `diffs/NAME`; a value containing `/` is fetched as a path.

## clean.sh

Deletes every `diffs/*.diff`.
