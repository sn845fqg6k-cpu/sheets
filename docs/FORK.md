# Personal fork workflow

This checkout is a fork of [`CasualOffice/sheets`](https://github.com/CasualOffice/sheets),
maintained at [`sn845fqg6k-cpu/sheets`](https://github.com/sn845fqg6k-cpu/sheets).

## Remotes

| Remote | URL | Purpose |
| --- | --- | --- |
| `origin` | `https://github.com/sn845fqg6k-cpu/sheets.git` | push local work |
| `upstream` | `https://github.com/CasualOffice/sheets.git` | pull upstream changes |

Both use HTTPS. The submodules in `.gitmodules` still use SSH URLs; see
[Submodule access](#submodule-access) if the machine has no GitHub SSH key.

## Sync from upstream

```bash
git fetch upstream
git switch main
git merge upstream/main
git submodule update --init --recursive
pnpm install

# Only when the Univer fork or design-system submodule changed:
pnpm fork:setup
```

Resolve conflicts in `apps/web`, `packages/sdk`, and the submodule gitlinks as
usual. The pinned submodule commits (`apps/server`, `vendor/univer-revamp`,
`vendor/design-system`) move with the merge; update the working copies with
`git submodule update --init --recursive` before building.

Prefer `git merge` over `git rebase` once the fork has its own commits: rebasing
rewrites shared history and makes upstream pulls harder to audit.

## Push to the fork

```bash
git push origin main
```

To contribute a change back, open a PR from
`sn845fqg6k-cpu:main` (or a topic branch) to `CasualOffice:main`.

## Submodule access

`.gitmodules` points at `git@github.com:...` URLs. On a machine without a GitHub
SSH key, clone them with a one-shot HTTPS rewrite:

```bash
git -c url."https://github.com/".insteadOf="git@github.com:" \
  submodule update --init --recursive
```

Or persist the rewrite for this checkout:

```bash
git config url."https://github.com/".insteadOf "git@github.com:"
```

## Build artifacts

`pnpm fork:setup` swaps the vendored fork's package.json `main`/`exports` to
`lib/` and leaves 61 modified files inside `vendor/univer-revamp`. That is a
local build artifact, not source: the parent repository records only the
submodule SHA, so it is not part of a commit. Run `pnpm fork:restore` to return
the submodule to its source shape.

## Validation

Before pushing, follow the gate in [`CONTRIBUTING.md`](../CONTRIBUTING.md):
`lint`, `format:check`, `typecheck`, `test:unit`, `build:web`, plus the relevant
Playwright suite for UI changes.
