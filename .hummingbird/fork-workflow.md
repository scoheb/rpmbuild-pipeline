# Fork Workflow

Managed fork of [konflux-ci/rpmbuild-pipeline](https://github.com/konflux-ci/rpmbuild-pipeline). Upstream lives in `.hummingbird/src/` (git submodule) and fork-specific changes live in `.hummingbird/patches/` (ordered `git format-patch` files). The repo root is generated — never edit root files directly.

## Make Targets

```bash
make -C .hummingbird sync                # rebuild repo root from upstream + patches
make -C .hummingbird check               # verify repo root is in sync (runs in CI)
make -C .hummingbird regenerate-patches   # re-export patches with current line numbers
make -C .hummingbird update-upstream      # pull latest upstream into the submodule
```

## Editing a Patch

For small changes, edit the `.patch` file directly — it's a standard unified diff.
Then run `make -C .hummingbird sync && make -C .hummingbird check`.

For larger changes, use a temporary git repo:

```bash
# 1. Set up a temp repo with upstream + patches as commits
patchdir="$(pwd)/.hummingbird/patches"
tmpdir=$(mktemp -d)
git init "$tmpdir"
git -C "$tmpdir" config user.name "patch-edit"
git -C "$tmpdir" config user.email "patch-edit@localhost"
rsync -a --exclude=.git .hummingbird/src/ "$tmpdir"/
git -C "$tmpdir" add -A && git -C "$tmpdir" commit -m "upstream"
git -C "$tmpdir" tag _upstream
for p in "$patchdir"/*.patch; do git -C "$tmpdir" am "$p"; done

# 2. Interactive rebase — mark the target patch as "edit"
git -C "$tmpdir" rebase -i _upstream

# 3. Make your changes in $tmpdir, then:
#    git -C "$tmpdir" add -A
#    git -C "$tmpdir" commit --amend --no-edit
#    git -C "$tmpdir" rebase --continue
#    (resolve conflicts if any, repeat until rebase finishes)

# 4. Export updated patches
rm -f "$patchdir"/*.patch
git -C "$tmpdir" format-patch --zero-commit --no-signature \
  -o "$patchdir" _upstream..HEAD
rm -rf "$tmpdir"

# 5. Verify
make -C .hummingbird sync && make -C .hummingbird check
```

## Updating Upstream

1. `make -C .hummingbird update-upstream`
2. `make -C .hummingbird sync` — fix any patches that fail to apply.
3. `make -C .hummingbird regenerate-patches`
4. `make -C .hummingbird check`
5. Commit everything (submodule, patches, root files).

## Rules

- **All fork changes must be patches.** The repo root must be fully reproducible from `src/` + `patches/`.
- **Do not revert upstream changes in patches.** Patches add fork functionality only.
- **New fork-specific files** (e.g. `.gitlab-ci.yml`) must be created by a patch.
