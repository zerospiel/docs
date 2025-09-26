# Regional Cluster Registration

> NOTE:
> Regional clusters are available starting from version 1.4.0.

To register an existing cluster as a regional cluster in {{{ docsVersionInfo.k0rdentName }}}, you must create
a `Region` object.

## Configuration Parameters

* `spec.kubeConfig`

Reference to the Secret containing the kubeconfig of the cluster being onboarded as a regional
cluster. The Secret must reside in the system namespace (default: `kcm-system`). Must specify both the `name`
of the Secret and the `key` where the kubeconfig content is stored. Required.

* `spec.core`

The core KCM and CAPI configuration. Allows modification of regional components (CAPI, Cluster API Operator, Velero,
Cert Manager). If unspecified, defaults are applied.

* `spec.providers`

List of enabled providers to deploy on the regional cluster.

It is possible to override the configuration (`config`) or template (`template`) of the core regional components or
providers. Example:

```yaml
spec:
  core:
    kcm:
      config:
        key1: value1
    capi:
      config:
        key1: value1
  providers:
  - name: <provider-name>
    config:
      key1: value1
    template: <provider-template-name>
```

## Examples

### Basic

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: Region
metadata:
  name: region1
spec:
  kubeConfig:
    name: kubeconfig-secret-name
    key: value
  providers:
  - name: cluster-api-provider-k0sproject-k0smotron
  - name: cluster-api-provider-openstack
  - name: projectsveltos
```

### Example with Regional Components Image Overrides

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: Region
metadata:
  name: region2
spec:
  core:
    kcm:
      config:
        cert-manager:
          cainjector:
            image:
              repository: custom-registry-url.com/jetstack/cert-manager-cainjector
          image:
            repository: custom-registry-url.com/jetstack/cert-manager-controller
          startupapicheck:
            image:
              repository: custom-registry-url.com/jetstack/cert-manager-startupapicheck
          webhook:
            image:
              repository: custom-registry-url.com/jetstack/cert-manager-webhook
        cluster-api-operator:
          image:
            manager:
              repository: custom-registry-url.com/capi-operator/cluster-api-operator
        velero:
          image:
            repository: custom-registry-url.com/velero/velero
  kubeConfig:
    name: kubeconfig-secret-name
    key: value
  providers:
  - name: cluster-api-provider-k0sproject-k0smotron
  - name: cluster-api-provider-aws
  - name: cluster-api-provider-azure
  - name: projectsveltos
```

## After the Region is Created

Once the Region object is created:

* The regional core components and enabled providers start installing on the regional cluster automatically.
* You can monitor the Region’s status and readiness using `kubectl wait`, for example:

```bash
kubectl wait --for=condition=Ready=True region/<region-name>
```

Once the Region is ready, you can start creating `Credentials` and deploy clusters in that region. See:
[Creating Credential in Region](creating-credential-in-region.md).
