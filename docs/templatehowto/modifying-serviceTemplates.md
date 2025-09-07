# Modifying ServiceTemplates

### Alternative Template Sources

Aside from Helm charts, `ServiceTemplate` also supports alternative resource definitions using either Kustomize or raw resources.

You can use one of the following fields in `.spec` (they are mutually exclusive):

- `.spec.kustomize`
- `.spec.resources`
- `.spec.helm.chartSource`

Each of these fields accepts a `SourceSpec`, which defines the origin of the template content. Only one can be used at a time.

A `SourceSpec` includes:

| **Field**          | **Description**                                                                                                                                                                      |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `deploymentType`   | Must be either `Local` or `Remote`. Defines whether resources will be deployed to management (local) or to managed (remote) cluster. Ignored if defined in `.spec.helm.chartSource`. |
| `localSourceRef`   | Reference to a local source (e.g., `ConfigMap`, `Secret`, `GitRepository`, `Bucket`, `OCIRepository`). `ConfigMap` and `Secret` are not supported in `.spec.helm.chartSource`.       |
| `remoteSourceSpec` | Configuration for a remote source. Includes support for `Git`, `Bucket`, or `OCI` repositories.                                                                                      |
| `path`             | Path within the source object pointing to the manifests or kustomize config. Ignored when deploying raw resources using `ConfigMap` or `Secret`                                      |

> NOTE:
> Fields `.spec.*.remoteSourceSpec.git`, `.spec.*.remoteSource.Spec.bucket` and `.spec.*.remoteSourceSpec.oci` are mutually exclusive.

Example using `.spec.kustomize`:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ServiceTemplate
metadata:
   name: example-kustomization
   namespace: kcm-system
spec:
  kustomize:
    deploymentType: Remote
    remoteSourceSpec:
      git:
        url: https://github.com/example/repo
        ref:
          branch: main
    path: ./overlays/dev
```

Example using `.spec.resources`:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ServiceTemplate
metadata:
   name: example-resources
   namespace: kcm-system
spec:
  resources:
    deploymentType: Local
    localSourceRef:
      kind: ConfigMap
      name: my-configmap
    path: ./manifests
```

All of the above follow the same mutual exclusivity and version constraint rules as Helm.

## Required and exposed providers definition

The `*Template` object must specify the list of Cluster API providers that are either **required** (for
`ClusterTemplates` and `ServiceTemplates`) or **exposed** (for `ProviderTemplates`). These providers include
`infrastructure`, `bootstrap`, and `control-plane`. This can be achieved in two ways:

1. By listing the providers explicitly in the `spec.providers` field.
2. Alternatively, by including specific annotations in the `Chart.yaml` of the referenced Helm chart. The annotations should list the providers as a `comma-separated` value.

For example:

In a `Template` spec:

```yaml
spec:
  providers:
  - bootstrap-k0sproject-k0smotron
  - control-plane-k0sproject-k0smotron
  - infrastructure-aws
```

In a `Chart.yaml`:

```bash
annotations:
    cluster.x-k8s.io/provider: infrastructure-aws, control-plane-k0sproject-k0smotron, bootstrap-k0sproject-k0smotron
```

## Compatibility attributes

