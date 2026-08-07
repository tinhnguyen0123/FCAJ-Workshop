---
title: "AWS FCAJ Agent Forge - Deepdive"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: “AWS FCAJ Agent Forge - Deepdive”
### Event Information

- **Event name:** AWS FCAJ Agent Forge - Deepdive
- **Date:** August 1, 2026
- **Role:** Attendee
- **Scale:** attendees, speakers, and organizers from AWS, AWS Partner Network (APN), and the community.

---

### Event Objectives

- Provide a comprehensive overview of Agentic AI architectures and degrees of AI Agent autonomy.
- Guide the design and deployment of Level 300 production-ready AI Agents for enterprise environments.
- Deep dive into Amazon Bedrock Agent Core architecture across three core components: Runtime, Identity, and Gateway.
- Deliver hands-on practical experience on AWS to bridge theoretical concepts with real-world application.

### Speakers

- **Nghia** – Main Presenter (Deep-dive technical knowledge & Amazon Bedrock Agent Core architecture theory)
- **Hai Anh** – Co-presenter (Lead instructor for practical hands-on labs)

### Key Highlights

#### Overview of Agentic AI & Emerging Communication Protocols

- **Definition of Agentic AI**: Autonomous software systems capable of reasoning, planning multi-step actions, and executing tasks autonomously.
- **Autonomy Levels**: Ranging from simple conversational assistants to controlled deterministic workflows with human-in-the-loop, up to fully autonomous multi-agent systems.
- **Modern Protocols**: Beyond traditional HTTP/REST APIs, AI Agents utilize **MCP** (Model Context Protocol for tool integration) and **A2A** (Agent-to-Agent communication).
- **Strand SDK**: AWS's open-source SDK specifically engineered to streamline agent initialization and cloud deployment on AWS.

#### Amazon Bedrock Agent Core – Runtime Environment

- **Serverless Architecture**: Built on a pay-as-you-go model, supporting auto-scaling from real-time requests to long-running asynchronous background jobs.
- **Flexible Deployment**: Packaging options via Docker containers (ECR), compressed archives (S3), or direct console templates.
- **Firecracker MicroVM Isolation**: Complete hardware-level isolation for each user session using MicroVM technology, ensuring strong security and data isolation.
- **Versioning & Endpoints**: Full support for versioning, aliases (Dev/Prod), and canary rollouts (e.g., 5-10% traffic shifts) with seamless rollback capability.
- **Bidirectional Streaming**: Real-time two-way streaming support for multi-modal inputs including text, audio, and vision.

#### Amazon Bedrock Agent Core – Identity Layer

- **Authentication & Authorization Governance**: Features dual-layer security through Inbound Authentication (user-to-agent) and Outbound Authentication (agent-to-tools).
- **Secure Token Exchange**: Converts user JWTs into **WHAT** (Workload Access Tokens) combining agent and user identities to prevent exposure of user credentials.
- **Token Vault**: Secure encrypted storage for third-party OAuth credentials and API keys.
- **Native AWS Integration**: Seamlessly connects with AWS Cognito and IAM role policies out of the box.

#### Amazon Bedrock Agent Core – Gateway Layer

- **Centralized Middleware**: Connects hundreds of agents to thousands of tools/MCP servers without complex point-to-point mesh configurations.
- **Human-in-the-Loop**: Enables administrators to manually review, approve, or deny sensitive or out-of-bound requests.
- **Semantic Search & Tool Indexing**: Automatically discovers relevant tools based on MCP schemas and semantic query understanding.
- **Interceptors & Guardrails**: Inbound/outbound filtering mechanisms to scrub Sensitive/PII data before returning responses to clients.
- **Enterprise Network Connectivity**: Flexible deployment topologies including Public, Half-Private, and Fully Private (via AWS PrivateLink/VPC) suitable for enterprise and banking security compliance.

### Key Takeaways

#### Design Mindset

- **Balancing Autonomy & Control**: Enterprise production workloads require deterministic workflows combined with Human-in-the-Loop mechanisms rather than 100% unconstrained AI autonomy.
- **Model Selection Strategy**: Match model capabilities to task requirements (e.g., selecting between Claude Haiku, Sonnet, Opus, or Amazon Nova based on speed, cost, and reasoning demands).

#### Technical Architecture & Security

- **Multi-Layered Security**: Ensure system safety using token exchange, MicroVM isolation, and VPC PrivateLink setups.
- **Optimized Latency**: Utilize asynchronous execution and bidirectional streaming to preserve smooth user experience for complex tasks.
- **Centralized Tool Governance**: Standardize agent-tool interactions through Agent Gateway and MCP schemas.

#### Modernization Strategy

- **Safe Production Rollout**: Implement canary deployment strategies (5-10% traffic shifts) paired with CloudWatch logging and metrics for observability.
- **Cost Efficiency**: Leverage serverless compute to pay strictly for active request workloads.

### Applying to Work

- **Enterprise AI Agent Development**: Adopt Amazon Bedrock Agent Core to build, containerize, and manage AI agents rather than building raw infrastructure from scratch.
- **Adopt MCP & A2A Standards**: Standardize tool connectivity and agent-to-agent interoperability across internal microservices.
- **Implement Gateway & Identity Controls**: Configure Token Vaults, IAM policies, and Guardrail interceptors to protect sensitive enterprise data.
- **Leverage Strand SDK**: Utilize the Strand SDK to accelerate agent codebase creation and AWS cloud resource binding.

### Event Experience

Attending the **“AWS FCAJ Agent Forge - Deepdive”** workshop was exceptionally insightful, offering a comprehensive view on transitioning Generative AI applications from Proof-of-Concept (PoC) to enterprise-grade Production environments. Key highlights included:

#### Insights from Experienced Speakers
- Level 300 technical depth delivered by AWS experts focusing on production-grade architectural patterns.
- Detailed, clear explanations of complex mechanics like Token Exchange, MicroVM isolation, and the MCP protocol.

#### Hands-on Technical Exposure
- Effective blend of deep theoretical architecture with practical AWS console lab exercises.
- Practical experience in creating Runtimes, configuring Gateways, and setting up Identity layers.

#### Lessons Learned
- Building enterprise-ready agents demands rigorous standards across Security, Scalability, and Governance.
- Utilizing Amazon Bedrock Agent Core significantly accelerates time-to-market while enforcing AWS architectural best practices.

> Overall, the event provided both rich technical depth and a clear strategic roadmap for designing and operating secure, scalable AI Agent architectures in production.

### Event participation image
![Event participation image](/FCAJ-Workshop/images/AgentForge%20-Deepdive(Day1).png)