---
title: "Nhật ký tuần 7"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Tuần 7 - Kubernetes local và observability

### Mục tiêu tuần 7:

- Chuyển hệ thống từ Compose sang Kubernetes bằng kind.
- Mô phỏng kiến trúc tương tự EKS trong môi trường local.
- Cấu hình scaling, availability controls, metrics, logs, traces, dashboards và alerts.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Tạo kind cluster và chuẩn bị Kubernetes namespaces, configuration resources. | 20/07/2026 | 26/07/2026 | [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/); [Kubernetes Documentation - Deployments and Services](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) |
| 2 | Triển khai frontend, backend, workers, chat, dependencies và Kubernetes services. | 20/07/2026 | 26/07/2026 | [Kubernetes Documentation - Deployments and Services](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) |
| 3 | Cấu hình ingress, migration jobs, probes, HPA và PDB. | 20/07/2026 | 26/07/2026 | [Kubernetes Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/); [Kubernetes Pod Disruption Budget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/); [Kubernetes Pod Lifecycle and Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) |
| 4 | Bổ sung metrics, OpenTelemetry tracing, Prometheus và ServiceMonitor. | 20/07/2026 | 26/07/2026 | [OpenTelemetry Getting Started](https://opentelemetry.io/docs/getting-started/); [Prometheus Documentation - Overview](https://prometheus.io/docs/introduction/overview/); [Prometheus Getting Started](https://prometheus.io/docs/prometheus/latest/getting_started/) |
| 5 | Cấu hình Grafana dashboards, Loki logs, Tempo traces và alert rules. | 20/07/2026 | 26/07/2026 | [Grafana Getting Started](https://grafana.com/docs/grafana/latest/getting-started/); [Grafana Loki Documentation](https://grafana.com/docs/loki/latest/); [Grafana Tempo Documentation](https://grafana.com/docs/tempo/latest/) |

### Kết quả đạt được trong tuần:

- Hệ thống có thể chạy trong local Kubernetes.
- Services giao tiếp thông qua Kubernetes DNS.
- Backend và chat có thể chạy nhiều replicas.
- HPA, PDB và health probes hỗ trợ scaling và availability.
- Metrics, logs, traces, dashboards và alert rules được chuẩn bị.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-7/
-->
