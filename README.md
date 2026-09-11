# Plugin Factory CI

Reusable GitHub Actions workflows for WordPress plugin repositories.

The workflows in this repository are called from individual plugin repositories;
plugin repositories keep their project-specific commands and deployment policy,
while this repository owns the shared quality and packaging gates.

## Use from a plugin repository

Create a caller workflow in the plugin repository and pin both reusable workflows
to a release tag or an immutable commit SHA:

```yaml
name: Plugin CI

on:
  push:
  pull_request:

permissions:
  contents: read

jobs:
  quality:
    uses: quadcore-factory/plugin-factory-ci/.github/workflows/reusable-plugin-quality.yml@v1
    with:
      php-version: '8.3'
      run-lint: true
      run-tests: true

  package:
    needs: quality
    uses: quadcore-factory/plugin-factory-ci/.github/workflows/reusable-plugin-package.yml@v1
    with:
      package-name: my-plugin
      include-vendor: false
```

Replace `@v1` with a reviewed release tag such as `@v1.0.0`, or pin to a full
commit SHA when the caller requires immutable workflow source. Quality expects a
Composer project and runs its `lint` and `test` scripts by default. A requested
script that is absent fails the corresponding gate clearly.

The package workflow uploads a zip artifact. It excludes `.git`, `.github`,
`tests`, `docs`, Markdown and common PHP tool configuration files; `vendor/` is
also excluded unless `include-vendor` is enabled, in which case only production
dependencies are installed.

These workflows do not deploy releases, access environment secrets, or assume
self-hosted runners. Deployment and release policy remain in each plugin repo.
