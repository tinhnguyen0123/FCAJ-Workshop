---
title : "Prepare Test Environment"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Setup API Testing Environment

Before making requests to the deployed backend, configure your testing tool (Postman / Insomnia / cURL) or the Unity Client.

#### Environment Variables

Create an environment in Postman with the outputs received from your CDK deployment:

- `BASE_URL`: `https://<api-id>.execute-api.<region>.amazonaws.com/prod`
- `USER_POOL_ID`: `<cognito-user-pool-id>`
- `CLIENT_ID`: `<cognito-app-client-id>`
- `ID_TOKEN`: *(Will be populated after Login request)*

#### Setting Up Request Headers

For all protected routes (`/character`, `/story/*`, `/battle/*`, `/inventory/*`), add the Authorization header:

```http
Authorization: Bearer {{ID_TOKEN}}
Content-Type: application/json
```