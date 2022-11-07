# git-tag

A GitHub Action to create and read git tags.
This is a composite action that combines other actions, like:
- anothrNick/github-tag-action
- peter-evans/find-comment
- peter-evans/create-or-update-comment
- actions/github-script
- chuhlomin/render-template
The simplest use-case for this action is to create a git tag from a push event.
In that case all it does is create and push the tag.
Another use-case is to create a tag from a pull request using a label event.
In that case the action creates and pushes the git tag, but also adds a comment to the pull request with information about the tag.
This action is also able to read comments created in PRs and retrieve the tag name.

## Inputs

| Input                    | Description                                                                                          | Required | Default |
|--------------------------|------------------------------------------------------------------------------------------------------|----------|---------|
| create                   | "Whether to create a tag comment comment."                                                           | true     | ''      |
| read                     | "Whether to read a tag comment.                                                                      | true     | ''      |
| pr-number                | "If the action is running on a PR, this input defines the PR number."                                | false    | ''      |
| release-candidate-suffix | "If the action is running on a PR, this input defines git tag suffix for the release candidate tag." | false    | 'rc'    |
| tag-comment-header       | "The header on the tag comment, used to create the comment and to find existing comments."           | false    | ''      |
| workflow-run-url         | "The url of the workflow run."                                                                       | false    |         |
| github-token             | "The GitHub token used for creating the tag."                                                        | true     |         |


# Outputs

| Output | Description            | 
|--------|------------------------|
| tag    | "The name of the tag." |

## Create a tag on a push event

The following workflow configuration creates tags on push events to `main` branch. 
The tags are created sequentially and prefixed with `v`.
```yaml
name: Git Tag - Push


on:
  push:
    branches:
      - main


jobs:
  tag:
    name: Create git tag
    runs-on: ubuntu-latest
    steps:
      - name: Git tag
        uses: GRESB/action-git-tag@main
        with:
          create: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

##  Create a tag from a pull request

The following workflow configuration creates a tag when a pull request is labeled with `tag`.
The tags are created sequentially taking into account both the latest tag in the repository and the latest tag created in the pull request.
The tags are prefixed with `v` and suffixed with `pr<PR-NUMBER>-rc.<TAG-NUMBER>`.
```yaml
name: Git Tag - Pull Request


on:
  pull_request:
    types: [ labeled ]


jobs:
  tag:
    name: Create git tag with comment on PR
    runs-on: ubuntu-latest
    if:  github.event.label.name == 'tag'
    steps:
      - name: Git tag
        uses: GRESB/action-git-tag@main
        with:
          create: true
          pr-number: ${{ github.event.pull_request.number }}
          release-candidate-suffix: rc
          workflow-run-url: "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
After the tag is created a comment is added to the pull request, naming the tag.
When the same pull request is re-labeled with `tag` the comment is updated.

## Read a tag from a pull request comment

The following workflow configuration reads a previously created tag from a pull request comment, when the pull request is labeled with `read-tag`.
The tag name is then made available as an output.
```yaml
name: Git Tag - Pull Request


on:
  pull_request:
    types: [ labeled ]


jobs:
  read:
    name: Read PR tag
    runs-on: ubuntu-latest
    if: github.event.label.name == 'read-tag'
    outputs:
      release: ${{ steps.read-pr-tag.outputs.tag }}
    steps:
      - name: Read PR Tag
        id: read-pr-tag
        uses: GRESB/action-git-tag@main
        with:
          read: true
          pr-number: ${{ github.event.pull_request.number }}
          workflow-run-url: "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
```
