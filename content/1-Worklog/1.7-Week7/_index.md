---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Week 7 - Local Kubernetes Deployment and Observability

### Week 7 Objectives:

- Move the system from Compose to Kubernetes using kind.
- Simulate an EKS-like architecture locally.
- Configure scaling, availability controls, metrics, logs, traces, dashboards, and alerts.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Create a kind cluster and prepare Kubernetes namespaces and configuration resources. | 20/07/2026 | 26/07/2026 | [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/); [Kubernetes Documentation - Deployments and Services](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) |
| 2 | Deploy frontend, backend, workers, chat, dependencies, and Kubernetes services. | 20/07/2026 | 26/07/2026 | [Kubernetes Documentation - Deployments and Services](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) |
| 3 | Configure ingress, migration jobs, probes, HPA, and PDB. | 20/07/2026 | 26/07/2026 | [Kubernetes Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/); [Kubernetes Pod Disruption Budget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/); [Kubernetes Pod Lifecycle and Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) |
| 4 | Add metrics, OpenTelemetry tracing, Prometheus, and ServiceMonitor. | 20/07/2026 | 26/07/2026 | [OpenTelemetry Getting Started](https://opentelemetry.io/docs/getting-started/); [Prometheus Documentation - Overview](https://prometheus.io/docs/introduction/overview/); [Prometheus Getting Started](https://prometheus.io/docs/prometheus/latest/getting_started/) |
| 5 | Configure Grafana dashboards, Loki logs, Tempo traces, and alert rules. | 20/07/2026 | 26/07/2026 | [Grafana Getting Started](https://grafana.com/docs/grafana/latest/getting-started/); [Grafana Loki Documentation](https://grafana.com/docs/loki/latest/); [Grafana Tempo Documentation](https://grafana.com/docs/tempo/latest/) |

### Week 7 Achievements:

- The system can run in local Kubernetes.
- Services communicate through Kubernetes DNS.
- Backend and chat can run multiple replicas.
- HPA, PDB, and health probes support scaling and availability.
- Metrics, logs, traces, dashboards, and alert rules are prepared.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-7/
-->
