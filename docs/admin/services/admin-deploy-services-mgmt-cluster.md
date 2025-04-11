# Deploy beach-head services on Management Cluster itself

There is a SveltosCluster object named mgmt in the mgmt namespace where k0rdent has been installed. This object represents the management cluster itself. The following command can be used to get this object along with labels:

```sh
kubectl -n mgmt get sveltoscluster mgmt --show-labels
```

The output should be similar to this:

```sh
NAME   READY   VERSION   LABELS
mgmt   true    v1.32.2   k0rdent.mirantis.com/management-cluster=true,projectsveltos.io/k8s-version=v1.32.2,sveltos-agent=present
```

To deploy beach-head services on the management cluster, a MultiClusterService object can be created that matches the mgmt SveltosCluster using the k0rdent.mirantis.com/management-cluster=true and sveltos-agent=present labels as shown in the following YAML:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: MultiClusterService
metadata:
  name: mgmt-mcs
spec:
  clusterSelector:
    matchLabels:
      k0rdent.mirantis.com/management-cluster: "true"
      sveltos-agent: present
  serviceSpec:
    services:
      - template: ingress-nginx-4-11-3
        name: ingress-nginx
        namespace: ingress-nginx
```

Any number of ServiceTemplates (ingress-nginx-4-11-3 in this example) can be added to the MultiClusterService's `.spec.serviceSpec.services` field. See [Using and Creating ServiceTemplates](./admin-service-templates.md) for how to create ServiceTemplates.

To verify that the ingress-nginx-4-11-3 beach-head service was sucessfully deployed, the status of the MultiClusterService can be queried with:

```sh
kubectl get multiclusterservice mgmt -o yaml
```

The output should be similar to the following showing that ingress-nginx has been Provisioned:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: MultiClusterService
. . .
status:
  . . .
  services:
  - clusterName: mgmt
    clusterNamespace: mgmt
    conditions:
    - lastTransitionTime: "2025-04-24T10:35:03Z"
      message: ""
      reason: Provisioned
      status: "True"
      type: Helm
    - lastTransitionTime: "2025-04-24T10:35:03Z"
      message: Release ingress-nginx/ingress-nginx
      reason: Managing
      status: "True"
      type: ingress-nginx.ingress-nginx/SveltosHelmReleaseReady
```

See [Deploy beach-head services using MultiClusterService](./admin-create-multiclusterservice.md) for more detail on deploying beach-head services using MultiClusterService.