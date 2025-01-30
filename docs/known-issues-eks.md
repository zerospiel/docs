# EKS Machines Are not Created: ControlPlaneIsStable Preflight Check Failed

[Related issue](https://github.com/k0rdent/kcm/issues/907)

The deployment of the EKS cluster is stuck waiting for the machines to be provisioned. The `MachineDeployment`
resource is showing the following conditions:

```
Type: MachineSetReady
Status: False
Reason: PreflightCheckFailed
Message: ekaz-eks-dev-eks-md: AWSManagedControlPlane kcm-system/ekaz-eks-dev-eks-cp is provisioning ("ControlPlaneIsStable" preflight check failed)
Type: Available
Status: False
Reason: WaitingForAvailableMachines
Message: ekaz-eks-dev-eks-md: Minimum availability requires 1 replicas, current 0 available
Type: Ready
Status: False
Reason: WaitingForAvailableMachines
Message: ekaz-eks-dev-eks-md: Minimum availability requires 1 replicas, current 0 available
```

As a result, the cluster was successfully created in EKS but no nodes are available.

**Workaround**

Add `machineset.cluster.x-k8s.io/skip-preflight-checks: "ControlPlaneIsStable"` annotation to skip the
`ControlPlaneIsStable` preflight check to the EKS `MachineDeployment` object:

```bash
kubectl --kubeconfig <management-kubeconfig> annotate MachineDeployment -n <cluster-namespace> <cluster-name>-md machineset.cluster.x-k8s.io/skip-preflight-checks=ControlPlaneIsStable
```
