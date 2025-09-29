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
cluster-related workloads to be hosted separately from the management cluster.

When using regional clusters, the management cluster remains the single pane of glass, responsible
for managing `ClusterDeployments`, `Credentials`, `Templates`, and other {{{ docsVersionInfo.k0rdentName }}} resources.
It then deploys the actual CAPI objects that implement those k0rdent resources on the regional cluster, a plain (non-k0rdent) 
Kubernetes cluster where provider-related infrastructure (such as CAPI
providers) and the objects that implement it (such as `Cluster`, and `Machine` objects) are deployed. 

Regional clusters provide several benefits:

* Cluster resources can be distributed across multiple, purpose-specific clusters.
* Improved fault isolation, because the management cluster only needs to deal with {{{ docsVersionInfo.k0rdentName }}} objects.
* Geographic placement of resources.

For more details, see:

- [Understanding Regional Components Segregation](components-segregation.md)
- [Register Regional Cluster](regional-cluster-registration.md)
- [Creating Credential in Region](creating-credential-in-region.md)
- [Deploying Clusters in Region](deploying-clusters-in-region.md)
