# Actions

Reusable [composite GitHub Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) shared across repositories.

## Actions

### `setup-project`

Shared CI setup step: checks out the repository, installs Node.js, and installs dependencies with Yarn.

```yaml
- uses: gempages/gem-fe-actions/.github/actions/setup-project@main
  with:
    node-version: "24.14.0" # optional, defaults to 24.14.0
    fetch-depth: "1" # optional, defaults to 1
    ref: ${{ github.sha }} # optional, defaults to the triggering ref
    cache: "yarn" # optional, disabled by default
    cache-dependency-path: "" # optional, required when cache is set
    install: "true" # optional, set to "false" to skip yarn install
    install-args: "--frozen-lockfile" # optional, args passed to yarn install
```

### `notify`

Sends a build notification to a Slack webhook, including the repo, branch, and workflow run URL.

```yaml
- uses: gempages/gem-fe-actions/.github/actions/notify@main
  with:
    webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    slack_id: ${{ secrets.SLACKID }}
```

## Usage

Reference an action from another repo's workflow using `<org>/gem-fe-actions/.github/actions/<action>@<ref>`, pinning `<ref>` to a branch, tag, or commit SHA.
