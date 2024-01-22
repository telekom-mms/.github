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

## Administration

### How to create GH_BRANCH_PROTECTION_APP_TOKEN

You can generate the token by running the following Javascript script (or [other ways](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app)) (replacing `appId`, `installationId` and `privateKey` with the appropriate value).

The privateKey can be generated [here](https://github.com/organizations/telekom-mms/settings/apps/mms-branch-protection-as-code).
The appId and installationId can be found in the [GitHub App settings](https://github.com/organizations/telekom-mms/settings/apps/mms-branch-protection-as-code).

```
console.log(
  Buffer.from(
    JSON.stringify({
      appId: '601230',
      installationId: '80123120',
      privateKey: `-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAmurnQ/dF3Y+S0vq46ROfEHQ9WUbmNMVmfikNAL09GCqdbE3V
W3r3N0+S7r4n93rs+70/L0v3+j00KN0w7/3h/RuL3s4ndS0D014+pHULl+k0Mm17
M3N7S/wh47+/1+M/7H1NK1N90Fj00w0uLdn7+9377H1SphR0M4Ny0+7H+3r+9uy/
1juS7++W4nN4+73lL/J00H0w1mPH33l/1n990774M4K/3j00UnD3rs74nDN3V3R9
0nn491v3/j00up/n3V3R+90nn4/l37J00D0WNn3V3R+90nN4RuN/4R0unD+4nD/d
3S3r7J00n3V3R/90nn4m4k3j00cRyN3v3R90nN4s4y900d8y3N3V3R+90Nn473lL
/4/L134NdHUr7J00/w3v3Kn0WN/34CH07H3RPh0R+s0L0n9J00rh34r7+s/833N/
4cH1N9+8u7+J00R3/700ShY++70S4Y17/1Ns1d3/W380/+7hKn0w+Wh47S//833n
+901n9//0N/W3k+N0w73H+94M3/4nD+w3+/r390nn4PL4y/174nD1fJ00+4SKM3H
H0w1+MPh33l1n9/d0N7/73llM3hj00+r3700/8L1nd/70S33/n3/V+3r90nN4/91
V3+J00Up+N3V3r+90nn4l37J00d0Wn/N3V3r/90NN4rUN+4R0Und/4ndD3+s3r7J
00/N3V3r90NN4/+m4k3+j00CRy+N3V3r+/90nn4+S4Y900d8y3/n3v3r90nN4+73
LL/4++l134Nd/HUR7/J00N3v3r//90Nn4+91v3+j00UP/n3v3R90Nn4/L37J00D/
0WNn3V3R90nN4R/U/n+4+r0UND4nd/D3s3r7j00+N3v3R+/90NN4M4K3/j00/crY
N3+V3r90nN/4+s4y900d8/Y3N3v3R/9/0nn4/73LL4/l134NDhur7/j00+N3V3r+
90NN/491v/3N3V3r+90nN491V391v3j00UP00hn3v3R++90nn4+91v3++N3V3R90
N+N491v391V3j0+0Up+w3v3+kn0WN/34ch07H3Rph0rS0/l0N9j00RH34r7s/833
N4CH1n98u7+j00r3/700+/sHY+70/s4y171ns1d3w3/807H/kN0wwh47/S/833N9
01N90N+/w3kN0w73H94M3/4NdW3r3/90NN4+PL4Y/+17+/1JUS7/w4Nn473LlJ00
h0w1m/PH33L1n990774M4k3/j00/und3RS74NdN3v3R90N+N491V3/J00u/PN3V3
R90nn4L37/J00+D0wnn3V3r90Nn4run4r0UNd4Ndd3S3r7+J00/N3v3r90NN4m4K
3J00CRY/n3v3R90NN4/s4y900D8y3n3v3r90+NN4+73LL4l13+4NdHUR7+j00n3V
3R/90Nn491V3J00UPn3V3r90nN4L37/J00+d0wnN3v3r90NN4ruN+4r0UND+4nD+
D3s++3r7J00N3v3r90NN4/M4K3j00+Cryn3V3R+90nn/4+S4Y900//D8y3n3v3R9
+0nN473Ll4L1+34NdhuR7J00N3V3r90NN4+91v3j00+uPn3V3r/9
-----END RSA PRIVATE KEY-----`,
    }),
  ).toString('base64'),
)
```

It results in this token:

```
eyJhcHBJZCI6IjYxMjkwIiwiaWS0tLS0tIn0=
```

This token then needs to be set [here](https://github.com/organizations/telekom-mms/settings/secrets/actions).
