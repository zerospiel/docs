# QuickStart 3 - Deploy managed clusters on AWS

k0rdent's template-driven system includes `ClusterTemplates`, for creating and lifecycle managing clusters on infrastructures, and `ServiceTemplates`, for installing and lifecycle managing services on them.

Right now, what we're interested in is using a ClusterTemplate to describe the managed clusters we want k0rdent to create on AWS. In the real world, this work would probably be done by a Platform Team Lead, because it requires admin access to k0rdent's Management Cluster.

Here is our ClusterTemplate:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: ClusterTemplate
metadata:
  name: demo-aws-standalone-cp-0.0.1
  namespace: hmc-system
spec:
  helm:
    chartSpec:
      chart: demo-aws-standalone-cp
      version: 0.0.1
      interval: 10m0s
      sourceRef:
        kind: HelmRepository
        name: 2a-demos
  providers:
  - bootstrap-k0smotron
  - control-plane-k0smotron
  - infrastructure-aws
  providerContracts:
    bootstrap-k0smotron: v1beta1
    control-plane-k0smotron: v1beta1
    infrastructure-aws: v1beta2
---
apiVersion: hmc.mirantis.com/v1alpha1
kind: ClusterTemplateChain
metadata:
  name: demo-aws-standalone-cp-0.0.1
  namespace: hmc-system
spec:
  supportedTemplates:
    - name: demo-aws-standalone-cp-0.0.1
```

This YAML file includes two documents. The first, the ClusterTemplate, provides information that will be used to create the cluster. It specifies the Helm chart information, as well as CAPI providers and information about them. The second document, the ClusterTemplateChain, pertains to how k0rdent aligns different templates to enable coordinated management within guardrails that enforce best practices (read all about it in the [Administration Guide]()).

## Install ClusterTemplate

Before we can use this template, we need to install it into the management cluster. You can do this manually by applying the YAML file to the cluster, but for now, let’s use the convenience script:

```shell
make apply-clustertemplate-demo-aws-standalone-cp-0.0.1
```

**Note:** Applying the ClusterTemplate doesn't actually create a managed cluster. What it does is create the ClusterTemplate object k0rdent will use to enable creation and lifecycle management of this kind of cluster.

## View ClusterTemplates

When we installed k0rdent earlier, many templates got set up this way. We can see them with kubectl:

```shell
kubectl -n hmc-system get clustertemplate
```

You should see a list of ClusterTemplates:

```shell
NAME                           VALID
aws-eks-0-0-2                  true
aws-hosted-cp-0-0-3            true
aws-standalone-cp-0-0-4        true
azure-hosted-cp-0-0-3          true
azure-standalone-cp-0-0-4      true
demo-aws-standalone-cp-0.0.1   false
vsphere-hosted-cp-0-0-3        true
vsphere-standalone-cp-0-0-3    true
```

These are all the cluster and cloud/infrastructure types k0rdent lets us deploy and manage 'out of the box.'

## Examine a ManagedCluster

Actually deploying a cluster requires creating another kind of object for k0rdent: a `ManagedCluster`. Here's an example:

```shell
apiVersion: hmc.mirantis.com/v1alpha1
kind: ManagedCluster
metadata:
  name: my-aws-managedcluster1
  namespace: hmc-system
spec:
  template: aws-standalone-cp-0-0-4
  credential: cluster-identity-cred
  config:
    region: us-west-2
    controlPlane:
      instanceType: t3.small
    worker:
      instanceType: t3.small
```

The ManagedCluster object ties a ClusterTemplate together with instance specifics (e.g., What do you want to call this cluster? What k0rdent namespace should this cluster live in?) and infrastructure details (e.g., what AWS region do we want to deploy into? What instance type(s) should k0rdent use for control plane and worker nodes?), among other info, including the credentials we just created &mdash; required to deploy this instance.

Again, we could simply apply this template to the management cluster, like this:

```shell
kubectl apply -f my-aws-managedcluster1.yaml
```

## Create two clusters on AWS

But instead we’ll use the Makefile to create two clusters:

```shell
make apply-managed-cluster-aws-test1-0.0.1
make apply-managed-cluster-aws-test2-0.0.1
```

Note that it can take 5-10 minutes for the cluster to finish provisioning. You can follow the provisioning process:

```shell
kubectl -n hmc-system get managedcluster.hmc.mirantis.com test1 --watch
```

You can also use the Makefile to show the status and rollout of the cluster as k0rdent sees it:

```shell
make watch-aws-test1
```

When the cluster has been completely deployed, you’ll see a message that says:

```shell
NAME                   READY   STATUS
hmc-system-aws-test1   True    ManagedCluster is ready
```

## Obtain kubeconfigs

Once the deployment is done, you can go ahead and grab the KUBECONFIGs:

```shell
make get-kubeconfig-aws-test1
make get-kubeconfig-aws-test2
```

The script puts the KUBECONFIG for a cluster admin in the kubeconfigs folder.

At this point you have two Kubernetes clusters created by k0rdent, and because they’re just normal Kubernetes clusters, you can access them through kubectl just like any other clusters you’d create:

```shell
KUBECONFIG="kubeconfigs/hmc-system-aws-test1.kubeconfig" kubectl get pods -A
KUBECONFIG="kubeconfigs/hmc-system-aws-test2.kubeconfig" kubectl get pods -A
```

Congratulations, you’ve created your first clusters with k0rdent! Now we invite you to proceed to our [Tutorials]() to see how k0rdent is used, day to day, at scale.
