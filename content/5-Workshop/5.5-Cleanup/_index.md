---
title : "Clean Up Resources"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Destroying AWS Infrastructure & Avoiding Unnecessary Charges

Congratulations on completing the **AI Dungeon RPG Adventure Game** workshop!

To prevent incurring unexpected AWS charges for resources provisioned during this lab, follow these cleanup steps to remove all stacks and resources.

---

#### Step 1: Destroy AWS CDK Stack

1. Open your terminal and navigate to the infrastructure directory:
   ```bash
   cd infrastructure
   ```

2. Execute the CDK destroy command:
   ```bash
   cdk destroy --all
   ```

3. Confirm the deletion prompt by entering `y`.

---

#### Step 2: Delete CloudWatch Log Groups

1. Open the [AWS CloudWatch Console](https://console.aws.amazon.com/cloudwatch/).
2. In the left navigation pane, select **Log groups**.
3. Search for log groups matching `/aws/lambda/GameBackend-*` and `/aws/apigateway/*`.
4. Select the matching log groups, click **Actions**, and choose **Delete log group(s)**.

---

#### Step 3: Verify DynamoDB & Cognito Pool Cleanup

1. Open the [Amazon DynamoDB Console](https://console.aws.amazon.com/dynamodb/) to ensure tables (`Users`, `Characters`, `StorySessions`, `BossEncounters`) have been deleted.
2. Open the [Amazon Cognito Console](https://console.aws.amazon.com/cognito/) to confirm the User Pool has been deleted.