---
name: edit-patch
description: >
  Edit an existing fork patch in .hummingbird/patches/ using a temporary git
  repo and interactive rebase. Use when modifying an existing patch for
  non-trivial changes.
disable-model-invocation: true
---

# Edit an existing fork patch

For small changes, edit the `.patch` file directly (it's a unified diff), then
run `make -C .hummingbird sync && make -C .hummingbird check`.

## Interactive rebase workflow (for non-trivial changes)

1. Set up a temp repo with upstream + patches as commits:
   ```bash
   patchdir="$(pwd)/.hummingbird/patches"
   tmpdir=$(mktemp -d)
   git init "$tmpdir"
   git -C "$tmpdir" config user.name "patch-edit"
   git -C "$tmpdir" config user.email "patch-edit@localhost"
   rsync -a --exclude=.git .hummingbird/src/ "$tmpdir"/
   git -C "$tmpdir" add -A && git -C "$tmpdir" commit -m "upstream"
   git -C "$tmpdir" tag _upstream
   for p in "$patchdir"/*.patch; do git -C "$tmpdir" am "$p"; done
   ```

2. Interactive rebase -- mark the target patch as `edit`:
   ```bash
   git -C "$tmpdir" rebase -i _upstream
   ```

3. Make changes, amend, continue:
   ```bash
   git -C "$tmpdir" add -A
   git -C "$tmpdir" commit --amend --no-edit
   git -C "$tmpdir" rebase --continue
   ```

4. Export updated patches:
   ```bash
   rm -f "$patchdir"/*.patch
   git -C "$tmpdir" format-patch --zero-commit --no-signature \
     -o "$patchdir" _upstream..HEAD
   rm -rf "$tmpdir"
   ```

5. Verify:
   ```bash
   make -C .hummingbird sync && make -C .hummingbird check
   ```

## Current patches

Review `.hummingbird/patches/` before editing. Filenames encode order and a
short description (`0001-*` is applied first, etc.). Read the headers and diff
hunks to understand intent.
