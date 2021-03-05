# Org Self Hosted Runners Grouper

Org Self Hosted Runners grouper triages PRs based on the paths that are modified in the PR.

## Usage

### Create `.github/action-grouper.yml`

Create a `.github/action-grouper.yml` file with a list of groups and [minimatch](https://github.com/isaacs/minimatch) globs to match to apply the group.

The key is the name of the group in your repository that you want to add (eg: "merge conflict", "needs-updating") and the value is the path (glob) of the changed files (eg: `src/**/*`, `tests/*.spec.js`) or a match object.

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
# Add 'group1' to any changes within 'example' folder or any subfolders
group1:
  - example/**/*

# Add 'group2' to any file changes within 'example2' folder
group2: example2/*
```

#### Common Examples

```yml
# Add 'repo' group to any root file changes
repo:
  - ./*
  
# Add '@domain/core' group to any change within the 'core' package
@domain/core:
  - package/core/*
  - package/core/**/*

# Add 'test' group to any change to *.spec.js files within the source dir
test:
  - src/**/*.spec.js

# Add 'source' group to any change to src files within the source dir EXCEPT for the docs sub-folder
source:
- any: ['src/**/*', '!src/docs/*']

# Add 'frontend` group to any change to *.js files as long as the `main.js` hasn't changed
frontend:
- any: ['src/**/*.js']
  all: ['!src/main.js']
```

### Create Workflow

Create a workflow (eg: `.github/workflows/action-grouper.yml` see [Creating a Workflow file](https://help.github.com/en/articles/configuring-a-workflow#creating-a-workflow-file)) to utilize the grouper action with content:

```
name: "Org Self Hosted Runners Grouper"
on:
- pull_request_target

jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/grouper@main
      with:
        repo-token: "${{ secrets.GITHUB_TOKEN }}"
```

_Note: This grants access to the `GITHUB_TOKEN` so the action can make calls to GitHub's rest API_

#### Inputs

Various inputs are defined in [`action.yml`](action.yml) to let you configure the grouper:

| Name | Description | Default |
| - | - | - |
| `repo-token` | Token to use to authorize group changes. Typically the GITHUB_TOKEN secret | N/A |
| `configuration-path` | The path to the group configuration file | `.github/action-grouper.yml` |
| `sync-groups` | Whether or not to remove groups when matching files are reverted or no longer changed by the PR | `false`
