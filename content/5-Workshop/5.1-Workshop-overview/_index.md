---
title : "Workshop Overview"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Architecture Overview

The **AI Dungeon RPG Adventure Game** architecture decouples the Unity 2D Game Client from the AWS Cloud Backend to ensure security, high performance, and minimal operational overhead.

```text
[ Unity 2D Client ]
       │ (REST APIs + JWT Token)
       ▼
[ Amazon API Gateway ] ◄───► [ Amazon Cognito (User Pool) ]
       │
       ▼
[ AWS Lambda Functions (.NET 8) ]
   ├── Auth & Character Service
   ├── Story & Prompt Builder Service ◄───► [ AWS Bedrock (Claude / Nova) ]
   ├── Battle & Boss Combat Resolver
   └── Inventory & Item Management
       │
       ▼
[ Amazon DynamoDB Tables ]
```

#### Core Components

1. **Unity 2D Game Client:**
   - User Interface for Authentication, Character Selection, Dynamic Story Dialogue, and Turn-based Battle.
   - Shares C# DTOs and Domain Models with the Backend via a `shared` C# class library.

2. **Amazon API Gateway & Amazon Cognito:**
   - API Gateway acts as the single entry point for all game endpoints.
   - Amazon Cognito manages user registration, login authentication, and issues JWT tokens.

3. **AWS Lambda (.NET 8):**
   - High-performance, serverless backend compute handling authentication, character state management, item inventory, battle resolution, and AI prompt building.

4. **AWS Bedrock:**
   - Serves as the AI Dungeon Master. Generates immersive storylines, dynamically evaluates player choices, and crafts vivid combat narratives in real-time.

5. **Amazon DynamoDB:**
   - Ultra-fast, single-digit millisecond latency NoSQL database storing Users, Characters, Items, Story Sessions, and Boss Encounters.

6. **AWS CDK (C#):**
   - Defines the entire cloud infrastructure as code in C#, enabling repeatable and reliable deployments.