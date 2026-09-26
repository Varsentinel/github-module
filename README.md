# github-module
Repository to Centralize GitHub Pipeline module

Reusable GitHub Actions workflows shared across Varsentinel Terraform module repos.

| Workflow | Purpose |
|---|---|
| [`tf-check.yml`](.github/workflows/tf-check.yml) | `terraform fmt -check` and Checkov scan |
| [`tf-create-tag.yml`](.github/workflows/tf-create-tag.yml) | Bump the semver tag and publish a GitHub release |

## Usage

Add these caller workflows to a module repo. Triggers stay in the caller; the job logic lives here.

The caller must grant the permissions listed below. A reusable workflow can't have more permissions than its caller gives it.

### TF Checks

`.github/workflows/tf-check.yml`:

```yaml
name: 'TF Checks'

on:
  pull_request:
    paths-ignore:
      - '.github/**'
      - '*.md'

jobs:
  tf-check:
    permissions:
      contents: read
      security-events: write
      actions: read
    uses: Varsentinel/github-module/.github/workflows/tf-check.yml@main
```

| Input | Type | Default | Description |
|---|---|---|---|
| `terraform_version` | string | `1.14.6` | Terraform version to install |
| `working_directory` | string | `.` | Directory containing the Terraform module |
| `checkov_skip_path` | string | `.github` | Path for Checkov to skip |
| `checkov_soft_fail` | boolean | `false` | Report Checkov failures without failing the job |

### Create Tag

`.github/workflows/tf-create-tag.yml`:

```yaml
name: Create Tag

on:
  push:
    branches:
      - main
    paths-ignore:
      - '.github/**'
      - '*.md'

jobs:
  release:
    permissions:
      contents: write
    uses: Varsentinel/github-module/.github/workflows/tf-create-tag.yml@main
    with:
      module_description: Terraform Module for EC2 K3D cluster
```

| Input | Type | Required | Description |
|---|---|---|---|
| `module_description` | string | yes | Short description of the module, shown in the release notes |

| Output | Description |
|---|---|
| `tag` | The tag that was created, e.g. `v1.2.0` |

The version bump comes from the **last commit message** on `main`:

| Commit message contains | Bump |
|---|---|
| `BREAKING CHANGE` or `[major]` | major |
| `feat:` or `[minor]` | minor |
| anything else | patch |

With the default "Create a merge commit" option, the last commit is `Merge pull request #...`, so every merge becomes a patch bump. Use squash merges so the PR title becomes the commit message.

## Setup

- **Private repo:** if this repo is private or internal, go to **Settings → Actions → General → Access** and allow access from repositories in the organization. Otherwise callers fail with "workflow was not found".
- **Pinning:** callers reference `@main`, so changes here reach every repo immediately. For stability, tag this repo (e.g. `v1`) and reference `@v1` instead.
