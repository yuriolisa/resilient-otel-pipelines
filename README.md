# Cloud Native Rejekts Europe - Paris  - 2024
## Building Resilient Observability Pipelines using OpenTelemetry Collector.

### Prerequisites

This tutorial requires a docker and Kubernetes cluster, refer to [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) for a local Kubernetes cluster installations and [Telemetrygen](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/cmd/telemetrygen) tool.

#### Starting Kind Cluster
```bash
 kind create cluster -n rejekts-2024 --config kind.yaml
```

#### Installing Telemetrygen
```bash
go install github.com/open-telemetry/opentelemetry-collector-contrib/cmd/telemetrygen@latest
```

#### Deploy observability backend

This tutorial uses Prometheus, Grafana,  Loki and Tempo as observability backend to store metrics, logs and traces.

Deploy the backend systems:

```bash
kubectl apply -f backend.yaml
```

Install Prometheus via Helm Chart:
```bash
helm install prometheus prometheus-community/kube-prometheus-stack --namespace prometheus --create-namespace
```

For visualisation port forward Grafana:

```bash
kubectl port-forward -n backend svc/grafana 3000:3000
```

### OpenTelemetry 

#### Installing CertManager
````bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml
````

#### Installing the OpenTelemetry Operator
````bash
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml
````

Enable the Prometheus Feature Gates
```
- --feature-gates=+operator.autoinstrumentation.go,+operator.autoinstrumentation.nginx,+operator.observability.prometheus
```

#### Deploy the OpenTelemetry Collector Local
````bash
kubectl apply -f otel-local.yaml
````

#### Deploy the OpenTelemetry Collector as Backend
````bash
kubectl apply -f otel-backend.yaml
````

#### Deploy the ServiceMonitors to send metrics to Promethus
````bash
kubectl apply -f servicemonitor.yaml
````

#### Run the Telemetrygen to generate Logs
Forward a connection to OpenTelemetry Local

kubectl port-forward svc/local-collector 4317:4317

````bash
telemetrygen logs --otlp-insecure --logs 1_000_000
````

#### Check the Logs being transmited on Grafana Dashboard

http://localhost:3000/grafana/d/fb8648c6-5488-4966-bda0-298c12512f2e/rejekts-2024?orgId=1&from=now-15m&to=now

#### Stop the Backend Instance to simulate an Outage
````bash
kubectl delete -f otel-backend.yaml
````

#### Check the Queue size incresing every second

http://localhost:3000/grafana/d/fb8648c6-5488-4966-bda0-298c12512f2e/rejekts-2024?orgId=1&from=now-15m&to=now


#### Redeploy the OTel Collector Backend
````bash
kubectl apply -f otel-backend.yaml
````

#### Check the Queue getting back to Zero.

http://localhost:3000/grafana/d/fb8648c6-5488-4966-bda0-298c12512f2e/rejekts-2024?orgId=1&from=now-15m&to=now