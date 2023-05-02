# Pull Request Title Checker

This is a fork of https://github.com/thehanimo/pr-title-checker to allow for organization-wide actions.

<!-- prettier-ignore -->
This action checks if PR titles conform to the Contribution Guidelines :ballot_box_with_check: <br/><br/>
Consistent title names help maintainers organise their projects better :books: <br/><br/>
Shows if the author has _reaaaaally_ read the Contribution Guidelines :P

## Usage

### Usage as organization-wide action

This repo contains its config embedded.

To use it in your organization requires two steps:
1. create a repo containing a yaml file {{pr-title-lint-blocking.yaml}}:
```yaml
name: 'pr-title-lint-blocking'
on:
  pull_request_target:
    types:
      - opened
      - edited
      - synchronize
      - labeled
      - unlabeled

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - name: check
        id: check
        uses: alexschwartz/pr-title-checker-global@v1.3.7.6
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          pass_on_octokit_error: false
  explain:
    runs-on: ubuntu-latest
    if: ${{ failure() }}
    needs: check
    steps:
      - name: explain
        uses: thollander/actions-comment-pull-request@v2
        with:
          comment_tag: pre-title-comment
          message: |
            Hello emnify contributor ! :wave: 
            
            as of March 2023 there is a little change: 
            please ensure the jira story or task is mentioned in your **_PR title_** and the **_merge commit_**.
    
            Example: `[TLC-42] more details in error message`

            Please see https://emnify.atlassian.net/l/cp/137qAdn1 for details.
```
2. reference this file in the global config of your github enterprise under https://github.com/organizations/myorg/settings/actions "Required workflows"

### Standard Usage

Create a config file `.github/pr-title-checker-config.json` like this one below:

```json
{
  "LABEL": {
    "name": "title needs formatting",
    "color": "EEEEEE"
  },
  "CHECKS": {
    "prefixes": ["fix: ", "feat: "],
    "regexp": "docs\\(v[0-9]\\): ",
    "regexpFlags": "i",
    "ignoreLabels" : ["dont-check-PRs-with-this-label", "meta"]
  },
  "MESSAGES": {
    "success": "All OK",
    "failure": "Failing CI test",
    "notice": ""
  }
}
```
You can pass in one of `prefixes` or `regexp` or even both based on your use case. `regexpFlags` and `ignoreLables` are optional fields.

If `LABEL.name` is set to `""`, adding or removing labels will be skipped. The CI test will continue to pass/fail accordingly.

If none of the checks pass, a label will be added to that pull request. \
If at least one of them passes, the label will be removed.

This action causes CI tests to fail by default. However, if you do not want CI tests failing just because of this action, simply set `alwaysPassCI` as true in the CHECKS field. **An invalid config file will always cause the action to fail.**

Also, adding label names to the optional `ignoreLabels` field will forfeit any checks for PRs with those labels.

## Create Workflow

Create a workflow (eg: `.github/workflows/pr-title-checker.yml` see [Creating a Workflow file](https://help.github.com/en/articles/configuring-a-workflow#creating-a-workflow-file)) to utilize the pr-title-checker action with content:

```yaml
name: "PR Title Checker"
on:
  pull_request_target:
    types:
      - opened
      - edited
      - synchronize
      - labeled
      - unlabeled

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: thehanimo/pr-title-checker@v1.3.7
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          pass_on_octokit_error: false
          configuration_path: ".github/pr-title-checker-config.json"
```
NOTE:
* [`pull_request_target`](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#pull_request_target) event trigger should be used (not [`pull_request`](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#pull_request)) in order to support checking PRs from forks. This was added in `v1.3.2`. See [#8](https://github.com/thehanimo/pr-title-checker/issues/8).
* `pass_on_octokit_error` is an optional input which defaults to false. Setting it to true will prevent the CI from failing when an octokit error occurs. This is useful when the environment this action is run in is not consistent. For e.g, it could be a missing GITHUB_TOKEN. Thanks to [@bennycode](https://github.com/bennycode) for pointing this out.

* `configuration_path` is also an optional input which defaults to `".github/pr-title-checker-config.json"`. If you wish to store your config file elsewhere, pass in the path here.
