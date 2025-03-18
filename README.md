# Loki Alloy Grafana

## Install

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
# install loki
helm upgrade --install --namespace monitoring loki grafana/loki --versions 3.4.2 -f poc.yaml
# install alloy
# need to edit config.alloy
vi config.alloy
kubectl create configmap --namespace monitoring alloy-config "--from-file=config.alloy=./config.alloy"
helm upgrade --install --namespace monitoring alloy grafana/alloy --versions v1.7.1 -f poc.yaml
```