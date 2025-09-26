# Regional Clusters

> NOTE:
> Regional clusters are available starting from version 1.4.0.

> WARNING:
> There are some limitations in deploying regional clusters, which will be addressed in upcoming
> {{{ docsVersionInfo.k0rdentName }}} releases:
>
> * Automated credential distribution is not yet supported. You must manually create the corresponding ClusterIdentity
> objects (e.g., `Secrets`, `AWSClusterIdentity`) on each regional cluster.
> * The kubeconfig Secret must exist in the system namespace (default: `kcm-system`) when registering a new region.

To improve isolation, {{{ docsVersionInfo.k0rdentName }}} introduces the concept of regional clusters. This allows
cluster workloads to be hosted separately from the management cluster.

A regional cluster is a separate cluster where user workloads and provider-related infrastructure (such as CAPI
providers, clusters, and machines) are deployed. The management cluster remains the single pane of glass, responsible
for managing ClusterDeployments, Credentials, Templates, and other {{{ docsVersionInfo.k0rdentName }}} resources.

Regional clusters provide several benefits:

* Workloads can be distributed across multiple, purpose-specific clusters.
* Improved fault isolation.
* Geographic placement of resources.

For more details, see:

- [Understanding Regional Components Segregation](components-segregation.md)
- [Register Regional Cluster](regional-cluster-registration.md)
- [Creating Credential in Region](creating-credential-in-region.md)
- [Deploying Clusters in Region](deploying-clusters-in-region.md)
