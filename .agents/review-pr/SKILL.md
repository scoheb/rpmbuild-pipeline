---
name: review-pr
description: >
  Review a pull request or change with fork-workflow awareness. Use when
  reviewing MRs, PRs, or diffs in this repository to ensure fork rules, YAML
  conventions, and CI requirements are met.
---

# Review a PR / change

## Checklist

### 1. Are root files edited directly?

Files in `pipeline/`, `task/`, `docs/`, `CONTRIBUTING.md`, `COPYING`,
`README.md`, `renovate.json`, `diff-flavor.sh`, `other-flavors.sh`, and
`test-konflux-build-locally` are generated. If modified:
- A corresponding patch in `.hummingbird/patches/` must also be updated.
- `make -C .hummingbird check` must pass (CI runs this automatically).

### 2. Are patches well-formed?

- Must not revert upstream changes -- add fork functionality only.
- New fork-only files must be created by a patch.
- Patches apply in filename order (`0001-*`, `0002-*`).
- Commit messages in patches should be descriptive.

### 3. YAML style

- 2-space indentation throughout.
- OCI image refs must be Renovate-bot compatible.
- `__TASK_BUNDLE_TAG__` placeholders must be present in task bundle refs.

### 4. Pipeline parameters

See `docs/parameters.md`. New parameters should:
- Have sensible defaults.
- Be documented.
- Be backwards-compatible.

### 5. CI checks that must pass

- `upstream:check` -- root matches upstream + patches
- `bundles:validate` -- YAML well-formed (merge train)
- `bundles:test` -- bundles push to Quay with ephemeral tags
- `.tekton/` PipelineRuns build real RPMs on every PR

### 6. Upstream vs. fork

If a change benefits upstream, consider contributing it there first and syncing,
rather than adding a fork-only patch.
