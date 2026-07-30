---
title: "Nhật ký tuần 7"
date: 2026-07-20
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Tuần 7 - Triển khai Kubernetes local và xây dựng hệ thống observability

### Mục tiêu tuần 7:

- Chuyển môi trường chạy hệ thống từ Docker Compose sang Kubernetes bằng kind.
- Mô phỏng cách tổ chức và vận hành ứng dụng container tương tự môi trường Amazon EKS.
- Thiết lập các thành phần hỗ trợ routing, health check, scaling và availability.
- Bổ sung metrics, logs, distributed tracing, dashboards và alert rules để theo dõi hệ thống.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Tạo Kubernetes cluster local bằng kind, chuẩn bị namespaces, ConfigMaps và Secrets; triển khai frontend, backend, worker, chat service cùng các dependencies và Kubernetes Services cần thiết. | 20/07/2026 | 20/07/2026 | [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/); [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/); [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/) |
| 2 | Cấu hình Ingress để định tuyến request, tạo migration job và bổ sung startup, readiness, liveness probes; thiết lập HPA và Pod Disruption Budget cho backend và chat service. | 22/07/2026 | 22/07/2026 | [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/); [Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/); [Pod Disruption Budget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/); [Configure Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) |
| 3 | Tích hợp OpenTelemetry, Prometheus và ServiceMonitor để thu thập metrics và traces; cấu hình Grafana dashboards, Loki logs, Tempo traces và các alert rules cơ bản cho hệ thống. | 24/07/2026 | 24/07/2026 | [OpenTelemetry Getting Started](https://opentelemetry.io/docs/getting-started/); [Prometheus Overview](https://prometheus.io/docs/introduction/overview/); [Grafana Getting Started](https://grafana.com/docs/grafana/latest/getting-started/); [Grafana Loki](https://grafana.com/docs/loki/latest/); [Grafana Tempo](https://grafana.com/docs/tempo/latest/) |

### Kết quả đạt được trong tuần:

- Hệ thống có thể chạy trong Kubernetes cluster local bằng kind.
- Các service giao tiếp với nhau thông qua Kubernetes Services và DNS nội bộ.
- Ingress hỗ trợ điều hướng request từ bên ngoài vào frontend, backend và chat service.
- Health probes giúp Kubernetes xác định trạng thái khởi động, sẵn sàng và hoạt động của từng container.
- Backend và chat service có thể chạy nhiều replicas và được hỗ trợ bởi HPA cùng Pod Disruption Budget.
- Prometheus thu thập metrics, Loki quản lý logs và Tempo lưu distributed traces.
- Grafana dashboards và alert rules hỗ trợ theo dõi hiệu năng, lỗi và tình trạng hoạt động của hệ thống.

<!--
TODO: Add kind cluster screenshots, Kubernetes resources, pod status, HPA results, Grafana dashboards, logs, traces, alerts, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-7/
-->