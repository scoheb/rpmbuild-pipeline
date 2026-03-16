---
name: sync
description: >
  Rebuild the repo root from upstream submodule + fork patches. Use when patches
  have been added or edited and the root files need regenerating.
---

# Sync repo root from upstream + patches

## Steps

1. Ensure the submodule is initialized:
   ```
   git submodule update --init
   ```

2. Run the sync target:
   ```
   make -C .hummingbird sync
   ```
   This removes old synced files, copies upstream from `.hummingbird/src/`, and
   applies all patches from `.hummingbird/patches/` in order.

3. Verify the result:
   ```
   make -C .hummingbird check
   ```

4. Review changes with `git diff`.

## If a patch fails to apply

- Read the failing patch in `.hummingbird/patches/` to understand its intent.
- Edit the `.patch` file directly (it's a standard unified diff) to fix context.
- Re-run `make -C .hummingbird sync && make -C .hummingbird check`.
- For non-trivial conflicts, use the `/edit-patch` workflow instead.

## What gets synced

The `SYNC_ITEMS` list in `.hummingbird/Makefile` controls which upstream files
are copied: `pipeline/`, `task/`, `docs/`, `CONTRIBUTING.md`, `COPYING`,
`README.md`, `renovate.json`, `diff-flavor.sh`, `other-flavors.sh`,
`test-konflux-build-locally`. Everything else (`.agents/`, `.claude/`,
`.hummingbird/`, `.tekton/`, `.gemini/`) is untouched.
