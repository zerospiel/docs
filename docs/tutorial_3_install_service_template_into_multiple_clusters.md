# Tutorial 3 - Install ServiceTemplate into multiple clusters

Now we’re going to look at installing a ServiceTemplate into multiple Clusters. k0rdent lets us automate installation of services at scale (potentially at huge scale) without requiring us to reference service additions in the individual `ManagedCluster` resources for each cluster we want to change.

If you’ve been following along with QuickStarts 1-3 and prior tutorials, you should have two managed clusters to work with (on AWS). We’re going to use ServiceTemplates to install [Kyverno](https://kyverno.io/) on both of them &mdash; forming the core of a solution for policy-as-code enforcement.

## Examine the ServiceTemplate

As before, we’ll start with a ServiceTemplate:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: ServiceTemplate
metadata:
  name: demo-kyverno-3.2.6
  namespace: hmc-system
spec:
  helm:
    chartSpec:
      chart: demo-kyverno
      version: 3.2.6
      interval: 10m0s
      sourceRef:
        kind: HelmRepository
        name: 2a-demos
---
apiVersion: hmc.mirantis.com/v1alpha1
kind: ServiceTemplateChain
metadata:
  name: demo-kyverno-3.2.6
  namespace: hmc-system
spec:
  supportedTemplates:
    - name: demo-kyverno-3.2.6
```

## Install the ServiceTemplate into k0rdent

As you can see, there’s nothing particularly special about this template; it looks like the other base templates we’ve been using.  Let’s go ahead and install it into k0rdent:

```shell
make apply-servicetemplate-demo-kyverno-3.2.6
```

You can see it installed in the system:

```shell
PATH=$PATH:./bin kubectl -n hmc-system get servicetemplates
```

You should be able to see that the template is valid:

```shell
NAME                        VALID
demo-ingress-nginx-4.11.0   true
demo-kyverno-3.2.6          true <-- This is the new template
ingress-nginx-4-11-0        true
ingress-nginx-4-11-3        true
kyverno-3-2-6               true
```

## Define a MultiClusterService to deploy the service on multiple clusters

Instead of always needing to reference a ServiceTemplate in individual ManagedClusters, k0rdent gives us another abstraction for installing ServiceTemplates on multiple target clusters. 

This is called a `MultiClusterService`, and it uses a `clusterSelector` field. Here's an example:

```shell
apiVersion: hmc.mirantis.com/v1alpha1
kind: MultiClusterService
metadata:
  name: global-kyverno
spec:
  servicesPriority: 1000
  clusterSelector:
    matchLabels:
      app.kubernetes.io/managed-by: Helm
  services:
    - template: kyverno-3-2-6
      name: kyverno
      namespace: kyverno
```

As you can see, we’re referencing the kyverno-3-2-6 ServiceTemplate we installed in the last step, then specifying that we want it installed in all clusters that are managed by Helm. Clearly, this is very flexible: we can filter for clusters using many different labels, as well as combinations of labels.

## Apply the MultiClusterService template to install services on clusters

Now we can apply that MultiClusterService template:

```shell
make apply-multiclusterservice-global-kyverno
```

We can see that Kyverno is being installed in both clusters:

```shell
KUBECONFIG="kubeconfigs/hmc-system-aws-test1.kubeconfig" kubectl get pods -n kyverno
KUBECONFIG="kubeconfigs/hmc-system-aws-test2.kubeconfig" kubectl get pods -n kyverno
```

Note that it may take 1-2 minutes before Kyverno begins installing.

## Next up: k0rdent workflows

In our next set of tutorials, we start looking at ways k0rdent helps Platform Architects and Platform Engineering Teams deliver innovation across their organizations. Specifically, we're going to show you (hands-on) how Platform Architects can:

* Approve ClusterTemplates and ServiceTemplates for use by others
* Safely share credentials required for others to use these templates
* Place constraints on how templates can be used (e.g., only within specific namespaces)

We'll then go on to show how Platform Leads and other 'innovation consumers' can use k0rdent safely within these critical guardrails.   
