# Tutorial 2 - Install a ServiceTemplate into a single cluster

Clusters are only useful if developers can build on them, so let’s go ahead and look at that process. 

Just as you can create clusters by referencing templates, you can use ServiceTemplates to create workloads and other services in a managed cluster. These services can be something that’s been prepared and made available by the company or by a vendor, or they can be part of an application built by the project team.

## Examine a ServiceTemplate

In this case, we’re going to install a ServiceTemplate that represents a deployment of Nginx Ingress to the managed cluster. The template looks like this:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: ServiceTemplate
metadata:
  name: demo-ingress-nginx-4.11.0
  namespace: hmc-system
spec:
  helm:
    chartSpec:
      chart: demo-ingress-nginx
      version: 4.11.0
      interval: 10m0s
      sourceRef:
        kind: HelmRepository
        name: 2a-demos
---
apiVersion: hmc.mirantis.com/v1alpha1
kind: ServiceTemplateChain
metadata:
  name: demo-ingress-nginx-4.11.0
  namespace: hmc-system
spec:
  supportedTemplates:
    - name: demo-ingress-nginx-4.11.0
```

As you can see, the template specifies what you need to deploy an app into the Kubernetes cluster: Helm chart information. Also, we’ve got the ServiceTemplateChain, which is similar to the ClusterTemplateChain in that it will specify upgrades (when appropriate).

As before, we’ll start by installing the template into the management cluster so that k0rdent can use it to install to any of the managed clusters:

```shell
make apply-servicetemplate-demo-ingress-nginx-4.11.0
```

Then we’ll apply the ServiceTemplate to the aws-test2 cluster by referencing it in a YAML file:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: ManagedCluster
metadata:
  name: ${NAMESPACE}-aws-${CLUSTERNAME}
  namespace: ${NAMESPACE}
spec:
  template: demo-aws-standalone-cp-0.0.1
  credential: aws-cluster-identity-cred
  config:
    region: us-east-2
    publicIP: true
    controlPlaneNumber: 1
    workersNumber: 2
    controlPlane:
      instanceType: t3.small
    worker:
      instanceType: t3.small
  services:
    - template: demo-ingress-nginx-4.11.0
      name: ingress-nginx
      namespace: ingress-nginx


Notice that there are several placeholders in this template, in the form of ${PLACEHOLDERNAME}. These placeholders are what let you easily use these ServiceTemplates in different contexts.
Otherwise, notice that we’re specifying an entire cluster. Yes, we’re just altering the existing cluster, but as with the upgrade task, we’re counting on k0rdent to reconcile the difference between the current state of the cluster and the desired state. And of course those differences include the addition of services, which references the ServiceTemplate and explains where it should go.
Let’s go ahead and apply it to the management cluster:
make apply-managed-cluster-aws-test2-0.0.1-ingress


You can watch for it to be installed:
watch KUBECONFIG="kubeconfigs/hmc-system-aws-test2.kubeconfig" PATH=$PATH:./bin kubectl get pods -n ingress-nginx

When it’s running, you’ll see the pod:

NAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-86bd747cf9-dxbgq   1/1     Running   0          55s


This technique becomes even more powerful when you want to install into multiple clusters.
