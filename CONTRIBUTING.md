# Contributing

## Local setup

`omarchy plugin validate .` rejects a plugin folder that contains a
symlink, so a plain `ln -s` of this repo into
`~/.config/omarchy/plugins/` won't load. Clone it there instead
(a real, separate working copy — like `omarchy plugin add` would
leave):

```bash
git clone "$(pwd)" ~/.config/omarchy/plugins/vinicgobbi.workspaces
omarchy-shell shell rescanPlugins
omarchy plugin enable vinicgobbi.workspaces
```

To pick up local edits without re-cloning, add this repo as a remote
in the installed copy and pull:

```bash
git -C ~/.config/omarchy/plugins/vinicgobbi.workspaces remote add dev "$(pwd)"
git -C ~/.config/omarchy/plugins/vinicgobbi.workspaces pull dev development
```

This plugin's bar widget is a clone of the built-in `omarchy.workspaces`
(own id, so it doesn't collide with the built-in). Because the manifest
declares `omarchy.clonedFrom`, enabling it replaces the built-in in the
same bar position, and disabling it restores the built-in.

The shell logs a reload once the installed copy of `BarWidget.qml` is
updated, but that isn't always enough for the bar to show the new
version. If it still looks the same, restart the shell:

```bash
omarchy restart shell
```

Validate the manifest before publishing:

```bash
omarchy plugin validate .
```

## Structure

- `manifest.json` — plugin metadata (id, kinds, entry points)
- `BarWidget.qml` — cloned from `omarchy.workspaces`'s `Workspaces.qml`:
  the workspace indicators and click-to-focus

## CI

`.github/workflows/ci.yml` runs on every push to `main` (and on pull
requests): it validates `manifest.json`, runs Shellcheck on any shell
scripts, and lints every `.qml` file with `qmllint`, so a syntax error
can't land on `main`.

## Commits and releases

Commits follow [Conventional Commits](https://www.conventionalcommits.org/)
and are checked with [Commitizen](https://commitizen-tools.github.io/commitizen/):

```bash
pipx install commitizen
cz commit   # interactive, conventional-commits-compliant commit
```

Releases are automatic: `.github/workflows/release.yml` runs after CI
passes on every push to `main` (it can also be run by hand from the
Actions tab, on `main`). It uses Commitizen to bump `manifest.json`'s
version and the changelog based on the commit types since the last
release, tags it (`vX.Y.Z`), and publishes a GitHub Release with the
changelog entry. If there's nothing to bump (no `feat`/`fix`/`BREAKING
CHANGE` commits since the last release), it's a no-op — no tag, no
release.

When it does bump, the workflow pushes a `bump: version …` commit to
`main`, so run `git pull` before your next push.
