---
name: regenerate-patches
description: >
  Re-export patches with clean line numbers. Does not change content, only
  normalizes context. Use after sync reports offset warnings or as part of the
  update-upstream workflow.
---

# Regenerate patches

## When to use

- After `make sync` reports offset warnings.
- After editing patches, to clean up context.
- As part of the `/update-upstream` workflow.

## Steps

1. Confirm the repo root is already in sync:
   ```
   make -C .hummingbird check
   ```

2. Regenerate:
   ```
   make -C .hummingbird regenerate-patches
   ```

3. Re-sync and verify:
   ```
   make -C .hummingbird sync && make -C .hummingbird check
   ```

4. Review with `git diff` to confirm only context line numbers changed.
