---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building & Deploying AI Dungeon RPG Game Backend on AWS

![Architecture Overview](images/aws-architect-project.jpg)

#### Overview

The **AI Dungeon RPG Adventure Game** is a modern 2D Unity RPG that combines Generative AI (**AWS Bedrock**) with a scalable, cost-effective Serverless architecture on AWS (**.NET 8 Lambda**, **Amazon API Gateway**, **Amazon Cognito**, **Amazon DynamoDB**).

In this workshop, you will learn how to set up, deploy, test, and validate a production-ready Serverless backend for an AI-driven game, and integrate it with a Unity 2D Client built using the **MVP (Model-View-Presenter)** pattern.

#### Key Highlights

- **AWS Bedrock Integration:** Real-time dynamic story generation and turn-based combat resolution using Large Language Models (LLMs).
- **Serverless Infrastructure:** Highly scalable, pay-as-you-go architecture using AWS Lambda (.NET 8 runtime) and DynamoDB.
- **Server-Authoritative Security:** User authentication via AWS Cognito and server-side calculation for stats, health, combat, and loot drops to prevent client hacking.
- **Infrastructure as Code (IaC):** Automated stack deployment using AWS CDK in C#.
- **C# Full-Stack Monorepo:** Shared `GameShared` class library used by both Unity Client and AWS Lambda Backend, eliminating schema mismatches.
- **Unity MVP Architecture:** Clean separation of concerns with Model / Presenter / View pattern across all game scenes.
- **Mock / Online Mode:** Unity Client supports seamless switching between offline Mock Mode (no backend needed) and Online Mode via `GameConfigSO`.

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Detailed Deployment Guide](5.3-Detail-guide/)
4. [Testing & Validation](5.4-Testing-Validation/)
5. [Clean Up Resources](5.5-Cleanup/)
6. [References & Resources](5.6-References/)