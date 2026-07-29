---
title: "FCAJ x Agentic AI Build Week 2026"
date: 2026-07-26
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# FCAJ x Agentic AI Build Week 2026

## Event Introduction

FCAJ x Agentic AI Build Week 2026 was a knowledge-sharing event about building Agentic AI products in a hackathon environment. Participating teams presented their real-world problems, system architectures, AWS services, development challenges, demonstrations, and lessons learned.

Unlike a traditional chatbot that mainly generates responses, the Agentic AI solutions presented at the event were designed to understand goals, create plans, use tools, retrieve trusted data, and perform actions. The four featured projects were S.H.E.P.H.E.R.D., KFC Bot Agent, Solution Architect Professional AI Native App, and SignalScout.

## Event Information

| Information | Details |
|---|---|
| Event | FCAJ x Agentic AI Build Week 2026 |
| Participation date | July 26, 2026 |
| Format | In person |
| Main topics | Agentic AI, AWS Cloud, hackathons, and product development |
| Activities | Project journeys, architecture presentations, demonstrations, and team reflections |

## 1. S.H.E.P.H.E.R.D. – Crowd Monitoring and Congestion Prediction

Team 3KA shared its 24-hour journey of building S.H.E.P.H.E.R.D., a system for monitoring crowd density, queue conditions, and potential congestion through camera footage.

The solution uses computer vision to detect and track people, then converts the results into operational information for venue staff.

Its main components include:

- YOLO and ByteTrack for person detection and tracking.
- Amazon SageMaker for model-related processing.
- Amazon Bedrock AgentCore and Strands Agent for the Agentic AI layer.
- A React dashboard for displaying crowd conditions and alerts.
- An Autonomous Monitor that continuously evaluates crowd metrics.
- An Operator Copilot that answers natural-language questions using live information.

The main challenges included maintaining reliable live video, reducing inference latency, preserving tracking between frames, selecting suitable camera positions, and keeping the project achievable within the hackathon timeline.

One of the team's strongest lessons was that a small, completed product is more valuable than a large but unfinished idea. Clear goals, defined responsibilities, and an early demonstration plan can significantly improve team performance.

## 2. KFC Bot Agent – Ordering Inside a Conversation

OneTeam presented KFC Bot Agent, an AI Agent that allows customers to place orders directly through messaging platforms such as Zalo, Messenger, and future supported channels.

The solution reduces the need to leave a conversation, open another application, create a new account, and repeat the order. Customers can describe menu items, quantities, drink sizes, and promotions using natural language.

The Agent processes an order through the following stages:

1. Understand the ordering intent.
2. Plan the required steps.
3. Search trusted product data and business rules.
4. Update the cart or apply a promotion.
5. Verify the real cart before confirmation.

An important design principle is that the language model does not decide what is true by itself. Tools and trusted business data verify products, prices, vouchers, and cart status.

The architecture was designed for extension. A new communication channel can be added through an adapter, a new business through a connector, and a new capability through an additional tool. According to the scenario presented by the team, the solution achieved an estimated end-to-end latency of 3–5 seconds and an estimated cost of approximately USD 0.006 per order.

## 3. Solution Architect Professional AI Native App

Plan V presented an AI Native App designed to support Solution Architects in requirement analysis and AWS architecture preparation.

The project addressed a common situation in which an architect must read BRDs or PRDs, extract requirements, propose architecture, create diagrams, and estimate costs within a limited timeframe. These activities are often manual and highly dependent on individual experience.

The application can:

- Analyze natural-language and structured project requirements.
- Build an initial Requirements Catalogue.
- Propose high-level architecture options.
- Support hybrid-cloud considerations and company standards.
- Generate editable Draw.io and AWS architecture diagrams.
- Produce directional AWS cost estimates for `ap-southeast-1`.
- Identify assumptions, recommendations, and missing requirements.
- Refine the result through a chat interface.
- Support automatic Infrastructure as Code generation.

The solution is not intended to replace the Solution Architect. Instead, it creates a grounded initial draft that can be reviewed and improved, rather than requiring the architect to begin from a blank page.

## 4. SignalScout – Early Detection of Corporate Strategic Changes

SignalScout is an AI-supported platform for detecting early signals of corporate restructuring, strategic change, and operational adjustment.

The system collects information from multiple sources, connects scattered evidence, analyzes financial and operational indicators, and presents its findings through dashboards, timelines, reports, and risk alerts.

Its main value propositions include:

- Detecting strategic changes early.
- Connecting separate signals into a clear narrative.
- Supporting conclusions with verifiable evidence.
- Helping organizations make Maintain, Adapt, or Accelerate decisions.
- Keeping humans responsible for final decisions.
- Supporting corporate strategy, enterprise risk, and competitive-intelligence teams.

The architecture includes services such as Amazon Bedrock, AgentCore, AWS Lambda, Amazon DynamoDB, Amazon S3, Amazon API Gateway, Amazon Cognito, Amazon CloudWatch, and AWS WAF. The team also evaluated different usage levels and proposed a more cost-efficient architecture instead of focusing only on feature development.

## Key Lessons

The four presentations demonstrated that an Agentic AI product requires more than a language model. A reliable system combines the model with trusted data, tools, business rules, monitoring, and human control.

The main lessons included:

- Begin with a specific user pain point instead of beginning with an AI model.
- Keep the MVP scope small enough to complete and demonstrate.
- An AI Agent should verify information through trusted data and tools before acting.
- The architecture should support new channels, tools, and capabilities without requiring a complete rebuild.
- Cost, latency, security, and operations should be considered during the design stage.
- Clear responsibilities and demonstration preparation are important in a hackathon.
- AI should support human decisions rather than automatically replacing human responsibility.

## Connection to the Internship Project

The event helped me understand how Agentic AI principles can be applied to the AI matching component of the internship project. Instead of allowing the model to make every decision, the system should combine parsers, structured data, recruitment criteria, and a reranker to produce results that can be reviewed and explained.

The lesson about limiting project scope also supports the decision to focus the AI matching feature on HR users. The system helps HR rank and review applicants, while the final recruitment decision remains under human control.

## Conclusion

FCAJ x Agentic AI Build Week 2026 provided practical examples of turning AI ideas into demonstrable products. Although the projects addressed different problems, they showed that Agentic AI creates meaningful value only when combined with suitable architecture, trusted data, actionable tools, and clear user requirements.

The event also demonstrated that a hackathon is not only a technology competition. It is an environment for learning how to manage scope, collaborate under pressure, solve problems quickly, and transform an idea into a working product within a limited period.