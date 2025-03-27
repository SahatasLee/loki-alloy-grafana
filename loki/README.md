# Loki

```sh
https://grafana.com/docs/loki/latest/get-started/deployment-modes/
```

## Install

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki -n loki -f poc.yaml
# debugs
# -config.file=/etc/loki/config/config.yaml
kubectl -n loki exec loki-2 -c loki -- cat /etc/loki/config/config.yaml
kubectl -n loki exec -it loki-2 -c loki -- /bin/sh
```

config

```yml
# DNS Services
global:
  dnsService: "coredns"
```

## Storage

https://grafana.com/docs/loki/latest/configure/storage/

```sh
storage_config:
  aws:
    # Note: use a fully qualified domain name (fqdn), like localhost.
    # full example: http://loki:supersecret@localhost.:9000
    s3: http<s>://<username>:<secret>@<fqdn>:<port>
    s3forcepathstyle: true
  tsdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/index_cache
    cache_ttl: 24h         # Can be increased for faster performance over longer query periods, uses more disk space

schema_config:
  configs:
    - from: 2020-07-01
      store: tsdb
      object_store: s3
      schema: v13
      index:
        prefix: index_
        period: 24h
```

Storage

https://community.grafana.com/t/need-help-with-loki3-and-external-minio-connection/125105/3

Create

    # bucketNames:
    #   chunks: FIXME
    #   ruler: FIXME
    #   admin: FIXME

```yaml
loki:
  storage:
    # Loki requires a bucket for chunks and the ruler. GEL requires a third bucket for the admin API.
    # Please provide these values if you are using object storage.
    # bucketNames:
    #   chunks: FIXME
    #   ruler: FIXME
    #   admin: FIXME
    type: s3
    s3:
      s3: null
      endpoint: null
      region: null
      secretAccessKey: null
      accessKeyId: null
      signatureVersion: null
      s3ForcePathStyle: false
      insecure: false
      http_config: {}
      # -- Check https://grafana.com/docs/loki/latest/configure/#s3_storage_config for more info on how to provide a backoff_config
      backoff_config: {}
      disable_dualstack: false
    # Loki now supports using thanos storage clients for connecting to object storage backend.
    # This will become the default way to configure storage in a future releases.
    use_thanos_objstore: false

    object_store:
      # Type of object store. Valid options are: s3, gcs, azure
      type: s3
      prefix: null  # Optional prefix for storage keys

      # S3 configuration (when type is "s3")
      s3:
        endpoint: null     # S3 endpoint URL
        region: null       # Optional region
        access_key_id: null      # Optional access key
        secret_access_key: null  # Optional secret key
        insecure: false       # Optional. Enable if using self-signed TLS
        sse: {}         # Optional server-side encryption configuration
        http: {}       # Optional HTTP client configuration
```

## Retention

https://grafana.com/docs/loki/latest/operations/storage/retention/

```yaml
compactor:
  working_directory: /data/retention
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
  retention_delete_worker_count: 150
  delete_request_store: gcs
schema_config:
    configs:
      - from: "2020-07-31"
        index:
            period: 24h
            prefix: index_
        object_store: gcs
        schema: v13
        store: tsdb
storage_config:
    tsdb_shipper:
        active_index_directory: /data/index
        cache_location: /data/index_cache
    gcs:
        bucket_name: loki
```

## Thanos

```sh
https://grafana.com/docs/loki/latest/configure/examples/thanos-storage-configs/
```

## Notes

```sh
** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace loki

If pods are taking too long to schedule make sure pod affinity can be fulfilled in the current cluster.

***********************************************************************
Installed components:
***********************************************************************
* loki

Loki has been deployed as a single binary.
This means a single pod is handling reads and writes. You can scale that pod vertically by adding more CPU and memory resources.


***********************************************************************
Sending logs to Loki
***********************************************************************

Loki has been configured with a gateway (nginx) to support reads and writes from a single component.

You can send logs from inside the cluster using the cluster DNS:

http://loki-gateway.loki.svc.cluster.local/loki/api/v1/push

You can test to send data from outside the cluster by port-forwarding the gateway to your local machine:

  kubectl port-forward --namespace loki svc/loki-gateway 3100:80 &

And then using http://127.0.0.1:3100/loki/api/v1/push URL as shown below:

```
curl -H "Content-Type: application/json" -XPOST -s "http://127.0.0.1:3100/loki/api/v1/push"  \
--data-raw "{\"streams\": [{\"stream\": {\"job\": \"test\"}, \"values\": [[\"$(date +%s)000000000\", \"fizzbuzz\"]]}]}" \
-H X-Scope-OrgId:foo
```

Then verify that Loki did receive the data using the following command:

```
curl "http://127.0.0.1:3100/loki/api/v1/query_range" --data-urlencode 'query={job="test"}' -H X-Scope-OrgId:foo | jq .data.result
```

***********************************************************************
Connecting Grafana to Loki
***********************************************************************

If Grafana operates within the cluster, you'll set up a new Loki datasource by utilizing the following URL:

http://loki-gateway.loki.svc.cluster.local/

***********************************************************************
Multi-tenancy
***********************************************************************

Loki is configured with auth enabled (multi-tenancy) and expects tenant headers (`X-Scope-OrgID`) to be set for all API calls.

You must configure Grafana's Loki datasource using the `HTTP Headers` section with the `X-Scope-OrgID` to target a specific tenant.
For each tenant, you can create a different datasource.

The agent of your choice must also be configured to propagate this header.
For example, when using Promtail you can use the `tenant` stage. https://grafana.com/docs/loki/latest/send-data/promtail/stages/tenant/

When not provided with the `X-Scope-OrgID` while auth is enabled, Loki will reject reads and writes with a 404 status code `no org id`.

You can also use a reverse proxy, to automatically add the `X-Scope-OrgID` header as suggested by https://grafana.com/docs/loki/latest/operations/authentication/

For more information, read our documentation about multi-tenancy: https://grafana.com/docs/loki/latest/operations/multi-tenancy/

> When using curl you can pass `X-Scope-OrgId` header using `-H X-Scope-OrgId:foo` option, where foo can be replaced with the tenant of your choice.
```