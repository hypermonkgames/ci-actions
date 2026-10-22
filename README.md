# ci-actions

Shared GitHub Actions used across HyperMonk repositories.

## Actions

| Action | Description |
| --- | --- |
| [`notify`](notify/action.yml) | Posts or updates a CI status message in Slack through eldermonk. |

## notify

Sends a CI status message to a Slack channel via the eldermonk `POST /api/v1/ci/notify` endpoint. Calling it again within the same job updates the existing message instead of posting a new one.

Notification failures never fail the build: the action logs a warning and exits successfully.

### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `credentials` | yes | | JSON with the eldermonk URL and token: `{"url":"https://...","token":"..."}`. Store it as the `ELDERMONK_CREDENTIALS` secret. |
| `channel` | yes | | Slack channel ID. |
| `status` | yes | | `requested`, `started`, `success`, `failed`, `aborted`, or a job status (`success`, `failure`, `cancelled`). |
| `title` | no | `""` | Message title. Empty uses the server default for the status. |
| `mention-email` | no | `""` | Email of the user to mention. Empty uses the author of the commit that triggered the run. Bot and `@users.noreply.github.com` addresses are skipped. |
| `fields` | no | `[]` | Extra fields as JSON, e.g. `[{"name":"Package","value":"x"}]`. Appended after the default fields. |
| `links` | no | `[]` | Extra links as JSON, e.g. `[{"name":"Docs","url":"https://..."}]`. Appended after the run link. |

Default fields: Repository, Branch, Commit, Message (subject of the triggering commit), Triggered by. Default link: Run.

### Outputs

| Name | Description |
| --- | --- |
| `ref-channel` | Channel of the message, to be reused for updates. |
| `ref-timestamp` | Timestamp of the message, to be reused for updates. |

The same values are also exported as the `ELDERMONK_SLACK_CHANNEL` and `ELDERMONK_SLACK_TS` environment variables, so later `notify` steps in the same job update the message automatically.

### Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hypermonkgames/ci-actions/notify@main
        with:
          credentials: ${{ secrets.ELDERMONK_CREDENTIALS }}
          channel: C0123456789
          status: started

      # ... build steps ...

      - uses: hypermonkgames/ci-actions/notify@main
        if: always()
        with:
          credentials: ${{ secrets.ELDERMONK_CREDENTIALS }}
          channel: C0123456789
          status: ${{ job.status }}
          fields: '[{"name":"Package","value":"my-package"}]'
```

### Requirements

The runner needs `bash`, `curl` and `jq` (all present on GitHub-hosted runners). The checkout step is required for the default commit-author mention.
