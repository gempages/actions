# Actions

Reusable [composite GitHub Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) shared across repositories.

## Actions

### `setup-project`

Shared CI setup step: checks out the repository, installs Node.js, and installs dependencies with Yarn.

```yaml
- uses: gempages/actions/.github/actions/setup-project@main
  with:
    node-version: "22.23.2" # optional, defaults to 22.23.2
    fetch-depth: "1" # optional, defaults to 1
    ref: ${{ github.sha }} # optional, defaults to the triggering ref
    cache: "yarn" # optional, disabled by default
    cache-dependency-path: "" # optional, required when cache is set
    install: "true" # optional, set to "false" to skip yarn install
    install-args: "--frozen-lockfile" # optional, args passed to yarn install
```

### `setup-git`

Checks whether the triggering commit's author should trigger an auto deploy, then checks out the repo and configures git for an automated commit. Outputs `should_deploy` so callers can insert their own steps (gated on the same output) before pushing.

```yaml
- name: Setup Git
  id: setup_git
  uses: gempages/actions/.github/actions/setup-git@main
  with:
    token: ${{ secrets.GH_ACTION_TOKEN }}
    fetch-depth: "1" # optional, defaults to 1
    excluded_authors: '["haicaodac","HarrisonD-Seal","TheoVu197","VinhVictor","github-actions[bot]"]' # optional, this is the default

# insert any custom steps here, e.g.:
# - name: Bump version
#   if: steps.setup_git.outputs.should_deploy == 'true'
#   run: ./bump-version.sh
```

### `push-code`

Commits and pushes whatever is in the working tree as an auto deploy commit. Pair with `setup-git`.

```yaml
- name: Push code
  uses: gempages/actions/.github/actions/push-code@main
  with:
    should_deploy: ${{ steps.setup_git.outputs.should_deploy }}
    commit_message: "Auto deploy" # optional, this is the default
```

### `auto-release-extension`

Deploys a Shopify app extension for a given environment, then notifies Slack on a successful `production` release.

```yaml
- name: Deploy to Shopify (${{ github.ref_name }})
  uses: gempages/actions/.github/actions/auto-release-extension@main
  with:
    app_name: gempages # Shopify app config name, matching shopify.app.<app_name>.toml
    webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    slack_id: ${{ secrets.SLACKID }}
  env:
    SHOPIFY_APP_AUTOMATION_TOKEN: ${{ secrets.SHOPIFY_CLI_PARTNERS_TOKEN }}
```

### `auto-release-packages`

Runs `setup-project`, exits/enters changeset pre-release mode based on branch, publishes to NPM via `changesets/action`, and auto-merges the resulting Version Packages PR on `dev`/`staging`. Outputs `tag` (npm dist-tag derived from the branch) and `code` (a timestamp string for commit messages).

```yaml
- name: Auto release package
  id: autoRelease
  uses: gempages/actions/.github/actions/auto-release-packages@main
  with:
    publish-cmd: yarn release
    install-args: --immutable # optional, defaults to ""
    gh-token: ${{ secrets.GH_ACTION_TOKEN }}
    npm-token: ${{ secrets.NPM_TOKEN }}
```

### `notify`

Sends a build notification to a Slack webhook, including the repo, branch, and workflow run URL.

```yaml
- uses: gempages/actions/.github/actions/notify@main
  with:
    webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    slack_id: ${{ secrets.SLACKID }}
```

## Usage

Reference an action from another repo's workflow using `<org>/actions/.github/actions/<action>@<ref>`, pinning `<ref>` to a branch, tag, or commit SHA.
