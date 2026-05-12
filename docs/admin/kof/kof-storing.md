# Storing KOF data

## Overview

KOF data (metrics, logs, traces) can be collected from each cluster and stored in specific places:

```mermaid
sequenceDiagram
    Child cluster->>Regional cluster: KOF data of the<br>child cluster<br>is stored in the<br>regional cluster.
    Regional cluster->>Regional cluster: KOF data of the<br>regional cluster<br>is stored in the same<br>regional cluster.
    Management cluster->>Management cluster: KOF data of the<br>management cluster<br>can be stored in:<br><br>the same management cluster,
    Management cluster->>Regional cluster: the regional cluster,
    Management cluster->>Third-party storage: a third-party storage,<br>e.g. AWS CloudWatch.
```

## Storage Class Requirements for VictoriaMetrics Cluster

When deploying VictoriaMetrics Cluster (used by KOF for metrics storage), consider the following Kubernetes storage class requirements:

- **ReadWriteMany:** Not required. Each `vmstorage` pod uses its own PersistentVolumeClaim (PVC) and does not share volumes with other pods. A `ReadWriteOnce` access mode is sufficient and recommended for most environments.
- **Reclaim Policy:** The default `Delete` policy is typically used, but you may choose `Retain` if you want to preserve data after PVC deletion for manual recovery.
- **Volume Expansion:** Enabling volume expansion is recommended. VictoriaMetrics can benefit from expanding storage as your data grows, and resizing PVCs is supported by most modern storage classes.
- **Required Space:** Storage requirements depend on your metrics volume, retention period, and replication factor. As a starting point, allocate at least 10–50 GiB per `vmstorage` pod for small clusters, and plan for growth based on actual ingestion rates and retention settings.
- **Volume Binding Mode:** `WaitForFirstConsumer` is recommended for better pod scheduling and to ensure volumes are provisioned in the correct availability zone or node pool.

