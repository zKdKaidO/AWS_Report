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

### Week 7 Achievements:

- Enabled the system to run in a local Kubernetes cluster created with kind.
- Enabled service communication through Kubernetes Services and internal DNS.
- Used Ingress to route external requests to the frontend, backend, and chat service.
- Added health probes so Kubernetes could identify container startup, readiness, and runtime status.
- Enabled the backend and chat service to run with multiple replicas supported by HPA and Pod Disruption Budgets.
- Used Prometheus to collect metrics, Loki to manage logs, and Tempo to store distributed traces.
- Prepared Grafana dashboards and alert rules for monitoring performance, errors, and service availability.

<!--
TODO: Add kind cluster screenshots, Kubernetes resources, pod status, HPA results, Grafana dashboards, logs, traces, alerts, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-7/
-->