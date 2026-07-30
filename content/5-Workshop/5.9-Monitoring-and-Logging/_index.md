---
title: "Monitoring and Logging"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Objective

Monitor application health, deployment state, logs, metrics, queues, Lambda processing, and AWS managed services.

## Architecture context

The repository contains Kubernetes ServiceMonitor and PrometheusRule manifests for application metrics. AWS-side CloudWatch metrics and alarms must be verified from the target account before being marked as deployed.

## Kubernetes resource inspection

Use `kubectl get` for fast status checks:

```bash
kubectl get deployments,pods,svc,endpoints,hpa,pdb,jobs -n internship -o wide
kubectl get ingress internship-public -n internship
kubectl get events -n internship --sort-by=.lastTimestamp
```

Expected result:

- `backend` and `chat-service` have Ready replicas.
- `backend-outbox-dispatcher` is Ready.
- `backend-processing-worker` is either Ready or intentionally scaled to zero.
- `ai-service` is Ready when AI deployment is enabled.
- `backend-migrate` and `chat-init` have completed.

## Rollout status

```bash
kubectl rollout status deployment/backend -n internship --timeout=300s
kubectl rollout status deployment/chat-service -n internship --timeout=300s
kubectl rollout status deployment/backend-outbox-dispatcher -n internship --timeout=300s
kubectl rollout status deployment/backend-processing-worker -n internship --timeout=300s
kubectl rollout status deployment/ai-service -n internship --timeout=900s
```

Only check `ai-service` when AI deployment is enabled.

## Application logs

```bash
kubectl logs deployment/backend -n internship --tail=200
kubectl logs deployment/chat-service -n internship --tail=200
kubectl logs deployment/backend-outbox-dispatcher -n internship --tail=200
kubectl logs deployment/backend-processing-worker -n internship --tail=200
kubectl logs deployment/ai-service -n internship --tail=200
```

Use `--previous` when a container has restarted:

```bash
kubectl logs deployment/backend -n internship --previous --tail=200
```

## Describe commands

Use `kubectl describe` when readiness, scheduling, probes, or Ingress reconciliation fail:

```bash
kubectl describe pod -n internship -l app=backend
kubectl describe pod -n internship -l app=chat-service
kubectl describe hpa backend -n internship
kubectl describe ingress internship-public -n internship
```

## Health endpoints

Backend readiness checks PostgreSQL:

```bash
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/api/health/ready
```

Expected response:

```json
{"status":"ready","service":"internship-api","dependencies":{"postgres":true}}
```

Chat readiness checks Redis and DynamoDB:

```bash
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/chat/health/ready
```

Expected response:

```json
{"status":"ready","dependencies":{"redis":true,"dynamodb":true}}
```

AI readiness checks that `SAGEMAKER_ENDPOINT_NAME` is configured:

```bash
kubectl port-forward service/ai-service 18082:8010 -n internship
curl -fsS http://127.0.0.1:18082/health/ready
```

## Prometheus and Grafana

The repository defines ServiceMonitor resources for:

- `backend`
- `chat-service`
- `backend-outbox-dispatcher`
- `backend-processing-worker`

Apply when the Prometheus Operator CRDs are installed:

```bash
kubectl apply -f k8s/observability/service-monitors.yaml
kubectl apply -f k8s/observability/prometheus-rules.yaml
```

Verify:

```bash
kubectl get servicemonitor,prometheusrule -n monitoring
kubectl port-forward service/kube-prometheus-stack-prometheus 9092:9090 -n monitoring
curl -fsS "http://127.0.0.1:9092/api/v1/targets?state=active"
```

Expected application targets:

- `backend`
- `chat-service`
- `backend-outbox-dispatcher`
- `backend-processing-worker`

## Repository-defined alerts

The repository includes Prometheus alerts for:

| Alert | Purpose |
|---|---|
| `InternshipBackendDown` | Backend metrics target unavailable |
| `InternshipChatDown` | Chat metrics target unavailable |
| `InternshipChatDependencyDown` | Chat Redis or DynamoDB dependency unavailable |
| `InternshipOutboxDispatcherDown` | Dispatcher metrics target unavailable |
| `InternshipProcessingWorkerDown` | Processing worker metrics target unavailable |
| `InternshipBackend5xxHigh` | Elevated backend 5xx rate |
| `InternshipBackendP95LatencyHigh` | High backend p95 latency |
| `InternshipChatMessageFailureRatioHigh` | High chat message failure rate |
| `InternshipPodCrashLooping` | Pod CrashLoopBackOff |
| `InternshipOutboxDeadEventsPresent` | Dead outbox events need operator action |
| `InternshipOutboxPendingTooOld` | Pending outbox event age too high |
| `InternshipOutboxTransportErrors` | SQS transport errors |
| `InternshipProcessingQueueOld` | Processing queue age too high |
| `InternshipProcessingJobsFailing` | Processing jobs failing |
| `InternshipAIProviderFailures` | AI provider failures |

