# .github

Documentation, template files, workflows and descriptions for this Organization.
The repository uses a [functionality of GitHub](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions), where you can provide certain documents and use them as default documents for all repositories in an organization.

## Workflow Actions

### Publish collection to Ansible Galaxy

#### Description

Publishes an Ansible collection to Ansible Galaxy.

It should only run when a release is published.

The action builds and deploys the collection. It then checks out the main-branch and updates the `galaxy.yml` with the tag and finally pushes the `galaxy.yml`

#### Inputs

| secrets        | description                                                                     | required |
| -------------- | ------------------------------------------------------------------------------- | -------- |
| GALAXY_API_KEY | the API key to deploy to Galaxy, created by @rndmh3ro as an Organisation secret | true     |

#### Example Usage

``` yaml
name: Publish collection to Ansible Galaxy

on:
  release:
    types:
      - published

jobs:
  deploy:
    uses: T-Systems-MMS/.github/.github/workflows/ansible-galaxy-publish.yml@main
    secrets:
      GALAXY_API_KEY: ${{ secrets.GALAXY_API_KEY }}
```

### Golang Linting

#### Description

Linting Golang code.

It should run on `push` and `pull_request`.

The action sets up Go, installs needed modules and lints the code.

#### Inputs

#### Example Usage

``` yaml
name: linting

on: [push, pull_request]

jobs:
  call-linting:
    uses: T-Systems-MMS/.github/.github/workflows/golang_linting.yml@main
```

### Release

#### Description

Publishes a new Release on GitHub.

It should only run on push to master/main branch.

The action builds a new tag and updates the changelog. Furthermore it adds defined files (e.g. created in other workflows) to the release.

#### Inputs

| inputs | description                                   | type   | required |
| ------ | --------------------------------------------- | ------ | -------- |
| files  | files which should be included in the release | string | false    |

#### Example Usage

``` yaml
name: release

on:
  push:
    branches:
      - main

jobs:
  call-release:
    uses: T-Systems-MMS/.github/.github/workflows/release.yml@main
```

### Terraform Docs

#### Description

Creates the README.md from terraform code. For custom configuration create a `.terraform-docs.yml` in your repository. For further configuration options see [terraform-docs](https://terraform-docs.io/user-guide/configuration/).

It should only run on push to master/main branch.

#### Inputs

#### Example Usage

``` yaml
name: release

on:
  push:
    branches:
      - main

jobs:
  call-readme:
    uses: T-Systems-MMS/.github/.github/workflows/terraform_docs.yml@main
  call-release:
    uses: T-Systems-MMS/.github/.github/workflows/release.yml@main
    with:
      files: README.md
```

### Terraform Linting

#### Description

Lint Terraform code.

It should run on `push` and `pull_request`.

The action lint and validate the code.

#### Inputs

none

#### Example Usage

``` yaml
name: linting

on: [push, pull_request]

jobs:
  call-linting:
    uses: T-Systems-MMS/.github/.github/workflows/terraform_linting.yml@main
```

### Terrascan

#### Description

Detect compliance and security violations across Infrastructure as Code (IaC). For further information see [terrascan](https://runterrascan.io/docs/).

It should run on `pull_request`.

#### Inputs

| inputs        | description                                                                                                                                                                               | type    | default   | required |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | --------- | -------- |
| iac_type      | type of code that should be checked, e.g. helm, k8s, kustomize, terraform - to see all supported types look at [terrascan#iac_type](https://github.com/tenable/terrascan-action#iac_type) | string  | terraform | true     |
| policy_type   | which policies should be tested, e.g. all, aws, azure, gcp, github, k8s - to see all supported types look at https://github.com/tenable/terrascan-action#policy_type                      | string  | all       | false    |
| non_recursive | Weather directories and modules should be scanned recursively                                                                                                                                        | boolean | false     | false    |

#### Example Usage

``` yaml
name: scan

on: [pull_request]

jobs:
  call-scan:
    uses: T-Systems-MMS/.github/.github/workflows/terrascan.yml@main
    with:
      iac_type: terraform
      policy_type: all
      non_recursive: true
```

### Terratest

#### Description

Automated Tests for your infrastructure code.

It should run on `pull_request`.

The action sets up Go, logs into to your hosting platform if needed, prepares the test setup and runs the tests. For further information about the used action for the tests see [terratest-action](https://github.com/T-Systems-MMS/terratest-action).

#### Inputs

| secrets           | description                                                              | required |
| ----------------- | ------------------------------------------------------------------------ | -------- |
| AZURE_CREDENTIALS | Credentials for az login, created by @rndmh3ro as an Organisation secret, scoped to `terraform-*` repositories | true     |

| inputs | description                                          | type   | required |
| ------ | -----------------------------------------------------| ------ | -------- |
| test   | name of the test to run, currently supported [azure] | string | true     |

#### Example Usage

``` yaml
name: test

on: [pull_request]

jobs:
  call-test:
    uses: T-Systems-MMS/.github/.github/workflows/terratest.yml@main
    with:
      test: azure
    secrets:
          AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
```
