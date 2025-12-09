---
title : "Amazon Bedrock & AgentCore"
date: 2025-09-24
weight : 3
chapter : false
tags: ["AWS", "CloudDay", "Vietnam", "GenAI", "AI Agents", "Events"]
pre : " <b> 4.2.3 </b> "
---
## 1. Introducing Amazon Bedrock

Amazon Bedrock delivers a comprehensive platform to **build, deploy, and scale Generative AI applications**.
Strengths: fully managed infrastructure, access to leading models, and AWS-grade security.

![](AmazonBedrockOverview.jpg)

### Core capabilities:

* **Choose the best model**: Aggregate models from multiple providers.
* **Optimize for cost, latency, and accuracy**: Balance budget and performance.
* **Securely customize with your data**: Tailor models with enterprise data.
* **Apply safety and responsible AI checks**: Enforce responsible AI guardrails.
* **Deploy and operate agents**: Run AI agents on top of Bedrock.

```cli
+-------------------+   +-------------------+   +-------------------+
|  Model Selection  |-->|  Optimization     |-->|  Customization    |
+-------------------+   +-------------------+   +-------------------+
                               |
                               v
+-------------------+   +-------------------+
| Responsible AI    |-->| Agent Deployment |
+-------------------+   +-------------------+
```

---

## 2. Broadest Model Selection

Bedrock unifies a **wide portfolio of fully managed models** from top AI companies:

![](AmazonBedrockModelProviders.jpg)

* **AI21 Labs**: JAMBA - efficient processing, long-context reasoning.
* **Amazon**: NOVA - frontier intelligence with standout performance.
* **Anthropic**: Claude - strong at reasoning and code generation.
* **Cohere**: Command R - multilingual, search, and retrieval focus.
* **Meta**: LLaMA - image and language reasoning.
* **Mistral, Kestrel**: expert-grade models.
* **Stability AI**: Stable Diffusion - image generation.
* **TwelveLabs**: video AI (Marengo, Pegasus).
* **Writer**: personalized enterprise writing.
* **Luma**: high-quality video AI.
* **Poolside (coming soon)**: enterprise-scale software engineering AI.

-> Bedrock operates as a fully managed **model marketplace** within the AWS ecosystem.

```cli
[ AI21 ]   [ Amazon ]   [ Anthropic ]   [ Cohere ]
[ Meta ]   [ Mistral ]  [ Stability ]   [ TwelveLabs ]
[ Writer ] [ Luma ]     [ Poolside ]    [ Kestrel ]
```

---

## 3. Amazon Bedrock AgentCore

AgentCore is the **foundational service** for deploying AI agents **securely and at scale**.

![](AmazonBedrockAgentCore.jpg)

### Core capabilities:

* **Deploy securely at scale**: Launch on AWS cloud infrastructure.
* **Enhance with tools and memory**: Connect to APIs, databases, external tools, and add memory.
* **Monitor**: Observe and manage agents in production environments.

```cli
+-------------------+       +-------------------+       +-------------------+
|   Deployment      | ----> |  Tools + Memory   | ----> |     Monitoring    |
| Secure & Scalable |       | Integration       |       | Observability     |
+-------------------+       +-------------------+       +-------------------+
```

---

## 4. Strategic Meaning

* **Bedrock** = neutral AI infrastructure with diverse models and no vendor lock-in.
* **AgentCore** = the operating layer for enterprise-scale AI agents.
* Helps companies **move from POC -> production** quickly and safely.

-> A critical milestone for Vietnam and the region to **adopt enterprise-grade AI agents** instead of stopping at experiments.

---

## 5. Key Insights for the Notion materials

* **Bedrock = model and tooling hub** for applied AI development.
* **AgentCore = runtime OS for AI agents** at large scale.
* Establishes an **AWS AI ecosystem** that serves both startups and enterprises.

