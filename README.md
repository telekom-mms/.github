# .github

Documentation, template files, workflows and descriptions for this Organization.
The repository uses a [functionality of GitHub](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions), where you can provide certain documents and use them as default documents for all repositories in an organization.

## Workflow Actions

### GitHub Repository Settings

#### Description

Configures the GitHub repository settings via the API.

It should only run on `push` to master/main branch and on schedule.

#### Requirements

The GitHub Application `MMS settings as code` must be installed in the GitHub repository. The Settings must be specified in a json configuration file, default is `.github/settings.json`. The configuration of the settings based on the [GitHub REST API](https://docs.github.com/en/rest) documentation.

| setting      | REST API Documentation                                                                                                                       | required basic configuration |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| repository   | [Update a repository](https://docs.github.com/en/rest/repos/repos?apiVersion=latest#update-a-repository)                                     | `{ "repos": {} }`            |
| collaborator | [Add a repository collaborator](https://docs.github.com/en/rest/collaborators/collaborators?apiVersion=latest#add-a-repository-collaborator) | `{ "collaborators": {} }`    |

#### Inputs

| secrets                  | description                                                                                                                                  | required |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| GH_APP_CREDENTIALS_TOKEN | password of the GitHub Application `MMS settings as code`, created by @jandd as an Organisation secret, scoped to `terraform-*` repositories | true     |

| inputs   | description                    | type   | required | default               |
| -------- | -------------------------------| ------ | -------- | --------------------- |
| settings | path/name of the settings file | string | false    | .github/settings.json |

#### Example Usage

``` yaml
name: Settings

on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 10 * * *'

jobs:
  call-settings:
    # docs: https://github.com/telekom-mms/.github#github-repository-settings
    uses: telekom-mms/.github/.github/workflows/github_repository.yml@main
    secrets:
      GH_APP_CREDENTIALS_TOKEN: ${{ secrets.GH_APP_CREDENTIALS_TOKEN }}

```

`.github/settings.json`

``` json
{
  "repos": {
    "description": "A Terraform module that manages the container resources from the azurerm provider.",
    "homepage": "https://telekom-mms.github.io",
    "visibility": "public",
    "default_branch": "main",
    "topics": [
      "terraform",
      "azure",
      "azurerm-container-registry"
    ]
  }
}
```

### Publish collection to Ansible Galaxy

#### Description

Publishes an Ansible collection to Ansible Galaxy.

It should only run when a release is published.

The action builds and deploys the collection. It then checks out the main-branch and updates the `galaxy.yml` with the tag and finally pushes the `galaxy.yml`

#### Inputs

| secrets                  | description                                                                                                                                  | required |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| GALAXY_API_KEY           | the API key to deploy to Galaxy, created by @rndmh3ro as an Organisation secret                                                              | true     |
| GH_APP_CREDENTIALS_TOKEN | password of the GitHub Application `MMS settings as code`, created by @jandd as an Organisation secret, scoped to `terraform-*` repositories | true     |

#### Example Usage

``` yaml
name: Publish collection to Ansible Galaxy

on:
  release:
    types:
      - published

jobs:
  deploy:
    # docs: https://github.com/telekom-mms/.github#publish-collection-to-ansible-galaxy
    uses: telekom-mms/.github/.github/workflows/ansible-galaxy-publish.yml@main
    secrets:
      GALAXY_API_KEY: ${{ secrets.GALAXY_API_KEY }}
      GH_APP_CREDENTIALS_TOKEN: ${{ secrets.GH_APP_CREDENTIALS_TOKEN }}
```

### Publish role to Ansible Galaxy

#### Description

Publishes an Ansible role to Ansible Galaxy.

It should only run when a release is published.

The action builds and deploys the role.

#### Inputs

| secrets        | description                                                                     | required |
| -------------- | ------------------------------------------------------------------------------- | -------- |
| GALAXY_API_KEY | the API key to deploy to Galaxy, created by @rndmh3ro as an Organisation secret | true     |

#### Example Usage

``` yaml
name: Publish role to Ansible Galaxy

on:
  release:
    types:
      - published

jobs:
  deploy:
    # docs: https://github.com/telekom-mms/.github#publish-role-to-ansible-galaxy
    uses: telekom-mms/.github/.github/workflows/ansible-galaxy-role-publish.yml@main
    secrets:
      GALAXY_API_KEY: ${{ secrets.GALAXY_API_KEY }}
```

### Python Linting

#### Description

Lint Python code with poetry and black.

It should run on `push` and `pull_request`.

The action sets up Python and Poetry, installs required dependencies and lints the code.

#### Inputs

#### Example Usage

``` yaml
name: Linting

on: [push, pull_request]

jobs:
  linting:
    # docs: https://github.com/telekom-mms/.github#python-linting
    uses: telekom-mms/.github/.github/workflows/python_linting.yml@main
```


### Golang Linting

#### Description

Lint Golang code.

It should run on `push` and `pull_request`.

The action sets up Go, installs required modules and lints the code.

#### Inputs

#### Example Usage

``` yaml
name: Linting

on: [push, pull_request]

jobs:
  linting:
    # docs: https://github.com/telekom-mms/.github#golang-linting
    uses: telekom-mms/.github/.github/workflows/golang_linting.yml@main
```

### Release

#### Description

Publishes a new *draft* release on GitHub.

It should only run on `push` to master/main branch.

The action creates a new *draft* release and updates the changelog. Furthermore it adds defined files (e.g. created in other workflows) to the release.
The user then has to publish the new release.

The push step of this action used the [Github app](https://github.com/organizations/telekom-mms/settings/installations) "MMS branch protection as code"
to push to the main branch, bypassing branch protection rules. Make sure to pass the secret to the job (see the example).

#### Inputs

| inputs | description                                   | type   | required |
| ------ | --------------------------------------------- | ------ | -------- |
| files  | files which should be included in the release | string | false    |

| secrets                        | description                                                                                                                                                                                          | required |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| GH_BRANCH_PROTECTION_APP_TOKEN | password of the GitHub Application `MMS branch protection as code`, created by @jandd as an Organisation secret, scoped to specific repositories. Pass it exactly as described in the example below. | true     |

#### Example Usage

``` yaml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    # docs: https://github.com/telekom-mms/.github#release
    if: github.repository != '$TEMPLATE_REPOSITORY'
    uses: telekom-mms/.github/.github/workflows/release.yml@main
    secrets:
      GH_BRANCH_PROTECTION_APP_TOKEN: ${{ secrets.GH_BRANCH_PROTECTION_APP_TOKEN }}
```

### Terraform Docs

#### Description

Creates the README.md from terraform code. For custom configuration create a `.terraform-docs.yml` in your repository. For further configuration options see [terraform-docs](https://terraform-docs.io/user-guide/configuration/).

It should only run on push to master/main branch.

#### Inputs

#### Example Usage

``` yaml
name: Update Docs

on:
  push:
    branches:
      - main

jobs:
  readme:
    # docs: https://github.com/telekom-mms/.github#terraform-docs
    uses: telekom-mms/.github/.github/workflows/terraform_docs.yml@main
```

### Terraform Linting

#### Description

Lint Terraform code.

It should run on `push` and `pull_request`.
#### Inputs

none

#### Example Usage

``` yaml
name: Linting

on: [push, pull_request]

jobs:
  linting:
    # docs: https://github.com/telekom-mms/.github#terraform-linting
    uses: telekom-mms/.github/.github/workflows/terraform_linting.yml@main
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
name: Scan

on: [pull_request]

jobs:
  scan:
    # docs: https://github.com/telekom-mms/.github#terrascan
    uses: telekom-mms/.github/.github/workflows/terrascan.yml@main
    with:
      iac_type: terraform
      policy_type: all
      non_recursive: true
```

### Terratest

#### Description

Automated Tests for your infrastructure code.

It should run on `pull_request`.

The action sets up Go, logs into to your hosting platform if needed, prepares the test setup and runs the tests. For further information about the used action for the tests see [terratest-action](https://github.com/telekom-mms/terratest-action).

#### Inputs

| secrets               | description                                                                                                                      | required |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------- |
| AZURE_CLIENT_SECRET   | password of the azure ad application, created by @rndmh3ro as an Organisation secret, scoped to `terraform-*` repositories       | false    |
| AZURE_CLIENT_ID       | application_id of the azure ad application, created by @rndmh3ro as an Organisation secret, scoped to `terraform-*` repositories | false    |
| AZURE_SUBSCRIPTION_ID | azure subscription id, created by @rndmh3ro as an Organisation secret, scoped to `terraform-*` repositories                      | false    |
| AZURE_TENANT_ID       | azure tenant id, created by @rndmh3ro as an Organisation secret, scoped to `terraform-*` repositories                            | false    |

| inputs | description                                          | type   | required |
| ------ | -----------------------------------------------------| ------ | -------- |
| test   | name of the test to run, currently supported [azure] | string | true     |

#### Example Usage

``` yaml
name: Test

on: [pull_request]

jobs:
  test:
    # docs: https://github.com/telekom-mms/.github#terratest
    uses: telekom-mms/.github/.github/workflows/terratest.yml@main
    with:
      test: azure
    secrets:
      AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
```

### Codespell

#### Description

Spellchecking that is compatible with a lot of programming languages and text formats.

It should run on `push` and `pull_request`.

Integrates the tool [codespell](https://github.com/codespell-project/codespell).
#### Inputs

| inputs              | description                                                       | type   | required |
| ------------------- | ------------------------------------------------------------------| ------ | -------- |
| ignore_words_list   | Comma-separated list of words which will be ignored by codespell. | string | false    |
| skip                | Comma-separated list of files to skip (it accepts globs as well). | string | false    |

#### Example Usage

``` yaml
name: Codespell - Spellcheck

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  codespell:
    # docs: https://github.com/telekom-mms/.github#codespell
    uses: "telekom-mms/.github/.github/workflows/codespell.yml@main"
```
### ansible-lint 

#### Description

Run ansible-lint against your code.

It should run on `push` and `pull_request`.

Integrates the tool [ansible-lint](https://github.com/ansible/ansible-lint)
#### Inputs

none

#### Example Usage

``` yaml
name: ansible-lint

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  codespell:
    # docs: https://github.com/telekom-mms/.github#ansible-lint
    uses: "telekom-mms/.github/.github/workflows/ansible-lint.yml@main"
```

### Prettier-md

#### Description

* An opinionated code formatter
* Supports many languages
* Integrates with most editors
* Has few options

It should run on `push` and `pull_request`.

Integrates the tool [prettier](https://prettier.io/)

#### Inputs

none

#### Example Usage

``` yaml
name: Prettify Markdown

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  prettier-md:
    # docs: https://github.com/telekom-mms/.github#prettier-md
    uses: "telekom-mms/.github/.github/workflows/prettier-md.yml@main"
```
