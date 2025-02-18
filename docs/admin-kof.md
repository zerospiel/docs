# k0rdent Observability and FinOps (kof)

## Overview

k0rdent Observability and FinOps ([kof](https://github.com/k0rdent/kof)) provides enterprise-grade observability
and FinOps capabilities for k0rdent-managed Kubernetes clusters.
It enables centralized metrics, logging, and cost management
through a unified [OpenTelemetry](https://opentelemetry.io/docs/)-based architecture.

## Architecture

### High-level

From a high-level perspective, KOF consists of three layers: Collection, where the statistics and events are gathered, Storage, to keep track of them, and Management, where you
interact through the UI.

```
           ┌────────────────┐
           │   Management  │
           │   UI, promxy  │
           └────────┬───────┘
                    │
             ┌──────┴──────┐
             │            │
        ┌────┴─────┐ ┌─────┴────┐
        │ Storage │ │ Storage │
        │ region 1│ │ region 2│
        └────┬─────┘ └─────┬────┘
            │             │
      ┌──────┴──────┐     ...
      │             │
┌─────┴─────┐ ┌─────┴─────┐
│ Collect  │ │ Collect  │
│ managed 1│ │ managed 2│
└───────────┘ └───────────┘
```

### Mid-level

Getting a little bit more detailed, it's important to undrestand that data flows upwards, from observed resources to centralized Grafana om the
Management layer:

```
management cluster_____________________
│                                     │
│  kof-mothership chart_____________  │
│  │                               │  │
│  │ grafana-operator              │  │
│  │ victoria-metrics-operator     │  │
│  │ cluster-api-visualizer        │  │
│  │ sveltos-dashboard             │  │
│  │ k0rdent service templates     │  │
│  │ promxy                        │  │
│  │_______________________________│  │
│                                     │
│  kof-operators chart_____________   │
│  │                              │   │
│  │  opentelemetry-operator      │   │
│  │  prometheus-operator-crds    │   │
│  │______________________________│   │
│_____________________________________│


cloud 1...
│
│  region 1__________________________________________  region 2...
│  │                                                │  │
.  │  storage cluster_____________________          │  │
.  │  │                                  │          │  │
.  │  │  kof-storage chart_____________  │          │  .
   │  │  │                            │  │          │  .
   │  │  │ grafana-operator           │  │          │  .
   │  │  │ victoria-metrics-operator  │  │          │
   │  │  │ victoria-logs-single       │  │          │
   │  │  │ external-dns               │  │          │
   │  │  │____________________________│  │          │
   │  │                                  │          │
   │  │  cert-manager (grafana, vmauth)  │          │
   │  │  ingress-nginx                   │          │
   │  │__________________________________│          │
   │                                                │
   │                                                │
   │  cluster deployment 1__________________  2...  │
   │  │                                    │  │     │
   │  │  cert-manager (OTel-operator)      │  │     │
   │  │                                    │  │     │
   │  │  kof-operators chart_____________  │  .     │
   │  │  │                              │  │  .     │
   │  │  │  opentelemetry-operator____  │  │  .     │
   │  │  │  │                        │  │  │        │
   │  │  │  │ OpenTelemetryCollector │  │  │        │
   │  │  │  │________________________│  │  │        │
   │  │  │                              │  │        │
   │  │  │  prometheus-operator-crds    │  │        │
   │  │  │______________________________│  │        │
   │  │                                    │        │
   │  │  kof-collectors chart________      │        │
   │  │  │                          │      │        │
   │  │  │ opencost                 │      │        │
   │  │  │ kube-state-metrics       │      │        │
   │  │  │ prometheus-node-exporter │      │        │
   │  │  │__________________________│      │        │
   │  │                                    │        │
   │  │  observed resources                │        │
   │  │____________________________________│        │
   │________________________________________________│
```

### Low-level

At a low level, you can see how logs and traces work their way around the system.

![kof-architecture](assets/kof/otel.png)

## Helm Charts

KOF is deployed as a series of Helm charts at various levels.

### kof-mothership

- Centralized [Grafana](https://grafana.com/) dashboard, managed by [grafana-operator](https://github.com/grafana/grafana-operator)
- Local [VictoriaMetrics](https://victoriametrics.com/) storage for alerting rules only, managed by [victoria-metrics-operator](https://docs.victoriametrics.com/operator/)
- [cluster-api-visualizer](https://github.com/Jont828/cluster-api-visualizer) for insight into multicluster configuration
- [Sveltos](https://projectsveltos.github.io/sveltos/) dashboard, automatic secret distribution
- [k0rdent](https://github.com/k0rdent) service templates to deploy other charts to regional clusters
- [Promxy](https://github.com/jacksontj/promxy) for aggregating Prometheus metrics from regional clusters

### kof-storage

- Regional [Grafana](https://grafana.com/) dashboard, managed by [grafana-operator](https://github.com/grafana/grafana-operator)
- Regional [VictoriaMetrics](https://victoriametrics.com/) storage with main data, managed by [victoria-metrics-operator](https://docs.victoriametrics.com/operator/)
    - [vmauth](https://docs.victoriametrics.com/vmauth/) entrypoint proxy for VictoriaMetrics components
    - [vmcluster](https://docs.victoriametrics.com/operator/resources/vmcluster/) for high-available fault-tolerant version of VictoriaMetrics database
    - [victoria-logs-single](https://github.com/VictoriaMetrics/helm-charts/tree/master/charts/victoria-logs-single) for high-performance, cost-effective, scalable logs storage
- [external-dns](https://github.com/kubernetes-sigs/external-dns) to communicate with other clusters

### kof-operators

- [prometheus-operator-crds](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-operator-crds) required to create OpenTelemetry collectors, also required to monitor `kof-mothership` itself
- [OpenTelemetry](https://opentelemetry.io/) [collectors](https://opentelemetry.io/docs/collector/) below, managed by [opentelemetry-operator](https://opentelemetry.io/docs/kubernetes/operator/)

### kof-collectors

- [prometheus-node-exporter](https://prometheus.io/docs/guides/node-exporter/) for hardware and OS metrics
- [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) for metrics about the state of Kubernetes objects
- [OpenCost](https://www.opencost.io/) "shines a light into the black box of Kubernetes spend"

## Installation

### Prerequisites

Before beginning KOF installation, you should have the following compoents in place:

* A k0rdent management cluster - You can get instructions to create one in the [quickstart guide](https://docs.k0rdent.io/v0.1.0/quickstart-1-mgmt-node-and-cluster/)
    * To test on [macOS](https://docs.k0sproject.io/stable/system-requirements/#host-operating-system) you can install using:
      `brew install kind && kind create cluster -n k0rdent`
* You will also need your infrastructure provider credentials, such as those shown in the [guide for AWS](https://docs.k0rdent.io/v0.1.0/quickstart-2-aws/)
    * Note that you should skip the "Create your ClusterDeployment" and later sections.
* Finally, you need access to create DNS records for service endpoints such as `kof.example.com`

### DNS auto-config

To avoid [manual configuration of DNS records for service endpoints](#manual-dns-config) later,
you can automate the process now using [external-dns](https://kubernetes-sigs.github.io/external-dns/latest/).

For example, for AWS you should use the [Node IAM Role](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#node-iam-role)
or [IRSA](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#iam-roles-for-service-accounts) methods in production.

For now, however, just for the sake of this demo based on the `aws-standalone` template,
you an use the most straightforward (though less secure) [static credentials](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#static-credentials) method:

1. Create an `external-dns` IAM user with [this policy](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#iam-policy).
2. Create an access key and `external-dns-aws-credentials` file, as in:
    ```
    [default]
    aws_access_key_id = <EXAMPLE_ACCESS_KEY_ID>
    aws_secret_access_key = <EXAMPLE_SECRET_ACCESS_KEY>
    ```
3. Create the `external-dns-aws-credentials` secret in the `kof` namespace:
    ```shell
    kubectl create namespace kof
    kubectl create secret generic \
      -n kof external-dns-aws-credentials \
      --from-file external-dns-aws-credentials
    ```

### Management Cluster

Follow these steps to install the KOF components on the management cluster:

1. Install `kof-operators` required by `kof-mothership`:
    ```shell
    helm install --create-namespace -n kof kof-operators \
      oci://ghcr.io/k0rdent/kof/charts/kof-operators --version 0.1.0
    ```

2. Construct the values for `kof-mothership`:
    ```shell
    cat >mothership-values.yaml <<EOF
    kcm:
      installTemplates: true
      kof:
        clusterProfiles:
          kof-aws-dns-secrets:
            matchLabels:
              k0rdent.mirantis.com/kof-aws-dns-secrets: "true"
            secrets:
              - external-dns-aws-credentials
    EOF
    ```

    It's important to understand that we override some [default values](https://github.com/k0rdent/kof/blob/main/charts/kof-mothership/values.yaml) here:

    * `kcm.installTemplates` installs the templates such as `cert-manager` and `kof-storage` into the management cluster. This makes it possible to reference them from `.spec.serviceSpec.services[].template` in the AWS `ClusterDeployment` below.
    * `external-dns-aws-credentials` is a secret created in the [DNS auto-config](#dns-auto-config) section and is auto-distributed to storage clusters by Sveltos. If you've opted out of [DNS auto-config](#dns-auto-config) then don't add the `kof-aws-dns-secrets` cluster profile above.
    * `storage-vmuser-credentials` is a secret auto-created by default and auto-distributed to other clusters by the Sveltos `ClusterProfile` [here](https://github.com/k0rdent/kof/blob/121b61f5f6de6ddfdf3525b98f3ad4cb8ce57eaa/charts/kof-mothership/values.yaml#L25-L31).
    * `grafana-admin-credentials` is a secret auto-created by default [here](https://github.com/k0rdent/kof/blob/121b61f5f6de6ddfdf3525b98f3ad4cb8ce57eaa/charts/kof-mothership/values.yaml#L64-L65). We will use it in the [Grafana](#grafana) section.

3. Install `kof-mothership`:
    ```shell
    helm install -f mothership-values.yaml -n kof kof-mothership \
      oci://ghcr.io/k0rdent/kof/charts/kof-mothership --version 0.1.0
    ```

4. Wait for all pods to show that they're `Running`:
    ```shell
    kubectl get pod -n kof
    ```

### Storage Cluster

Now install KOF on the Storage cluster:

1. Look through the [default values](https://github.com/k0rdent/kof/blob/main/charts/kof-storage/values.yaml) of the `kof-storage` chart.
2. Apply the quick start example for AWS, or use it as a reference.

3. Set your own KOF variables using your own values:
    ```shell
    STORAGE_CLUSTER_NAME=cloud1-region1
    STORAGE_DOMAIN=$STORAGE_CLUSTER_NAME.kof.example.com
    ADMIN_EMAIL=$(git config user.email)
    echo "$STORAGE_CLUSTER_NAME, $STORAGE_DOMAIN, $ADMIN_EMAIL"
    ```

4. Use the up-to-date KOF template, as in:
    ```shell
    kubectl get clustertemplate -n kcm-system | grep aws
    TEMPLATE=aws-standalone-cp-0-1-0
    ```

5. Compose the following objects:
    * `ClusterDeployment` - storage cluster
    * `PromxyServerGroup` - for metrics
    * `GrafanaDatasource` - for logs

    ```shell
    cat >storage-cluster.yaml <<EOF
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: $STORAGE_CLUSTER_NAME
      namespace: kcm-system
      labels:
        kof: storage
    spec:
      template: $TEMPLATE
      credential: aws-cluster-identity-cred
      config:
        clusterIdentity:
          name: aws-cluster-identity
          namespace: kcm-system
        controlPlane:
          instanceType: t3.large
        controlPlaneNumber: 1
        publicIP: true
        region: us-east-2
        worker:
          instanceType: t3.medium
        workersNumber: 3
        clusterLabels:
          k0rdent.mirantis.com/kof-storage-secrets: "true"
          k0rdent.mirantis.com/kof-aws-dns-secrets: "true"
      serviceSpec:
        priority: 100
        services:
          - name: ingress-nginx
            namespace: ingress-nginx
            template: ingress-nginx-4-11-3
          - name: cert-manager
            namespace: cert-manager
            template: cert-manager-1-16-2
            values: |
              cert-manager:
                crds:
                  enabled: true
          - name: kof-storage
            namespace: kof
            template: kof-storage-0-1-0
            values: |
              external-dns:
                enabled: true
              victoriametrics:
                vmauth:
                  ingress:
                    host: vmauth.$STORAGE_DOMAIN
                security:
                  username_key: username
                  password_key: password
                  credentials_secret_name: storage-vmuser-credentials
              grafana:
                ingress:
                  host: grafana.$STORAGE_DOMAIN
                security:
                  credentials_secret_name: grafana-admin-credentials
              cert-manager:
                email: $ADMIN_EMAIL
    ---
    apiVersion: kof.k0rdent.mirantis.com/v1alpha1
    kind: PromxyServerGroup
    metadata:
      labels:
        app.kubernetes.io/name: promxy-operator
        k0rdent.mirantis.com/promxy-secret-name: kof-mothership-promxy-config
      name: promxyservergroup-sample
      namespace: kof
    spec:
      cluster_name: $STORAGE_CLUSTER_NAME
      targets:
        - "vmauth.$STORAGE_DOMAIN:443"
      path_prefix: /vm/select/0/prometheus/
      scheme: https
      http_client:
        dial_timeout: "5s"
        tls_config:
          insecure_skip_verify: true
        basic_auth:
          credentials_secret_name: storage-vmuser-credentials
          username_key: username
          password_key: password
    ---
    apiVersion: grafana.integreatly.org/v1beta1
    kind: GrafanaDatasource
    metadata:
      labels:
        app.kubernetes.io/managed-by: Helm
      name: victoria-logs-storage0
      namespace: kof
    spec:
      valuesFrom:
        - targetPath: "basicAuthUser"
          valueFrom:
            secretKeyRef:
              key: username
              name: storage-vmuser-credentials
        - targetPath: "secureJsonData.basicAuthPassword"
          valueFrom:
            secretKeyRef:
              key: password
              name: storage-vmuser-credentials
      datasource:
        name: $STORAGE_CLUSTER_NAME
        url: https://vmauth.$STORAGE_DOMAIN/vls
        access: proxy
        isDefault: false
        type: "victoriametrics-logs-datasource"
        basicAuth: true
        basicAuthUser: \${username}
        secureJsonData:
          basicAuthPassword: \${password}
      instanceSelector:
        matchLabels:
          dashboards: grafana
      resyncPeriod: 5m
    EOF
    ```

6. Verify and apply the Storage `ClusterDeployment`:
    ```shell
    cat storage-cluster.yaml

    kubectl apply -f storage-cluster.yaml
    ```

7. Watch how the cluster is deployed to AWS until all values of `READY` are `True`:
    ```shell
    clusterctl describe cluster -n kcm-system $STORAGE_CLUSTER_NAME \
      --show-conditions all
    ```

### Cluster Deployment

On the actual cluster to be monitored, do the following:

1. Look through the default values of the [kof-operators](https://github.com/k0rdent/kof/blob/main/charts/kof-operators/values.yaml)
   and [kof-collectors](https://github.com/k0rdent/kof/blob/main/charts/kof-collectors/values.yaml) charts.

2. Apply the next quick start example for AWS, or use it as a reference.

3. Set your own value below, verifing [the variables](#storage-cluster):
    ```shell
    MANAGED_CLUSTER_NAME=$STORAGE_CLUSTER_NAME-managed1
    echo "$MANAGED_CLUSTER_NAME, $STORAGE_DOMAIN"
    ```

4. Use the up-to-date template, as in:
    ```shell
    kubectl get clustertemplate -n kcm-system | grep aws
    TEMPLATE=aws-standalone-cp-0-1-0
    ```

5. Compose the `ClusterDeployment`:

    ```shell
    cat >managed-cluster.yaml <<EOF
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: $MANAGED_CLUSTER_NAME
      namespace: kcm-system
      labels:
        kof: collector
    spec:
      template: $TEMPLATE
      credential: aws-cluster-identity-cred
      config:
        clusterIdentity:
          name: aws-cluster-identity
          namespace: kcm-system
        controlPlane:
          instanceType: t3.large
        controlPlaneNumber: 1
        publicIP: false
        region: us-east-2
        worker:
          instanceType: t3.small
        workersNumber: 3
        clusterLabels:
          k0rdent.mirantis.com/kof-storage-secrets: "true"
      serviceSpec:
        priority: 100
        services:
          - name: cert-manager
            namespace: kof
            template: cert-manager-1-16-2
            values: |
              cert-manager:
                crds:
                  enabled: true
          - name: kof-operators
            namespace: kof
            template: kof-operators-0-1-0
          - name: kof-collectors
            namespace: kof
            template: kof-collectors-0-1-0
            values: |
              global:
                clusterName: $MANAGED_CLUSTER_NAME
              opencost:
                enabled: true
                opencost:
                  prometheus:
                    username_key: username
                    password_key: password
                    existingSecretName: storage-vmuser-credentials
                    external:
                      url: https://vmauth.$STORAGE_DOMAIN/vm/select/0/prometheus
                  exporter:
                    defaultClusterId: $MANAGED_CLUSTER_NAME
              kof:
                logs:
                  username_key: username
                  password_key: password
                  credentials_secret_name: storage-vmuser-credentials
                  endpoint: https://vmauth.$STORAGE_DOMAIN/vls/insert/opentelemetry/v1/logs
                metrics:
                  username_key: username
                  password_key: password
                  credentials_secret_name: storage-vmuser-credentials
                  endpoint: https://vmauth.$STORAGE_DOMAIN/vm/insert/0/prometheus/api/v1/write
    EOF
    ```

6. Verify and apply the `ClusterDeployment`:
    ```shell
    cat managed-cluster.yaml

    kubectl apply -f managed-cluster.yaml
    ``:

7. Watch while the cluster is deployed to AWS until all values of `READY` are `True`:
    ```shell
    clusterctl describe cluster -n kcm-system $MANAGED_CLUSTER_NAME \
      --show-conditions all
    ```

### Verification

Finally, verify that KOF installed properly.

```shell
kubectl get clustersummaries -A -o wide
```
Wait until the value of `Provisioning` becomes `Provisioned`.

```shell
kubectl get secret -n kcm-system $STORAGE_CLUSTER_NAME-kubeconfig \
  -o=jsonpath={.data.value} | base64 -d > storage-kubeconfig

kubectl get secret -n kcm-system $MANAGED_CLUSTER_NAME-kubeconfig \
  -o=jsonpath={.data.value} | base64 -d > managed-kubeconfig

KUBECONFIG=storage-kubeconfig kubectl get pod -A
  # Namespaces: cert-manager, ingress-nginx, kof, kube-system, projectsveltos

KUBECONFIG=managed-kubeconfig kubectl get pod -A
  # Namespaces: kof, kube-system, projectsveltos
```
Wait for all pods to show as `Running`.

### Manual DNS config

If you've opted out of [DNS auto-config](#dns-auto-config), you will need to do the following:

1. Get the `EXTERNAL-IP` of `ingress-nginx`:
    ```shell
    KUBECONFIG=storage-kubeconfig kubectl get svc \
      -n ingress-nginx ingress-nginx-controller
    ```
    It should look like `REDACTED.us-east-2.elb.amazonaws.com`

2. Create the these DNS records of type `A` both pointing to that `EXTERNAL-IP`:
    ```shell
    echo vmauth.$STORAGE_DOMAIN
    echo grafana.$STORAGE_DOMAIN
    ```

## Sveltos

Use the [Sveltos dashboard](https://projectsveltos.github.io/sveltos/getting_started/install/dashboard/#platform-administrator-example)
to verify secrets have been auto-distributed to the required clusters:

1. Start by preparing the system:

    ```shell
    kubectl create sa platform-admin
    kubectl create clusterrolebinding platform-admin-access \
      --clusterrole cluster-admin --serviceaccount default:platform-admin

    kubectl create token platform-admin --duration=24h
    kubectl port-forward -n kof svc/dashboard 8081:80
    ```

2. Now open [http://127.0.0.1:8081/login](http://127.0.0.1:8081/login) and paste the token output in step 1 above.
3. Open the `ClusterAPI` tab: [http://127.0.0.1:8081/sveltos/clusters/ClusterAPI/1](http://127.0.0.1:8081/sveltos/clusters/ClusterAPI/1)
4. Check both storage and managed clusters:
    * Cluster profiles should be `Provisioned`.
    * Secrets should be distributed.

![sveltos-demo](assets/kof/sveltos-2025-02-06.gif)

## Grafana

### Access to Grafana

To make Grafana available, follow these steps:

1. Get the Grafana username and password:
    ```shell
    kubectl get secret -n kof grafana-admin-credentials -o yaml | yq '{
      "user": .data.GF_SECURITY_ADMIN_USER | @base64d,
      "pass": .data.GF_SECURITY_ADMIN_PASSWORD | @base64d
    }'
    ```

2. Start the Grafana dashboard:
    ```shell
    kubectl port-forward -n kof svc/grafana-vm-service 3000:3000
    ```

3. Login to [http://127.0.0.1:3000/dashboards](http://127.0.0.1:3000/dashboards) with the username/password printed above.
4. Open a dashboard:

![grafana-demo](assets/kof/grafana-2025-01-14.gif)

### Cluster Overview

From here you can get an overview of the cluster, including:

* Health metrics
* Resource utilization
* Performance trends
* Cost analysis

### Logging Interface

The logging interface will also be available, including:

* Real-time log streaming
* Full-text search
* Log aggregation
* Alert correlation

### Cost Management

Finally there are the cost management features, including:

* Resource cost tracking
* Usage analysis
* Budget monitoring
* Optimization recommendations

## Scaling Guidelines

The method for scaling KOF depends on the type of expansion:

### Regional Expansion

1. Deploy a [Storage Cluster](#storage-cluster) in new region
2. Update promxy configuration
3. Configure collector routing to point to the additional capacity

### Adding a New ClusterDeployment

1. Apply templates, as in the [Managed Cluster](#managed-cluster) section
2. Verify the data flow
3. Configure any custom dashboards

## Maintenance

### Backup Requirements

Backing up KOF requires backing up the following:

* Grafana configurations
* Alert definitions
* Custom dashboards
* Retention policies

### Health Monitoring

To implement health monitoring:

1. Apply the steps in the [Verification](#verification) section
2. Apply the steps in the [Sveltos](#sveltos) section

### Uninstallation

To remove the demo clusters created in this section:

> WARNING:
> Make sure these are just your demo clusters and do not contain important data.

```shell
kubectl delete -f managed-cluster.yaml
kubectl delete -f storage-cluster.yaml
```

To remove KOF, use helm:

```shell
helm uninstall -n kof kof-mothership
```

## Resource Limits

### Resources of Management Cluster

- [promxy](https://github.com/k0rdent/kof/blob/121b61f5f6de6ddfdf3525b98f3ad4cb8ce57eaa/charts/kof-mothership/values.yaml#L120-L126):
  ```yaml
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 100m
      memory: 128Mi
  ```

- [promxy-deployment](https://github.com/k0rdent/kof/blob/121b61f5f6de6ddfdf3525b98f3ad4cb8ce57eaa/charts/kof-mothership/templates/promxy/promxy-deployment.yaml#L107-L113):
  ```yaml
  resources:
    requests:
      cpu: 0.02
      memory: 20Mi
    limits:
      cpu: 0.02
      memory: 20Mi
  ```

- [promxy-operator](https://github.com/k0rdent/kof/blob/121b61f5f6de6ddfdf3525b98f3ad4cb8ce57eaa/promxy-operator/config/manager/manager.yaml#L87-L93):
  ```yaml
  resources:
    limits:
      cpu: 500m
      memory: 128Mi
    requests:
      cpu: 10m
      memory: 64Mi
  ```

### Resources of a ClusterDeployment

- [opentelemetry](https://github.com/k0rdent/kof/blob/121b61f5f6de6ddfdf3525b98f3ad4cb8ce57eaa/charts/kof-collectors/templates/opentelemetry/instrumentation.yaml#L18-L22):
  ```yaml
  resourceRequirements:
    limits:
      memory: 128Mi
    requests:
      memory: 128Mi
  ```

## Version Compatibility

| Component       | Version  | Notes                         |
|-----------------|----------|-------------------------------|
| k0rdent         | ≥ 0.0.7  | Required for template support |
| Kubernetes      | ≥ 1.32   | Earlier versions untested     |
| OpenTelemetry   | ≥ 0.75   | Recommended minimum           |
| VictoriaMetrics | ≥ 0.40   | Required for clustering       |

Detailed:

- [kof-mothership](https://github.com/k0rdent/kof/blob/main/charts/kof-mothership/Chart.yaml)
- [kof-storage](https://github.com/k0rdent/kof/blob/main/charts/kof-storage/Chart.yaml)
- [kof-operators](https://github.com/k0rdent/kof/blob/main/charts/kof-operators/Chart.yaml)
- [kof-collectors](https://github.com/k0rdent/kof/blob/main/charts/kof-collectors/Chart.yaml)

## More

- If you've applied this guide you should have kof up and running.
- Check [k0rdent/kof/docs](https://github.com/k0rdent/kof/tree/main/docs) for advanced guides such as configuring alerts.
