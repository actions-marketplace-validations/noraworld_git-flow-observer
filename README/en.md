# git-flow observer

| **English** | [日本語](/README/ja.md) |
| ----------- | ---------------------- |

git-flow observer is a CI tool made with GitHub Actions that allows you to check your project is in correct operation as git-flow.



## How it works
git-flow observer basically checks whether or not a head branch (in most cases this is a `develop` branch) is fast-forwarded compared to a base branch (in most cases this is a `main` branch.) It will pass if a head branch is fast-forwarded. Otherwise it will fail.

Here are some cases.

* It will fail when creating a pull request without a back-merge after merging a pull request...
    * into a head branch (`develop`)
    * from a release branch into a base branch (`main`) with additional commits
    * from a hotfix branch into a base branch (`main`)
    * with cherry-pick commits
* It will be skipped when creating a pull request from a base branch (`main`) into a head branch (`develop`) (a back-merge PR) [^1]

[^1]: Only in case you set `if: github.head_ref != 'main' || github.base_ref != 'develop'` in your workflow YAML.

You will be able to realize your project isn’t in correct operation as git-flow if it fails.



## Installation
It’s super easy to install git-flow observer to your GitHub repository. All you have to do is add the following YAML code to your repository as `.github/workflows/git-flow-observer.yml`.

```yaml
# .github/workflows/git-flow-observer.yml

name: "git-flow observer"

on: pull_request

jobs:
  git-flow-observer:
    if: github.head_ref != 'main' || github.base_ref != 'develop'
    runs-on: ubuntu-latest
    steps:
      - name: "Observe"
        uses: noraworld/git-flow-observer@latest
        with:
          head: "develop"
          base: "main"
```

Replace a part of the above YAML code with the following.

| Key                                         | Description                                                         | Required | Sample Value                                                |
| ------------------------------------------- | ------------------------------------------------------------------- | :------: | ----------------------------------------------------------- |
| `jobs.git-flow-observer.if`                 | Specify conditions when to skip a job                               | False    | `github.head_ref != 'main' \|\| github.base_ref != 'develop'` |
| `jobs.git-flow-observer.steps[*].with.head` | Specify a branch name as basis for comparison                       | True     | `"develop"`                                                 |
| `jobs.git-flow-observer.steps[*].with.base` | Specify a branch name that is checked whether or not fast-forwarded | True     | `"main"`                                                    |

HINT: You may need to set a base branch as `"master"` instead of `"main"`.

HINT: You setting up a `if: github.head_ref != 'main' || github.base_ref != 'develop'` statement, it will be skipped when a pull request from a base branch into a head branch. This may help fix the status check failure easier.

That’s done!



## Common questions
### What if it fails?

1. Make a head branch fast-forwarded

First, you need to make a head branch fast-forwarded. In most cases, it can be resolved by creating and merging a new pull request from a base branch into a head branch.

![Merge from main into develop](/screenshots/merge_from_main_into_develop_pr.png)

2. Re-run jobs

Then re-run jobs referring to the following screenshots.

![Failed CI details](/screenshots/failed_ci_details.png)

![Re-run jobs button](/screenshots/rerun_jobs_button.png)

![Re-run jobs dialog](/screenshots/rerun_jobs_dialog.png)

### How can I force PRs to require status checks to pass before merging?

You can set this up in your repository settings.

1. Head over to `Settings` (in your repository) > `Branches`, and click `Add rule`

HINT: `Settings` is in your repository, not your account settings.

2. Enter a branch name pattern you want to set up, enable `Require status checks to pass before merging`, and add git-flow observer to the status checks list

![Branch protection rule](/screenshots/branch_protection_rule.png)

3. Click a `Create` button at the bottom of the page

4. Repeat the procedure 1 through 3 for every branch

HINT: It is recommended to set up in both your head and base branches.



## License
All codes of this repository are available under the MIT license. See the [LICENSE](LICENSE) for more information.
