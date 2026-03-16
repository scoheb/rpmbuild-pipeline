---
name: update-upstream
description: >
  Pull latest changes from upstream konflux-ci/rpmbuild-pipeline into the fork.
  Use when the upstream repo has new commits to incorporate.
disable-model-invocation: true
---

# Update from upstream

## Steps

1. Ensure the submodule is initialized:
   ```
   git submodule update --init
   ```

2. Fetch and update the submodule to latest upstream `main`:
   ```
   make -C .hummingbird update-upstream
   ```

3. Regenerate root files:
   ```
   make -C .hummingbird sync
   ```
   Fix any patches that fail to apply (see below).

4. Regenerate patches with clean line numbers:
   ```
   make -C .hummingbird regenerate-patches
   ```

5. Verify:
   ```
   make -C .hummingbird check
   ```

6. Commit everything together (submodule pointer + patches + root files):
   ```
   git add -A
   git commit -m "chore: update upstream submodule"
   ```

## Fixing patch conflicts

When `make sync` fails because a patch no longer applies:

1. Read the failing patch to understand what it changes.
2. For small fixes, edit the `.patch` file directly -- adjust context lines and
   line numbers to match the new upstream.
3. For larger conflicts, use the `/edit-patch` workflow.
4. Re-run `make -C .hummingbird sync` and repeat until all patches apply.
