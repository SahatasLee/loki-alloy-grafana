# Loki Alloy Grafana

Monitor Logs by Loki Alloy Grafana

1. [Installing](#installing)
2. [Installing Alloy](#installing-alloy)
3. [Add Loki data source in grafana](#adding-a-loki-data-source-in-grafana) 

## Installing

```sh
# add helm repo
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm search repo grafana/loki

# install loki
helm upgrade --install --namespace monitoring loki grafana/loki --version 6.29.0 -f loki-with-external-s3.yaml

# install alloy
# step 1 edit config.alloy
vi config.alloy
# step 2 create configmap
kubectl create configmap --namespace monitoring alloy-config "--from-file=config.alloy=./config.alloy"
# step 3 deploy alloy
helm upgrade --install --namespace monitoring alloy grafana/alloy --version v1.0.1 -f poc.yaml
```

## Installing Alloy

Follow these steps to deploy **Alloy** in the desired cluster using Helm:

### Step 1: Create the Alloy Configuration File

Edit or create the `config.alloy` file.

```bash
vi config.alloy
```

### Step 2: Create a ConfigMap for the Alloy Configuration

```bash
kubectl create configmap alloy-config \
  --namespace monitoring \
  --from-file=config.alloy=./config.alloy
```

> This makes the configuration available to Alloy as a Kubernetes ConfigMap.

### Step 3: Deploy Alloy Using Helm

```bash
helm upgrade --install alloy grafana/alloy \
  --namespace monitoring \
  --version v1.0.1 \
  -f poc.yaml
```

> Ensure that `poc.yaml` includes references to the `alloy-config` ConfigMap and any other required values (like image settings, resources, and mount paths).

## Adding a Loki Data Source in Grafana

1. **Navigate to Data Sources:**

   * In the Grafana sidebar, go to **Configuration > Data Sources**.
   * Click **Add data source**.

2. **Select Loki:**

   * From the list of available data sources, choose **Loki**.

3. **Configure HTTP Headers:**

   * Scroll down to the **HTTP Headers** section.

    ![Select Loki](/imgs/lokidatasource1.png)

   * Add a custom header with the following details:

     * **Header name:** `X-Scope-OrgID`
     * **Value:** `$tenant_id`
     > `$tenant_id` is from config.alloy

        ```sh
        # config.alloy
        loki.write "default" {
        endpoint {
            // edit
            url = "http://loki-gateway.loki.svc.cluster.local/loki/api/v1/push"
            // Require if you enable multi tenant loki
            tenant_id = "1"
        }
        }
        ```

    ![HTTP Headers Configuration](/imgs/lokidatasource2.png)

4. **Save & Test:**

   * Click **Save & Test** to validate the connection.
