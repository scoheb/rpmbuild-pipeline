---
name: validate-bundles
description: >
  Validate Tekton Task and Pipeline YAML files are well-formed. Same check that
  runs in CI during merge trains. Use before pushing changes to task/ or
  pipeline/ files.
---

# Validate task and pipeline YAML

## Steps

Validate all YAML files:
```bash
for file in task/*.yaml pipeline/*.yaml; do
  echo "Validating $file..."
  yq eval '.' "$file" > /dev/null
done
```

If `yq` is not available, fall back to:
```bash
python3 -c 'import yaml, sys; yaml.safe_load(open(sys.argv[1]))' <file>
```

## What to check beyond syntax

- **Indentation**: 2-space throughout, including dictionary values.
- **Bundle refs**: The fork uses `__TASK_BUNDLE_TAG__` placeholders for task
  bundle references (substituted at CI bundle-push time).
- **Renovate compatibility**: keep OCI image reference formatting consistent so
  the Renovate bot can auto-update them.
- **Task placement**: tasks specific to this pipeline go in `task/`. Reusable
  tasks belong in the upstream build-definitions repo.

## CI bundle workflow

GitLab CI (`.gitlab-ci.yml`) publishes bundles to Quay:
- **Test** (MRs): `pipeline-{CI_PIPELINE_ID}` tag, 7-day expiry
- **Release** (main): `git-{CI_COMMIT_SHA}` tag + `latest` for pipeline
- Destination: `quay.io/hummingbird-ci/rpmbuild-task-{name}:{tag}` and
  `quay.io/hummingbird-ci/rpmbuild-pipeline:{tag}`
