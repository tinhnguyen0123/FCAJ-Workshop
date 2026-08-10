---
title: "Dev Corner: Learning AWS through an AI-integrated 2D game project"
date: 2026-08-08
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

> **Original Post:** [Posted on AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2237641900334103)

When starting a 2D RPG game that interacts with AI using a Serverless architecture (Unity with a .NET 8 backend, AWS Lambda, and Amazon Bedrock), seeing the AI reply smoothly in-game feels amazing. However, as the application began to "bloat" and AI calls happened continuously, our team learned a costly lesson: If you don't calculate costs and secure the app from the get-go, the AWS bill at the end of the month will literally be a massive shock. 🥶

If you are planning to build an AI project on AWS, you might want to check out this sharing to avoid wasting money:

### 1. LLM Token Costs Multiply if You Pass the Entire Chat History

Every time a player interacts, the app has to make a call to Amazon Bedrock (using models like Claude or Llama). These models charge for both input and output tokens. If every turn sends the entire chat history from the very beginning, the prompt gets longer and longer, which means token costs will skyrocket.

**👉 Our Solution:**
Our team uses a `summary_prompt` file to compress old gameplay turns into a short summary before sending it to the AI. At the same time, fixed data like location info or character attributes are cached in a DynamoDB table, so the AI doesn't have to "remember" or deduce them from scratch.

---

### 2. Tightening IAM Security with the Least Privilege Principle

When we first built the app, for the sake of speed, I often conveniently assigned `AdministratorAccess` or `FullAccess` permissions to Lambda functions. This approach helps devs move fast initially, but it becomes a highly dangerous security flaw if deployed to a real environment.

**👉 Our Solution:**
Thanks to using Infrastructure as Code (AWS CDK), our team formed the habit of assigning minimal permissions (**Least Privilege**). Any Lambda function that only needs to read data is assigned `grantReadData()`, and any function calling Bedrock is granted exactly the `bedrock:InvokeModel` permission. This aims to limit the blast radius and minimize any security vulnerabilities the system might encounter.

![IAM Role Policy](images/iam-role-policy.png)
*(Illustration of detailed permission policies for Lambda)*

---

### 3. Setting Up AWS Budgets and CloudWatch Alarms Before Writing the First Line of Code

Many devs new to the Cloud can easily swallow a "bitter pill" of wasted money by accidentally creating an infinite loop calling an API or forgetting to delete test resources.

**👉 Our Solution:**
A hard-earned lesson is to set up **AWS Budgets** right from the start to receive email alerts. For instance, configuring it so that if the cost hits a $5 or $10 threshold, the system immediately sends a warning email.

Furthermore, setting up **CloudWatch Alarms** to monitor the number of requests and the error rate of Lambda is absolutely essential. It helps the team detect early if the app gets stuck in an infinite loop or receives spam requests from the outside.

![AWS Budgets & Cost Explorer](images/cost-explorer-forecast.png)
*(Monthly cost forecast using Cost Explorer)*

![CloudWatch Metrics](images/cloudwatch-bedrock-nova-pro-v1.png)
*(Monitoring Requests and Errors on CloudWatch for timely handling)*

---

> *These lessons might sound theoretical, but you truly have to build it yourself and experience the "anxious feeling" over a bill to remember it for a long time. I'm also just in the beginning stages of getting familiar with the AWS ecosystem, so I'm sure the system still has plenty of room for optimization. I hope these practical experiences are somewhat helpful to you!*