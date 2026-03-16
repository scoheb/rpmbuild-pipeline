---
name: new-change
description: >
  Create a new fork-specific change as a patch in .hummingbird/patches/. Use
  when adding a new feature, fix, or file that is specific to the Hummingbird
  fork and not present upstream.
disable-model-invocation: true
---

# Make a new fork-specific change

All fork-specific changes must be implemented as patches. Never edit root files
directly.

## Decide: new patch vs. amending existing

- **New patch**: Logically independent change (new feature, new file, unrelated fix).
- **Amend existing**: Extends something an existing patch already touches.

## Creating a new patch

1. Set up a temporary git repo with upstream as base:
   ```bash
   patchdir="$(pwd)/.hummingbird/patches"
   tmpdir=$(mktemp -d)
   git init "$tmpdir"
   git -C "$tmpdir" config user.name "patch-edit"
   git -C "$tmpdir" config user.email "patch-edit@localhost"
   rsync -a --exclude=.git .hummingbird/src/ "$tmpdir"/
   git -C "$tmpdir" add -A && git -C "$tmpdir" commit -m "upstream"
   git -C "$tmpdir" tag _upstream
   ```

2. Apply existing patches:
   ```bash
   for p in "$patchdir"/*.patch; do git -C "$tmpdir" am "$p"; done
   ```

3. Make changes in `$tmpdir` (edit existing files or create new ones).

4. Commit with a descriptive message (becomes the patch header):
   ```bash
   git -C "$tmpdir" add -A
   git -C "$tmpdir" commit -m "feat: description of the change"
   ```

5. Export all patches:
   ```bash
   rm -f "$patchdir"/*.patch
   git -C "$tmpdir" format-patch --zero-commit --no-signature \
     -o "$patchdir" _upstream..HEAD
   rm -rf "$tmpdir"
   ```

6. Sync and verify:
   ```bash
   make -C .hummingbird sync && make -C .hummingbird check
   ```

## Rules

- Patches must not revert upstream changes -- add fork functionality only.
- New fork-only files (e.g. `.gitlab-ci.yml`) must be created by a patch.
- Patches apply in filename order (`0001-*`, `0002-*`, etc.).
