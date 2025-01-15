# Tutorial 1 - Upgrade a single cluster

k0rdent lets you easily upgrade a managed cluster by changing the ClusterTemplate that defines it or applying a new ClusterTemplate to it.

At the conclusion of [QuickStart 3](quickstart_3_deploy_managed_cluster_aws.md) we deployed two clusters on AWS, using version v1.31.1+k0s.1 of k0s Kubernetes, as specified in the `demo-aws-standalone-cp-0.0.1` template. Now we’ll upgrade one of our clusters (aws-test1) to v1.31.2+k0s.0 by applying the ClusterTemplate named `demo-aws-standalone-cp-0.0.2`.

## Examine the upgrade ClusterTemplate

The first thing we need to do is install the ClusterTemplate for the upgrade, which looks like this:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: ClusterTemplate
metadata:
  name: demo-aws-standalone-cp-0.0.2
  namespace: hmc-system
spec:
  helm:
    chartSpec:
      chart: demo-aws-standalone-cp
      version: 0.0.2
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
  name: demo-aws-standalone-cp-0.0.2
  namespace: hmc-system
spec:
  supportedTemplates:
    - name: demo-aws-standalone-cp-0.0.1
      availableUpgrades:
        - name: demo-aws-standalone-cp-0.0.2
    - name: demo-aws-standalone-cp-0.0.2
```

As you can see, this template is very similar to the original ClusterTemplate, but the ClusterTemplateChain sub-document tells k0rdent that the demo-aws-standalone-cp-0.0.2 template is an upgrade from demo-aws-standalone-cp-0.0.1.

## Install the upgrade ClusterTemplate

Now let’s use the Makefile to install `demo-aws-standalone-cp-0.0.2` on the management cluster:

```shell
make apply-clustertemplate-demo-aws-standalone-cp-0.0.2
```

You’ll see that it creates the appropriate objects:

```shell
clustertemplate.hmc.mirantis.com/demo-aws-standalone-cp-0.0.2 created
clustertemplatechain.hmc.mirantis.com/demo-aws-standalone-cp-0.0.2 created
```

## Confirm the upgrade is available

Once we’ve installed this template, k0rdent knows the upgrade is available, which we can see by looking for available upgrades:

```shell
make get-avaliable-upgrades
```

You should see a result like this. k0rdent traces the ClusterTemplateChain and determines that this upgrade can apply to both of your clusters:

```shell
Cluster hmc-system-aws-test1 available upgrades:
        - demo-aws-standalone-cp-0.0.2

Cluster hmc-system-aws-test2 available upgrades:
        - demo-aws-standalone-cp-0.0.2
```

## Apply the upgrade to one cluster

Now let’s go ahead and apply the upgrade to one of our clusters:

```shell
make apply-managed-cluster-aws-test1-0.0.2
```

## Monitor upgrade rollout

k0rdent automatically applies upgrades to k0s-based clusters using an extremely-reliable, workload-sparing rolling process. We can monitor progress and watch how new machines are created and old machines are deleted:

```shell
PATH=$PATH:./bin kubectl -n hmc-system get machines -w
```

Note that control plane nodes for k0s clusters are upgraded in place (check the version field) without provisioning new machines.

You can also see old nodes are drained and new nodes are attached:

```shell
KUBECONFIG="kubeconfigs/hmc-system-aws-test1.kubeconfig" PATH=$PATH:./bin kubectl get node -w
```

## Take aways

k0rdent not only lets you perform upgrades in a declarative way, but via ClusterTemplateChains, implements guardrail mechanisms that users can be made aware of available upgrades, and ensure that upgrades are processed safely, in proper order. k0s cluster rolling upgrades are efficient and avoid workload impacts.

In [Tutorial 2 - Install a ServiceTemplate into a cluster](tutorial_2_install_service_template_into_single_cluster.md) we'll begin exploring ServiceTemplates &mdash; and see how services can be orchestrated onto clusters, creating complete platforms.
