## Implement Slack Notifications for GitHub Actions (Failures Notifications Only)

### Description

As a DevOps engineer, I want to implement Slack notifications in the GitHub Actions pipeline that trigger only on failures for the main branches of both Terraform and Packer pipelines. This will ensure that the team is promptly alerted to critical issues that need immediate attention.

### Acceptance Criteria:

**1.** Create a new slack channel using the naming convention of the existing POD B and just add a suffix of CI/CD, so the notification goes into this channel.

**2.** **Slack Integration:**

- Integrate Slack notifications into the existing GitHub Actions pipeline using a Slack webhook URL.

- Ensure notifications are only sent when the pipeline fails.

**3.** **Branch Condition:**

- Configure the notification to trigger only on failures.

**4.** **Secret Management:**

- Store the Slack webhook URL securely using GitHub Secrets.

**5.** **Notification Content:**

- The Slack message should include relevant details such as the repository name, branch name, commit SHA, and a link to the failed GitHub Actions run.

**6.** **Documentation:**

- Update project documentation to include instructions for setting up and modifying Slack notifications in the GitHub Actions pipeline


### Instructions to Implement Slack Notifications for GitHub Actions (Failures Notifications Only)


**1. Create a new slack channel:**

- On the slack workspace, Click on the **`+`** button and create a channel.
- Use the naming convention of the existing POD A and just add a suffix of CI/CD.

**2. Obtain a Webhook URL:**

- Visit the [slack api website](https://api.slack.com/)
- Click on **Your Apps** and click on **create an app**.
- In the dialogue box, select the option that creates the app from scratch.
- Name the app and select the relevant workspace
- On the slack api page, click on incoming webhooks and set it up for the newly created channel.
- Copy the webhook URL

**3. Add the Slack Webhook URL to Github Secrets:**

- Go to your repository settings on GitHub.
- Navigate to Settings > Secrets and variables > Actions.
- Click on **Add a new secret**
- Enter the Secret Name: **`SLACK_WEBHOOK_URL`** and paste the Slack Webhook URL.

**4. Update the Packer Pipeline:**

- Add steps in the pipeline to notify the team via Slack when a failure occurs on the main branch.
- The **`Notify Slack on Failure`** step is added at the end of both the **`packer-test`** and **`deploy-to-production`** jobs. It checks if the pipeline has failed and triggers a slack notification.
- The Slack Webhook URL is securely stored in GitHub Secrets and is passed into the environment variables.

**5. Usecase for Step:**

```
#  This step checks if the pipeline has failed and triggers a slack notification.  
      - name: Notify Slack on Failure
        if: failure()  # Ensures the step runs only if previous steps failed
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {
              "text": "❌ Packer Pipeline failed in repository ${{ github.repository }} on branch ${{ github.ref_name }} (commit: ${{ github.sha }}). View details: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

**i.** **Condition to Run Step:**
   -  **`if: failure()`** This condition ensures that this step will only execute if any of the previous steps in the job have failed. If all previous steps succeed, this step will be skipped.

**ii.** **Slack Notification Action:**
   - This line specifies the action to use for sending a Slack notification. The **`slackapi/slack-github-action@v1.26.0`** is a GitHub Action that integrates with Slack's API to send messages to a specified Slack channel.

**iii.** **Slack Message Payload:**

```
with:
  payload: |
    {
      "text": "❌ Terraform Pipeline failed in repository ${{ github.repository }} on branch ${{ github.ref_name }} (commit: ${{ github.sha }}). View details: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
    }
```

This section defines the content of the Slack message that will be sent if the pipeline fails:
- ${{ github.repository }}: Replaced by the repository name.
- ${{ github.ref_name }}: Replaced by the name of the branch where the failure occurred.
- ${{ github.sha }}: Replaced by the commit SHA that triggered the pipeline.
- ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}: This generates a URL that links directly to the failed GitHub Actions run, allowing easy access to view details.

**iv.** **Environment Variables**

```
env:
  SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

- The **`SLACK_WEBHOOK_URL`** environment variable is needed for the Slack action to send the message to the correct Slack channel.
- **`${{ secrets.SLACK_WEBHOOK_URL }}`**: This is a secret stored in the GitHub repository settings. It contains the URL to the Slack webhook, ensuring that the webhook URL is kept secure and is not exposed in the code.
