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
        uses: ajilach/notifications-ghaction@v1.0.0
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
      - name: Get Previous Workflow Status
        run: |
          echo "Previous Workflow Status: ${{ github.event.workflow_run.conclusion }}"
          echo "Previous Workflow Run ID: ${{ github.event.workflow_run.id }}"

      - name: Notify workflow status
        uses: ajilach/notifications-ghaction@v1.0.0
        with:
          op_service_account_token: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
          slack_channel_id: "C064M8AVB5G"
          workflow_completion_notification: "true"
          workflow_name: "Workflow test"
          workflow_status: "${{ github.event.workflow_run.conclusion }}"
          workflow_run_id: "${{ github.event.workflow_run.id }}"
```


## Parameters

`op_service_account_token`: the 1Password service account token. Ideally this one is saved as GitHub secret.

`slack_channel_id`: the Slack channel ID used to post the messages to.

`job_status`: should always be set to `"${{ job.status }}"` so that the action has the job context. Only needed when reporting job status.

`workflow_completion_notification`: can be set to either `"true"` or `"false"`. Note that we are passing in strings here since a composite action cannot accept boolean values. Set to `"true"` when reporting workflow status, omit otherwise (default is `"false"`).

`workflow_status`: set to the status of the other workflow for which you would like to report: `"${{ github.event.workflow_run.conclusion }}"`.

`workflow_run_id`: set to the ID of the other workflow for which you would like to report: `"${{ github.event.workflow_run.id }}"`.