Each of the `*Template` resources has compatibility versions attributes to constraint the core `CAPI`, `CAPI` provider or Kubernetes versions.
CAPI-related version constraints must be set in the [`CAPI` contract format](https://cluster-api.sigs.k8s.io/developer/providers/contracts/overview).
Kubernetes version constraints must be set in the Semantic Version format.
Each attribute can be set either via the corresponding `.spec` fields or via the annotations.
Values set via the `.spec` have precedence over the values set via the annotations.

> NOTE:
> All of the compatibility attributes are optional, and validation checks only take place
> if **both** of the corresponding type attributes
> (e.g. provider contract versions in both `ProviderTemplate` and `ClusterTemplate`) are set.

1. The `ProviderTemplate` resource has dedicated fields to set compatible `CAPI` contract versions along
with CRDs contract versions supported by the provider.
Given contract versions will then be set accordingly in the `.status` field.
Compatibility contract versions are key-value pairs, where the key is **the core `CAPI` contract version**,
and the value is an underscore-delimited (_) list of provider contract versions supported by the core `CAPI`.
For the core `CAPI` Template values should be empty.

    Example with the `.spec`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1beta1
    kind: ProviderTemplate
    # ...
    spec:
      providers:
      - infrastructure-aws
      capiContracts:
        # commented is the example exclusively for the core CAPI Template
        # v1alpha3: ""
        # v1alpha4: ""
        # v1beta1: ""
        v1alpha3: v1alpha3
        v1alpha4: v1alpha4
        v1beta1: v1beta1_v1beta2
    ```

    Example with the `annotations` in the `Chart.yaml` with the same logic
    as in the `.spec`:

    ```yaml
    annotations:
      cluster.x-k8s.io/provider: infrastructure-aws
      cluster.x-k8s.io/v1alpha3: v1alpha3
      cluster.x-k8s.io/v1alpha4: v1alpha4
      cluster.x-k8s.io/v1beta1: v1beta1_v1beta2
    ```

1. The `ClusterTemplate` resource has dedicated fields to set an exact compatible Kubernetes version
in the Semantic Version format and required contract versions per each provider to match against
the related `ProviderTemplate` objects.
Given compatibility attributes will be then set accordingly in the `.status` field.
Compatibility contract versions are key-value pairs, where the key is **the name of the provider**,
and the value is the provider contract version required to be supported by the provider.

    Example with the `.spec`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1beta1
    kind: ClusterTemplate
    # ...
    spec:
      k8sVersion: 1.30.0 # only exact semantic version is applicable
      providers:
      - bootstrap-k0sproject-k0smotron
      - control-plane-k0sproject-k0smotron
      - infrastructure-aws
      providerContracts:
        bootstrap-k0sproject-k0smotron: v1beta1 # only a single contract version is applicable
        control-plane-k0sproject-k0smotron: v1beta1
        infrastructure-aws: v1beta2
    ```

    Example with the `.annotations` in the `Chart.yaml`:

    ```yaml
    annotations:
      cluster.x-k8s.io/provider: infrastructure-aws, control-plane-k0sproject-k0smotron, bootstrap-k0sproject-k0smotron
      cluster.x-k8s.io/bootstrap-k0sproject-k0smotron: v1beta1
      cluster.x-k8s.io/control-plane-k0sproject-k0smotron: v1beta1
      cluster.x-k8s.io/infrastructure-aws: v1beta2
      k0rdent.mirantis.com/k8s-version: 1.30.0
    ```

1. The `ServiceTemplate` resource has dedicated fields to set an compatibility constrained
Kubernetes version to match against the related `ClusterTemplate` objects.
Given compatibility values will be then set accordingly in the `.status` field.

    Example with the `.spec`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1beta1
    kind: ServiceTemplate
    # ...
    spec:
      k8sConstraint: "^1.30.0" # only semantic version constraints are applicable
    ```

    Example with the `annotations` in the `Chart.yaml`:

    ```yaml
    k0rdent.mirantis.com/k8s-version-constraint: ^1.30.0
    ```

### Compatibility attributes enforcement

The aforedescribed attributes are checked for compliance with the following rules:

* Both the exact and constraint version of the same type (for example `k8sVersion` and `k8sConstraint`) must
be set, otherwise no check is performed;
* If a `ClusterTemplate` object's provider's contract version does not satisfy contract versions
from the related `ProviderTemplate` object, updates to the `ClusterDeployment` object will be blocked;
* If a `ProviderTemplate` object's `CAPI` contract version
(for example, in a `v1beta1: v1beta1_v1beta2` key-value pair, the key `v1beta1` is the core `CAPI` contract version)
is not listed in the core `CAPI` `ProviderTemplate` object, the updates to the `Management` object will be blocked;
* If a `ClusterTemplate` object's exact kubernetes version does not satisfy the kubernetes version
constraint from the related `ServiceTemplate` object, the updates to the `ClusterDeployment` object will be blocked.