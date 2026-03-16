---
name: test-locally
description: >
  Test an RPM build locally using Podman and Mock in the same container
  environment as the pipeline. Use when smoke-testing a package build.
disable-model-invocation: true
argument-hint: <package-name>
compatibility: Requires podman, dist-git-client, and sufficient disk space.
---

# Test an RPM build locally

Performs a non-hermetic Fedora package build using the same Mock container
environment as the pipeline. Not a full replacement for the Tekton pipeline.

## Usage

```
./test-konflux-build-locally $ARGUMENTS
```

For example: `./test-konflux-build-locally cpio`

## What happens

1. Creates a temp directory under `/tmp/konflux-build-<name>-*`
2. Clones the package from Fedora DistGit via `dist-git-client`
3. Downloads source tarballs
4. Extracts the Mock environment image from `pipeline/build-rpm-package.yaml`
5. Runs Podman with Mock to build the package
6. Outputs RPMs to a `results/` directory

## What this tests

- Mock environment image works
- Package builds with current Mock config

## What this does NOT test

- Hermetic builds, multi-arch builds
- Trusted Artifacts, Quay/Pulp upload
- Full Tekton pipeline orchestration

## Troubleshooting

- `dist-git-client` not found: install from your distro or pip.
- Podman permission errors: ensure rootless Podman with `--privileged` works.
- Default chroot: `fedora-rawhide-x86_64`.
