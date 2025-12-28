# Fork Workflow

This repository is a tracking fork of [konflux-ci/rpmbuild-pipeline](https://github.com/konflux-ci/rpmbuild-pipeline). The `main` branch contains patches on top of upstream that are pending contribution back.

## Repository Structure

```
upstream/main:  A---B---C          (konflux-ci/rpmbuild-pipeline)
                        \
fork/main:               C---F1---P1---P2---P3  (fork-only commits + patches)
```

The fork's `main` contains two types of commits:

1. **Fork-only commits** (F1) - Fork-specific changes that should never go upstream (e.g., this documentation). These come first, immediately after upstream.
2. **Upstream patches** (P1, P2, P3) - Changes intended for upstream, each corresponding to an open or pending PR. New patches are simply added on top.

## Setup

Add the upstream remote (one-time):

```bash
git remote add upstream https://github.com/konflux-ci/rpmbuild-pipeline.git
git fetch upstream
```

## Current Patches

To see current patches: `git log --oneline upstream/main..HEAD`

| Description | Upstream PR |
|-------------|-------------|
| fix placement of params.build-platforms | [#128](https://github.com/konflux-ci/rpmbuild-pipeline/pull/128) |
| set "Accept-Encoding: identity" header | [#137](https://github.com/konflux-ci/rpmbuild-pipeline/pull/137) |
| Prevent tar running on non-archive files | [#140](https://github.com/konflux-ci/rpmbuild-pipeline/pull/140) |
| Normalize NVR for OCI tag compatibility in import-to-quay | [#142](https://github.com/konflux-ci/rpmbuild-pipeline/pull/142) |

## Updating from Upstream

When upstream has new commits, rebase your patches on top:

```bash
git fetch upstream
git checkout main
git rebase upstream/main
git push origin main --force
```

Resolve any conflicts that arise during rebase.

## When a PR is Merged Upstream

When one of your PRs is merged into upstream, the corresponding patch(es) should be dropped:

```bash
git fetch upstream
git checkout main
git rebase upstream/main
# Git will pause on commits that are now redundant
git rebase --skip  # for each already-merged patch
git push origin main --force
```

Update the "Current Patches" table in this document to remove the merged entry.

## Contributing a New Patch

1. **Create the patch on a feature branch from upstream/main:**

   ```bash
   git checkout -b feature/my-change upstream/main
   # make changes
   git commit -m "Description of change"
   ```

2. **Open a PR upstream:**

   ```bash
   git push origin feature/my-change
   gh pr create --repo konflux-ci/rpmbuild-pipeline
   ```

3. **Add the patch to your fork's main:**

   ```bash
   git checkout main
   git cherry-pick <commit-sha>
   git push origin main --force
   ```

4. **Update this document** with the new patch in the "Current Patches" table.

## Adding an Existing Upstream PR

To include a patch from an upstream PR that you didn't author:

```bash
# Fetch the PR
git fetch upstream pull/<PR-NUMBER>/head:pr/<PR-NUMBER>

# Cherry-pick onto main
git checkout main
git cherry-pick <commit-sha>  # use commits from the pr/<PR-NUMBER> branch
git push origin main --force
```

## Best Practices

- **Keep patches atomic**: One logical change per commit
- **Maintain independence**: Avoid patches that depend on other uncommitted patches when possible
- **Fork-only commits first**: Keep fork-specific commits (like this documentation) immediately after upstream, so new patches can simply be added on top
- **Tag before rebasing**: `git tag backup-$(date +%Y%m%d)` provides a recovery point
- **Update documentation**: Keep the patches table current
- **Force-push is expected**: The fork's `main` is a moving target by design
