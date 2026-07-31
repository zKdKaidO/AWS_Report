---
title: "Week 7 Worklog"
date: 2026-07-20
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Week 7 - Local Kubernetes Deployment and System Observability

### Week 7 Objectives:

- Move the system runtime environment from Docker Compose to Kubernetes using kind.
- Simulate the organization and operation of containerized applications in an environment similar to Amazon EKS.
- Configure routing, health checks, scaling, and availability controls.
- Add metrics, logs, distributed tracing, dashboards, and alert rules for system monitoring.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Created a local Kubernetes cluster with kind, prepared namespaces, ConfigMaps, and Secrets, and deployed the frontend, backend, worker, chat service, dependencies, and required Kubernetes Services. | 20/07/2026 | 20/07/2026 | [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/); [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/); [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/) |
| 2 | Configured Ingress routing, created a migration job, and added startup, readiness, and liveness probes; configured HPA and Pod Disruption Budgets for the backend and chat service. | 22/07/2026 | 22/07/2026 | [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/); [Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/); [Pod Disruption Budget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/); [Configure Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) |
| 3 | Integrated OpenTelemetry, Prometheus, and ServiceMonitor for metrics and traces; configured Grafana dashboards, Loki logs, Tempo traces, and basic alert rules for the system. | 24/07/2026 | 24/07/2026 | [OpenTelemetry Getting Started](https://opentelemetry.io/docs/getting-started/); [Prometheus Overview](https://prometheus.io/docs/introduction/overview/); [Grafana Getting Started](https://grafana.com/docs/grafana/latest/getting-started/); [Grafana Loki](https://grafana.com/docs/loki/latest/); [Grafana Tempo](https://grafana.com/docs/tempo/latest/) |

### Technical Implementation:

During Week 7, I deployed the containerized system to a local Kubernetes cluster using kind. The goal was to simulate an environment similar to Amazon EKS while learning how Kubernetes manages application deployment, networking, scaling, and monitoring.

<pre>
kind Cluster
      |
      +-- Frontend
      +-- Backend
      +-- Worker
      +-- Chat Service
      +-- PostgreSQL
      +-- Redis
</pre>

Namespaces, ConfigMaps, and Secrets were used to organize application resources and separate configuration from container images.

### Kubernetes Deployment:

Each application component was deployed as a Kubernetes Deployment and exposed through a Kubernetes Service.

Services communicate through the cluster's internal DNS instead of fixed container addresses.

<pre>
Ingress
    |
    +-- Frontend Service
    +-- Backend Service
    +-- Chat Service
            |
            v
        Application Pods
</pre>

This deployment model makes it easier to replace failed Pods without changing application endpoints.

### Health Checks and Availability:

Startup, readiness, and liveness probes were configured so Kubernetes could determine when containers were ready to receive traffic and when they should be restarted.

<pre>
Container
     |
     +-- Startup Probe
     +-- Readiness Probe
     +-- Liveness Probe
</pre>

Horizontal Pod Autoscaler (HPA) and Pod Disruption Budgets (PDB) were also configured to improve application availability during scaling and maintenance.

### Observability Stack:

An observability stack was integrated to collect metrics, logs, and distributed traces.

Prometheus collects application metrics, Loki stores logs, Tempo records traces, and Grafana visualizes the collected information.

<pre>
Application
      |
      +-- Metrics --> Prometheus
      +-- Logs -----> Loki
      +-- Traces ---> Tempo
      |
      v
Grafana Dashboard
</pre>

Basic alert rules were prepared to monitor service health, response failures, and resource usage.

### Problems and Solutions:

| Problem | Resolution | Status |
|---|---|---|
| Docker Compose did not reflect Kubernetes deployment behavior. | Migrated services to a local kind cluster. | Completed |
| Applications required stable internal communication. | Used Kubernetes Services and internal DNS. | Completed |
| External traffic needed centralized routing. | Configured Kubernetes Ingress. | Completed |
| Containers could receive traffic before becoming ready. | Added startup, readiness, and liveness probes. | Completed |
| Multiple replicas required availability protection. | Configured HPA and Pod Disruption Budgets. | Completed |
| System performance was difficult to observe. | Integrated Prometheus, Grafana, Loki, and Tempo. | Completed |
| Service failures needed early notification. | Added basic alert rules. | Completed |

### Technical Knowledge Gained:

This week helped me understand how Kubernetes manages deployments, networking, health checks, and scaling.

I also learned how observability combines metrics, logs, and traces to provide a complete view of application behavior.

### Weekly Results:

By the end of Week 7, the application was running inside a local Kubernetes cluster with service discovery, health probes, Ingress routing, autoscaling configuration, and an observability stack based on Prometheus, Grafana, Loki, and Tempo.

### Lessons Learned:

Running applications on Kubernetes requires more than containerization. Health checks, service discovery, scaling, monitoring, and alerting are all important for maintaining system reliability.

### Next Week Plan:

The next week will focus on deploying the application to AWS services, validating the cloud architecture, and collecting runtime evidence from the deployed environment.

<!--
TODO: Add kind cluster screenshots, Kubernetes resources, pod status, HPA results, Grafana dashboards, logs, traces, alerts, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-7/
-->