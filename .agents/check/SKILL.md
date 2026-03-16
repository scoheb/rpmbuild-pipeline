---
name: check
description: >
  Verify the repo root matches upstream + patches. Same check that runs in CI.
  Use after editing patches or root files to confirm consistency.
---

# Check repo root consistency

## Steps

1. Ensure the submodule is initialized:
   ```
   git submodule update --init
   ```

2. Run the check:
   ```
   make -C .hummingbird check
   ```

3. Interpret results:
   - **"Check passed"** -- repo root is in sync.
   - **Diff output + "FAIL"** -- root diverges from upstream + patches.

## Fixing a failed check

The failure means either:
- A root file was edited directly instead of through a patch, OR
- A patch was updated but `make sync` wasn't re-run.

To fix:
1. If the direct edit was intentional, update the corresponding patch in
   `.hummingbird/patches/` to include it, then `make -C .hummingbird sync`.
2. If accidental, run `make -C .hummingbird sync` to regenerate from upstream
   + patches.
3. Re-run `make -C .hummingbird check` to confirm.
