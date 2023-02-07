# .github

Documentation, template files, workflows and descriptions for this Organization.
The repository uses a [functionality of GitHub](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions), where you can provide certain documents and use them as default documents for all repositories in an organization.

## Workflow Actions

### Publish collection to Ansible Galaxy

#### Description

Publishes an Ansible collection to Ansible Galaxy.

It should only run when a release is published.

The action builds and deploys the collection. It then checks out the main-branch and updates the `galaxy.yml` with the tag and finally pushes the `galaxy.yml`

#### Required Inputs

* `GALAXY_API_KEY` - the API key to deploy to Galaxy, created by @rndmh3ro as an Organisation secret.

#### Example Usage

```
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

#### Required Inputs

#### Example Usage

### Release

#### Description

#### Required Inputs

#### Example Usage

### Terraform Docs

#### Description

#### Required Inputs

#### Example Usage

### Terrascan

#### Description

#### Required Inputs

#### Example Usage

### Terratest

#### Description

#### Required Inputs

#### Example Usage
