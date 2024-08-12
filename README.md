# notifications-ghaction
GitHub Action to deliver notifications

Currently, this [composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) posts messages to [Slack](https://slack.com) either for completing a workflow or for pointing out failing jobs.

**Note:** This action requires [1Password](https://1password.com/) for secret management.

## Usage examples

### Failed job

The following snippet calls the action to report the failed job status which occured during execution of failing `job1`.


```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - name: Failing Step
        run: exit 1
      - name: Notify workflow status
        if: always()
        uses: ajilach/notifications-ghaction@v1.4.0
        with:
          op_service_account_token: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
          slack_channel_id: "${{ inputs.slack_channel_id }}"
          job_status: "${{ job.status }}"
```

Here is a screenshot how a message on slack looks like:

![slack screenshot with a red dot and information about the failed job](./images/job1_failed_message.png "Slack message reporting failed job 1")

### Overall workflow status

Since the workflow status can only be captured in a separate workflow, create a new workflow which checks the status of the other workflow and notify accordingly:

```yaml
name: Post Hook Workflow

on:
  workflow_run:
    workflows: ["Workflow test"]
    types:
      - completed

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:

      - name: Notify workflow status
        uses: ajilach/notifications-ghaction@v1.0.0
        with:
          op_service_account_token: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
          slack_channel_id: C064M8AVB5G
          workflow_completion_notification: "true"
          workflow: ${{ github.workflow }}
          workflow_run_id: ${{ github.event.workflow_run.id }}
          repository: ${{ github.repository }}
          repository_branch: ${{ github.ref }}
          job_status: ${{ job.status }}

```


## Parameters

`op_service_account_token`: the 1Password service account token. Ideally this one is saved as GitHub secret.

`slack_channel_id`: the Slack channel ID used to post the messages to.

`job`: job to notify about. Only relevant when reporting on job level. Optional. Typically `${{ github.job }}`.

`job_status`: should always be set to `${{ job.status }}` so that the action has the job context. Optional.

`workflow_completion_notification`: can be set to either `"true"` or `"false"`. Note that we are passing in strings here since a composite action cannot accept boolean values. Set to `"true"` when reporting workflow status, omit otherwise (default is `"false"`).

`workflow`: set to the other workflow for which you would like to report: Typically `${{ github.workflow }}`.

`workflow_run_id`: set to the ID of the other workflow for which you would like to report: `${{ github.event.workflow_run.id }}`.

`repository`: set to the git repository for which you would like to report. Typically `${{ github.repository }}`.

`repository_branch`: sot to the repository's branch for which you would like to report. Typically `${{ github.ref }}`.

`deploy_flag`: set to `true` if this is a CI/CD workflow and the deploy step has been executed as well. Leave blank if this is not a CI/CD workflow.
