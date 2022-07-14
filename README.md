# Pypestream's GitHub Reusable workflows

There are multiple reusable workflows available 



## Continuous Integration `ci.yml`

### Prerequisites

- Secrets
    - `registry_url`
    - `vpn_profile`
    - `gh_personal_access_token`
    - `ssh_private_key` (Deploy Key)

- Inputs
    - Not Required

### Example

```yaml
name: CI 

on: [push]

jobs:
  continuouos-integration:
    uses: pypestream/github-workflows/.github/workflows/ci.yml@master
    secrets:
      registry_url: ${{ <secrets reference e.g secrets.SECRET_NAME> }}
      vpn_profile: ${{ <secrets reference e.g secrets.SECRET_NAME> }}
      gh_personal_access_token: ${{ <secrets reference e.g secrets.SECRET_NAME> }}
      ssh_private_key: ${{ <secrets reference e.g secrets.SECRET_NAME> }}
```


## Tagging `tag.yml`

### Prerequisites

- Secrets
    - `harbor_username`
    - `harbor_password`
    - `registry_url`
    - `vpn_profile`
    - `gh_personal_access_token`
    - `ssh_private_key`

- Inputs
    - `base_image_path`

### Example

```yaml
name: Build Tag
on:
  push:
    tags:
      - "*"

jobs:
  build-tag:
  
    uses: pypestream/github-workflows/.github/workflows/tag.yml@master

    with:
      base_image_path: devops/docker_build
    
    secrets:
      harbor_username: ${{ <secret> }}
      harbor_password: ${{ <secret> }}
      registry_url: ${{ <secret> }}
      vpn_profile: ${{ <secret> }}
      gh_personal_access_token: ${{ <secret> }}
      ssh_private_key: ${{ <secret> }}
```


## Build and Tag automatically for node applications `build-bump-and-tag.yml`
This workflow builds npm projects and run test cases and if all passes it'll bump the version using sementic versioning and push new tag on main branch.

### Prerequisites

- Secrets
    - `gh_personal_access_token`

- Inputs
    - `node_version`

### Example

```yaml
name: Build bump tag

on:
    push:
        branches:
            - "main"

jobs:
  build-tag:

    uses: pypestream/github-workflows/.github/workflows/build-bump-and-tag.yml@master

    with:
      node_version: "14"
      debug: false

    secrets:
      gh_personal_access_token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
```


## NPM Publish package `publish-npm.yml`
This workflow will be triggered when `Build bump tag` completes successfully and it'll publish the newer version to github private npm registry

### Prerequisites

- Secrets
    - `gh_personal_access_token`

- Inputs
    - `node_version`
    - `workflow_run_conclusion`

### Example

```yaml
name: Release

on:
    workflow_run:
        workflows: ["Build bump tag"]
        types:
            - completed
jobs:
  build-tag:

    uses: pypestream/github-workflows/.github/workflows/publish-npm.yml@master

    with:
      node_version: "14"
      debug: false
      workflow_run_conclusion: ${{ github.event.workflow_run.conclusion }}

    secrets:
      gh_personal_access_token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
```

## Create Pull Request `create-pull-request.yml`
This workflow will create pull requests upon push to any branch and the pull request title will contain the name of author and source branch > destination branch

### Prerequisites

- Secrets
    - `gh_personal_access_token`

- Inputs
    - `destination_branch`
    - `pr_template_file`

### Example

```yaml
name: Create Pull Request

on:
    push:
        branches:
            - "*/*-*/*"
            - "!main" # This is to protect specific branches
jobs:

  create-pr:
        
    uses: pypestream/github-workflows/.github/workflows/create-pull-request.yml@master

    with:
      destination_branch: main
      pr_template_file: ".github/PULL_REQUEST_TEMPLATE.md"
      debug: false

    secrets:
      gh_personal_access_token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
```
