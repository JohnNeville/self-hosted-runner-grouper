# Self Hosted Runners Grouper

Self Hosted Runners Grouper will process a list of rules against the org's repos and create or update self-hosted runner groups.

This project began as a way of managing self-hosted runner groups automatically and the pattern matching was inspired by the [actions/labeler](https://github.com/actions/labeler/) project.
Using minimatch for this is probably overkill but it allows for more complex rules without needing to deal with Regex.

## Usage

### Create `.github/self-hosted-runner-grouper.yml`

Create a `.github/self-hosted-runner-grouper.yml` file with a list of groups and [minimatch](https://github.com/isaacs/minimatch) globs to match to apply the group.

The key is the name of the [self hosted runner group](https://docs.github.com/en/actions/hosting-your-own-runners/managing-access-to-self-hosted-runners-using-groups) that you want to manage (eg: "octo-runner-group", "octo-node-group") and the value is the glob string to match against a repo name (eg: `*.ts`, `dotnet/docs*`) or a match object.

#### Match Object

For more control over matching, you can provide a match object instead of a simple path glob. The match object is defined as:

```yml
- any: ['list', 'of', 'globs']
  all: ['list', 'of', 'globs']
```

One or both fields can be provided for fine-grained matching. Unlike the top-level list, the list of path globs provided to `any` and `all` must ALL match against a path for the group to be applied.

The fields are defined as follows:
* `any`: match ALL globs against ANY changed path
* `all`: match ALL globs against ALL changed paths

A simple path glob is the equivalent to `any: ['glob']`. More specifically, the following two configurations are equivalent:
```yml
group1:
- example1/*
```
and
```yml
group1:
- any: ['example1/*']
```

From a boolean logic perspective, top-level match objects are `OR`-ed together and indvidual match rules within an object are `AND`-ed. Combined with `!` negation, you can write complex matching rules.

#### Basic Examples

```yml
# Add 'doc-builder' self-hosted runner group with every repo beginning with dotnet/docs.
doc-builder:
  - dotnet/docs*

# Add 'group2' self-hosted runner group with every repo beginning with dotnet/docs.
group2: dotnet/docs*
```

#### Common Examples

TODO

### Create Workflow

Create a workflow (eg: `.github/workflows/self-hosted-runner-grouper.yml` see [Creating a Workflow file](https://help.github.com/en/articles/configuring-a-workflow#creating-a-workflow-file)) to utilize the grouper action with content:

TODO

_Note: This grants access to the `GITHUB_TOKEN` so the action can make calls to GitHub's rest API_

#### Inputs

Various inputs are defined in [`action.yml`](action.yml) to let you configure the grouper:

| Name | Description | Default |
| - | - | - |
| `org-auth-token` | A Github PAT with org admin permissions. This MUST NOT be the GITHUB_TOKEN secret and must have the `admin:org` scope. | N/A |
| `org-name` | The name of the organization that should be grouped | N/A |
| `org-repo-type` | The types of repositories to load and add to groups | N/A |
| `configuration-path` | The path to the group configuration file | `.github/self-hosted-runner-grouper.yml` |
| `should-overwrite` | Whether or not to remove non-matching repos from managed groups | true |
| `should-create-missing` | Whether or not to add new groups that are missing | true |
| `dry-run` | Simulate non-GET API calls rather than actually performing the action | false |