For more details, see the [VictoriaMetrics Cluster documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/#cluster-resizing-and-scalability).

See also: [KOF Retention](./kof-retention.md) for details on configuring retention periods and replication factors for VictoriaMetrics and VictoriaLogs.

## From Child and Regional

KOF data collected from the child and regional clusters is routed out-of-the box.
No additional steps are required here.

## From Management to Management

This option stores KOF data of the management cluster in the same management cluster.

* VictoriaMetrics is provided by the `kof-mothership` chart, hence disabled in the `kof-storage` chart by default.
* PromxyServerGroup, VictoriaLogs, and VictoriaTraces are provided by the `kof-storage` chart.

To apply this option:

1. Add this patch to the existing `kof-values.yaml`:
    ```yaml
    kof-storage:
      enabled: true
    kof-collectors:
      enabled: true
      values:
        kcm:
          monitoring: true
        opentelemetry-kube-stack:
          clusterName: mothership
          defaultCRConfig:
            config:
              processors:
                resource/k8sclustername:
                  attributes:
                    - action: insert
                      key: k8s.cluster.name
                      value: mothership
                    - action: insert
                      key: k8s.cluster.namespace
                      value: kcm-system
              exporters:
                prometheusremotewrite:
                  external_labels:
                    cluster: mothership
                    clusterNamespace: kcm-system
    ```

    ??? note "If your management cluster is not [k0s](https://k0sproject.io/):"

        KOF scrapes `etcd` metrics using cert files like
        `/hostfs/${env:PKI_PATH}/pki/apiserver-etcd-client.crt`

        If you have `PKI_PATH` other than `var/lib/k0s`,
        e.g. when testing with [kind](https://kind.sigs.k8s.io/), merge this:

        ```yaml
        kof-collectors:
          values:
            opentelemetry-kube-stack:
              defaultCRConfig:
                env:
                  - name: PKI_PATH
                    value: etc/kubernetes
        ```

    ??? note "If you want to use a non-default storage class, merge this:"

        ```yaml
        kof-storage:
          values:
            victoria-logs-cluster:
              vlstorage:
                persistentVolume:
                  storageClassName: <EXAMPLE_STORAGE_CLASS>
        ```

2. Apply `kof-values.yaml` to the [Management Cluster](kof-install.md/#management-cluster):

{%
    include-markdown "../../../includes/kof-install-includes.md"
    start="<!--install-kof-start-->"
    end="<!--install-kof-end-->"
%}

## From Management to Regional

This option stores KOF data of the management cluster in the regional cluster
with the `REGIONAL_CLUSTER_NAME` configured [here](./kof-install.md#regional-cluster).

To apply this option:

1. Create the patch:
    ```bash
    cat <<EOF
    kof-child:
      values:
        fromManagement:
          toRegionalCluster:
            name: $REGIONAL_CLUSTER_NAME
    EOF
    ```

    ??? note "If your management cluster is not [k0s](https://k0sproject.io/):"

        KOF scrapes `etcd` metrics using cert files like
        `/hostfs/${env:PKI_PATH}/pki/apiserver-etcd-client.crt`

        If you have `PKI_PATH` other than `var/lib/k0s`,
        e.g. when testing with [kind](https://kind.sigs.k8s.io/), merge this:

        ```yaml
        kof-child:
          values:
            fromManagement:
              collectors:
                values:
                  opentelemetry-kube-stack:
                    defaultCRConfig:
                      env:
                        - name: PKI_PATH
                          value: etc/kubernetes
        ```

2. Add this patch to the existing `kof-values.yaml` file
    and then apply `kof-values.yaml` to the [Management Cluster](kof-install.md/#management-cluster):

{%
    include-markdown "../../../includes/kof-install-includes.md"
    start="<!--install-kof-start-->"
    end="<!--install-kof-end-->"
%}

## From Management to Third-party

This option stores KOF data of the management cluster in a third-party storage,
using the [AWS CloudWatch Logs Exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/awscloudwatchlogsexporter#readme) as an example.

Use the most secure option to [specify AWS credentials](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-credentials) in production.

For now, however, just for the sake of this demo, you can use the most straightforward
(though less secure) static credentials method:

1. Create AWS IAM user with access to CloudWatch Logs,
    for example, with `"Action": "logs:*"` allowed in the inline policy.

2. Create access key and save it to the `cloudwatch-credentials` file:
    ```
    AWS_ACCESS_KEY_ID=REDACTED
    AWS_SECRET_ACCESS_KEY=REDACTED
    ```

3. Create the `cloudwatch-credentials` secret:
    ```bash
    kubectl create secret generic -n kof cloudwatch-credentials \
      --from-env-file=cloudwatch-credentials
    ```

4. Create the patch:
    ```bash
    cat <<EOF
    kof-collectors:
      enabled: true
      values:
        kcm:
          monitoring: true
        opentelemetry-kube-stack:
          clusterName: mothership
          defaultCRConfig:
            env:
              - name: AWS_ACCESS_KEY_ID
                valueFrom:
                  secretKeyRef:
                    name: cloudwatch-credentials
                    key: AWS_ACCESS_KEY_ID
              - name: AWS_SECRET_ACCESS_KEY
                valueFrom:
                  secretKeyRef:
                    name: cloudwatch-credentials
                    key: AWS_SECRET_ACCESS_KEY
            config:
              processors:
                resource/k8sclustername:
                  attributes:
                    - action: insert
                      key: k8s.cluster.name
                      value: mothership
                    - action: insert
                      key: k8s.cluster.namespace
                      value: kcm-system
              exporters:
                awscloudwatchlogs:
                  region: us-east-2
                  log_group_name: management
                  log_stream_name: logs
                prometheusremotewrite: null
                otlphttp/logs: null
                otlphttp/traces: null
              service:
                pipelines:
                  logs:
                    exporters:
                    - awscloudwatchlogs
                    - debug
                  metrics:
                    exporters:
                    - debug
                  traces:
                    exporters:
                    - debug
    EOF
    ```

5. Add this patch to the existing `kof-values.yaml` file
    and then apply `kof-values.yaml` to the [Management Cluster](kof-install.md/#management-cluster):

{%
    include-markdown "../../../includes/kof-install-includes.md"
    start="<!--install-kof-start-->"
    end="<!--install-kof-end-->"
%}

6. Configure AWS CLI with the same access key, for verification:
    ```bash
    aws configure
    ```

7. Verify that the management cluster logs are stored in the CloudWatch:
    ```bash
    aws logs get-log-events \
      --region us-east-2 \
      --log-group-name management \
      --log-stream-name logs \
      --limit 1
    ```
    Example of the output:
    ```
    {"events": [{
      "timestamp": 1744305535107,
      "message": "{\"body\":\"10.244.0.1 - - [10/Apr/2025 17:18:55] \\\"GET /-/ready HTTP/1.1 200 ...
    ```
