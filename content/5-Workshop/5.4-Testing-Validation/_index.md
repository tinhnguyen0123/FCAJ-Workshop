---
title : "Testing & Validation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Overview

In this section, you will validate the deployed **AI Dungeon RPG Adventure Game** backend services to ensure that authentication, database persistence, Bedrock AI story generation, and game logic function correctly.

#### Testing Plan

1. **[5.4.1 Prepare Test Environment](5.4.1-prepare/)**
   Configure Postman / Insomnia or Unity Client environment with the deployed API Gateway endpoint and Cognito credentials.

2. **[5.4.2 Authentication & Character Creation APIs](5.4.2-auth-character-api/)**
   Verify User Registration, Login, JWT Token generation, and Character Creation endpoints.

3. **[5.4.3 Test AI Story Generation & Battle Resolution](5.4.3-test-endpoint/)**
   Validate dynamic story action prompts with AWS Bedrock and turn-based combat calculations in AWS Lambda.

4. **[5.4.4 End-to-End Game Flow Simulation](5.4.4-e2e-game-simulation/)**
   Simulate a complete player session from initial login to dynamic storytelling, combat resolution, and inventory update.
