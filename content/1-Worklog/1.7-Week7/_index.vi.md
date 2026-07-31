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

### Nội dung kỹ thuật đã triển khai:

Trong Tuần 7, tôi triển khai hệ thống container lên cụm Kubernetes cục bộ bằng kind. Mục tiêu là mô phỏng môi trường tương tự Amazon EKS và tìm hiểu cách Kubernetes quản lý việc triển khai, mạng nội bộ, khả năng mở rộng và giám sát hệ thống.

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

Namespace, ConfigMap và Secret được sử dụng để tổ chức tài nguyên và tách cấu hình khỏi container image.

### Triển khai trên Kubernetes:

Mỗi thành phần của hệ thống được triển khai dưới dạng Kubernetes Deployment và được truy cập thông qua Kubernetes Service.

Các service giao tiếp với nhau bằng DNS nội bộ của cluster thay vì địa chỉ container cố định.

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

Cách triển khai này giúp Kubernetes có thể thay thế Pod gặp sự cố mà không làm thay đổi endpoint của ứng dụng.

### Health Check và Khả năng sẵn sàng:

Startup Probe, Readiness Probe và Liveness Probe được cấu hình để Kubernetes xác định khi nào container sẵn sàng nhận request hoặc cần được khởi động lại.

<pre>
Container
     |
     +-- Startup Probe
     +-- Readiness Probe
     +-- Liveness Probe
</pre>

Horizontal Pod Autoscaler (HPA) và Pod Disruption Budget (PDB) cũng được cấu hình nhằm tăng khả năng sẵn sàng khi mở rộng hoặc bảo trì hệ thống.

### Hệ thống giám sát:

Một bộ công cụ observability được tích hợp để thu thập metrics, logs và distributed traces.

Prometheus thu thập metrics, Loki lưu logs, Tempo lưu traces và Grafana trực quan hóa toàn bộ dữ liệu.

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

Các alert rule cơ bản cũng được chuẩn bị để theo dõi trạng thái dịch vụ, lỗi và việc sử dụng tài nguyên.

### Vấn đề và cách giải quyết:

| Vấn đề | Cách giải quyết | Trạng thái |
|---|---|---|
| Docker Compose chưa phản ánh cách Kubernetes vận hành. | Chuyển hệ thống sang cụm kind. | Hoàn thành |
| Các service cần giao tiếp ổn định với nhau. | Sử dụng Kubernetes Service và DNS nội bộ. | Hoàn thành |
| Cần định tuyến request từ bên ngoài. | Cấu hình Kubernetes Ingress. | Hoàn thành |
| Container có thể nhận request trước khi sẵn sàng. | Bổ sung Startup, Readiness và Liveness Probe. | Hoàn thành |
| Nhiều replica cần đảm bảo tính sẵn sàng. | Cấu hình HPA và Pod Disruption Budget. | Hoàn thành |
| Khó theo dõi hiệu năng hệ thống. | Tích hợp Prometheus, Grafana, Loki và Tempo. | Hoàn thành |
| Cần phát hiện sớm lỗi dịch vụ. | Chuẩn bị các alert rule cơ bản. | Hoàn thành |

### Kiến thức kỹ thuật đã học:

Tuần này giúp tôi hiểu cách Kubernetes quản lý deployment, networking, health check và scaling.

Tôi cũng hiểu rằng observability là sự kết hợp giữa metrics, logs và traces để theo dõi toàn diện trạng thái của hệ thống.

### Kết quả tuần:

Đến cuối Tuần 7, hệ thống đã chạy trên cụm Kubernetes cục bộ với service discovery, health probe, Ingress routing, cấu hình autoscaling và bộ công cụ giám sát gồm Prometheus, Grafana, Loki và Tempo.

### Bài học rút ra:

Triển khai trên Kubernetes không chỉ là chạy container mà còn cần quản lý health check, service discovery, scaling, monitoring và alerting để đảm bảo hệ thống hoạt động ổn định.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung triển khai hệ thống lên AWS, kiểm tra kiến trúc cloud và thu thập runtime evidence từ môi trường thực tế.

<!--
TODO: Add kind cluster screenshots, Kubernetes resources, pod status, HPA results, Grafana dashboards, logs, traces, alerts, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-7/
-->