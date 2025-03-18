# Alloy

## Install

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install --namespace monitoring alloy grafana/alloy
kubectl get pods --namespace monitoring
```

## Config

https://grafana.com/docs/alloy/latest/configure/kubernetes/

```sh
kubectl create configmap --namespace monitoring alloy-config "--from-file=config.alloy=./config.alloy"
helm upgrade --install --namespace monitoring alloy grafana/alloy -f poc.yaml
kubectl -n monitoring rollout restart daemonsets.apps alloy
kubectl -n monitoring delete cm alloy-config
```

```sh
// loki.process receives log entries from other loki components, applies one or more processing stages,
// and forwards the results to the list of receivers in the component's arguments.
loki.process "cluster_events" {
  forward_to = [loki.write.<WRITE_COMPONENT_NAME>.receiver]

  stage.static_labels {
    values = {
      cluster = "<CLUSTER_NAME>",
    }
  }

  stage.labels {
    values = {
      kubernetes_cluster_events = "job",
    }
  }
}
```

Replace the following values:

- <CLUSTER_NAME>: The label for this specific Kubernetes cluster, such as production or us-east-1.
- <WRITE_COMPONENT_NAME>: The name of your loki.write component, such as default.

## Collect

https://grafana.com/docs/alloy/latest/collect/logs-in-kubernetes/

https://grafana.com/docs/alloy/latest/reference/components/loki/loki.write/

If no tenant_id is provided, the component assumes that the Loki instance at endpoint is running in single-tenant mode and no X-Scope-OrgID header is sent.

```
loki.write "default" {
  endpoint {
    url = "http://loki-gateway.loki.svc.cluster.local/loki/api/v1/push"
  }
}
```