Status: implemented in repository manifests. The supplied AWS monitoring evidence does not yet prove that these PrometheusRule resources were applied in the production EKS cluster. Verify the CRDs and resources before marking repository-defined Prometheus alerts as deployed.

## CloudWatch Logs

The supplied CloudWatch evidence shows these log groups:

| Log group | Size | Purpose |
|---|---:|---|
| `/aws/eks/internship-prod/cluster` | 478 MB | EKS cluster logs and current central log location for EKS-side investigation |
| `/aws/lambda/internship-outbox-handler` | 15 KB | Outbox Lambda logs |
| `/aws/sagemaker/Endpoints/internship-qwen3-4b` | 15 KB | SageMaker endpoint logs |
| `/aws/vpc/flowlogs` | 551 KB | VPC Flow Logs |

There are no provided CloudWatch log groups with separate names for `backend`, `chat-service`, `backend-processing-worker`, `backend-outbox-dispatcher`, or `ai-service`. If separate workload log groups are required, configure Container Insights, Fluent Bit, or another log shipping path and then attach the exported evidence.

Check Lambda logs:

```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/internship-outbox-handler \
  --region ap-southeast-1

aws logs tail /aws/lambda/internship-outbox-handler \
  --since 1h \
  --region ap-southeast-1
```

Check EKS-related CloudWatch logs through the cluster log group unless workload-specific log groups are configured:

```bash
aws logs describe-log-groups \
  --log-group-name-prefix /aws/eks/internship-prod/cluster \
  --region ap-southeast-1

aws logs tail /aws/eks/internship-prod/cluster \
  --since 1h \
  --region ap-southeast-1
```

## AWS managed-service metrics

| Service | Useful metrics |
|---|---|
| ALB | `HTTPCode_Target_5XX_Count`, `TargetResponseTime`, healthy hosts, LCUs |
| CloudFront | requests, 4xx/5xx rate, origin latency, bytes downloaded |
| RDS | CPU, free storage, connections, read/write IOPS, replica lag if any |
| ElastiCache Redis | CPU, memory, evictions, current connections |
| DynamoDB | throttled requests, consumed read/write capacity, system errors |
| SQS | approximate visible messages, age of oldest message, DLQ visible messages |
| Lambda | errors, duration, throttles, iterator age, concurrent executions |
| SageMaker | endpoint invocation errors, model latency, overhead latency, instance utilization |

## CloudWatch alarms

One CloudWatch alarm is evidenced as deployed:

| Alarm | State | Metric | Threshold |
|---|---|---|---|
| `InternshipOutboxDLQHasMessages` | OK | SQS `ApproximateNumberOfMessagesVisible` | `>= 1` |

The following alarms remain proposed until AWS evidence is exported:

| Alarm | Recommended condition |
|---|---|
| Main queue age high | Oldest SQS message age exceeds operational threshold |
| Lambda errors | `Errors > 0` for `internship-outbox-handler` |
| Lambda throttles | `Throttles > 0` |
| ALB target 5xx high | Target 5xx count/rate above threshold |
| RDS CPU high | CPU above sustained threshold |
| RDS free storage low | Free storage below threshold |
| Redis memory high | Memory or evictions above threshold |
| SageMaker errors | Invocation/model errors above threshold |
| CloudFront 5xx high | 5xx error rate above threshold |

## Expected result

- Kubernetes deployments are Ready.
- Health endpoints pass through CloudFront.
- Logs are available for each EKS workload and Lambda.
- Prometheus can scrape application metrics when observability is installed.
- SQS, Lambda, RDS, Redis, DynamoDB, ALB, CloudFront, and SageMaker metrics are visible in CloudWatch.
- Proposed alarms are either deployed and evidenced or clearly listed as proposed.

## Common errors

| Symptom | Cause | Resolution |
|---|---|---|
| `kubectl top` has no data | Metrics server unavailable or not scraped yet | Check metrics-server rollout |
| Prometheus target down | ServiceMonitor selector or namespace mismatch | Inspect Service labels and ServiceMonitor selectors |
| Chat health not ready | Redis or DynamoDB dependency down | Check chat logs and AWS service status |
| Lambda logs absent | Function not invoked or wrong region | Confirm event-source mapping and region |
| DLQ growing | Lambda or downstream processing failure | Inspect DLQ messages and Lambda logs |

## Outcome

Monitoring is complete when operational staff can inspect Kubernetes state, logs, health endpoints, Prometheus metrics, CloudWatch metrics, Lambda logs, SQS/DLQ state, and alarms without exposing secrets.
