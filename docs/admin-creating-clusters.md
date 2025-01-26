# Creating and lifecycle-managing managed clusters

Once you've installed k0rdent, you can use it to create, manage, update and even upgrade clusters.

## Deploying a Cluster

k0rdent is designed to simplify the process of deploying and managing Kubernetes clusters across various cloud platforms. It does this through the use of `ClusterDeployment`s, which include all of the information k0rdent needs to know in order to create the cluste you're looking for. This `ClusterDeployment` system relies on predefined templates and credentials. 

A cluster deployment typically involves:

1. Setting up credentials for the infrastructure provider (for example, AWS, vSphere).
2. Choosing a template that defines the desired cluster configuration (for example, number of nodes, instance types).
3. Submitting the configuration for deployment and monitoring the process.

Follow these steps to deploy a standalone Kubernetes cluster tailored to your specific needs:

1. Create the `Credential` object

    Credentials are essential for k0rdent to communicate with the infrastructure provider (for example, AWS, Azure, vSphere). These credentials enable k0rdent to provision resources such as virtual machines, networking components, and storage.

    `Credential` objects are generally created haead of time and made available to users, so before you look into creating a
    new one be sure what you're looking for doesn't already exist. You can see all of the existing `Credential` obects by 
    querying the management cluster:

    ```shell
    kubectl get credentials -n A
    ```

    If the `Credential` you need doesn't yet exist, go ahead and create it.

    Start by creating a `Credential` object that includes all required authentication details for your chosen infrastructure provider. Follow the instructions in the [chapter about credential management](admin-credentials.md), as well as the specific instructions for your [target infrastructure](admin-prepare.md).

    > **Tip:**  
    > Double-check to make sure that your credentials have sufficient permissions to create resources on the target infrastructure.

2. Select a Template

    Templates in k0rdent are predefined configurations that describe how the cluster should be set up. Templates include details such as:
    - The number and type of control plane and worker nodes.
    - Networking settings.
    - Regional deployment preferences.

    Templates act as a blueprint for creating a cluster. To see the list of available templates, use the following command:

    ```shell
    kubectl get clustertemplate -n kcm-system
    ```
    ```console
    NAMESPACE    NAME                            VALID
    kcm-system   adopted-cluster-0-0-2           true
    kcm-system   aws-eks-0-0-3                   true
    kcm-system   aws-hosted-cp-0-0-4             true
    kcm-system   aws-standalone-cp-0-0-5         true
    kcm-system   azure-aks-0-0-2                 true
    kcm-system   azure-hosted-cp-0-0-4           true
    kcm-system   azure-standalone-cp-0-0-5       true
    kcm-system   openstack-standalone-cp-0-0-2   true
    kcm-system   vsphere-hosted-cp-0-0-5         true
    kcm-system   vsphere-standalone-cp-0-0-5     true
    ```

    You can then get information on the actual template by describing it, as in:

    ```shell
    kubectl describe clustertemplate aws-standalone-cp-0-0-5 -n ksm-system
    ```

3. Create a ClusterDeployment YAML Configuration

    The `ClusterDeployment` object is the main configuration file that defines your cluster's specifications. It includes:
    - The template to use.
    - The credentials for the infrastructure provider.
    - Optional customizations such as instance types, regions, and networking.

    Create a `ClusterDeployment` configuration in a YAML file, following this structure:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: <cluster-name>
      namespace: <kcm-system-namespace>
    spec:
      template: <template-name>
      credential: <infrastructure-provider-credential-name>
      dryRun: <"true" or "false" (default: "false")>
      config:
        <cluster-configuration>
    ```

    You will of course want to replace the placeholders with actual values. (For more information about `dryRun` see [Understanding the Dry Run](appendix.md#understanding-the-dry-run).) For example, this is a simple AWS infrastructure provider `ClusterDeployment`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: my-managed-cluster
      namespace: kcm-system
    spec:
      template: aws-standalone-cp-0-0-3
      credential: aws-credential
      dryRun: false
      config:
        region: us-west-2
        controlPlane:
          instanceType: t3.small
        worker:
          instanceType: t3.small
    ```
    Note that the `.spec.credential` value should match the `.metadata.name` value of a created `Credential` object.

4. Apply the Configuration

    Once the `ClusterDeployment` configuration is ready, apply it to the k0rdent management cluster:

    ```shell
    kubectl apply -f clusterdeployment.yaml
    ```

    This step submits your deployment request to k0rdent. If you've set `dryRun` to `true` you can observe what would happen. Otherwise, k0rdent will go ahead and begin provisioning the necessary infrastructure.

5. Verify Deployment Status

    After submitting the configuration, verify that the `ClusterDeployment` object has been created successfully:

    ```shell
    kubectl -n <namespace> get clusterdeployment.kcm <cluster-name> -o=yaml
    ```

    The output shows the current status and any errors.

6. Monitor Provisioning

    k0rdent will now start provisioning resources (e.g., VMs, networks) and setting up the cluster. To monitor this process, run:

    ```shell
    kubectl -n <namespace> get cluster <cluster-name> -o=yaml
    ```

    > **Tip:**  
    > For a detailed view of the provisioning process, use the `clusterctl describe` command (note that this requires the [`clusterctl`](https://github.com/kubernetes-sigs/cluster-api/releases) CLI):

    ```shell
    clusterctl describe cluster <cluster-name> -n <namespace> --show-conditions all
    ```

7. Retrieve the Kubernetes Configuration

    When provisioning is complete, retrieve the kubeconfig file for the new cluster. This file enables you to interact with the cluster using `kubectl`:

    ```shell
    kubectl get secret -n <namespace> <cluster-name>-kubeconfig -o=jsonpath={.data.value} | base64 -d > kubeconfig
    ```
    You can then use this file to access the cluster, as in:

    ```shell
    export KUBECONFIG=kubeconfig
    kubectl get pods -A
    ```

    Store the kubeconfig file securely, as it contains authentication details for accessing the cluster.

## Adopting an Existing Cluster

Creating a new cluster isn't the only way to use k0rdent. Adopting an existing Kubernetes cluster enables you to 
bring it under the k0rdent's management. This process is useful when you already have a running cluster but want 
to centralize management and leverage k0rdent's capabilities, such as unified monitoring, configuration, and automation.

To adopt a cluster, k0rdent establishes communication between the management cluster (where kcm is installed) 
and the target cluster. This requires proper credentials, network connectivity, and a standardized configuration. 

Follow these steps to adopt an existing cluster:

1. Prerequisites

    Before you start, make sure you have the following:

    - A kubeconfig file for the cluster you want to adopt (this file provides access credentials and configuration details 
      for the cluster).
    - A management cluster with k0rdent installed and running. See the [installation instructions](admin-installation.md) 
      if you need to set it up.
    - Network connectivity between the management cluster and the cluster to be adopted (for example, ensure firewall 
      rules and VPNs allow communication).

2.  Create a Credential

    Start by creating a `Credential` object that includes all required authentication details for your chosen infrastructure 
    provider. Follow the instructions in the [Credential System](admin-credentials.md), as well as the specific instructions 
    for your [target infrastructure](admin-prepare.md).

    > **Tip:**  
    > Double-check that your credentials have sufficient permissions to create resources on the target infrastructure.

3. Configure the Adopted Cluster Template

    Set the `KUBECONFIG` environment variable to the path of your management cluster's kubeconfig file so you can 
    execute commands against the management cluster.

    For example:

    ```shell
    export KUBECONFIG=/path/to/management-cluster-kubeconfig
    ```

4. Create the `ClusterDeployment` YAML Configuration

    The `ClusterDeployment` object is used to define how k0rdent should manage the adopted cluster. Create a 
    YAML file for the `ClusterDeployment` object, as shown below:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: <cluster-name>
      namespace: <kcm-system-namespace>
    spec:
      template: adopted-cluster-<template-version>
      credential: <credential-name>
      dryRun: <"true" or "false" (default: "false")>
      config:
        <cluster-configuration>
    ```

    Replace placeholders like `<cluster-name>` and `<credential-name>` with actual values. The `dryRun` flag is useful for testing the configuration without making changes to the cluster. For more details, see the [Dry Run](#understanding-the-dry-run) section.

    You can also get a list of the available templates with:

    ```shell
    kubectl get clustertemplate -n kcm-system
    ```
    ```console
    NAMESPACE    NAME                            VALID
    kcm-system   adopted-cluster-0-0-2           true
    kcm-system   aws-eks-0-0-3                   true
    kcm-system   aws-hosted-cp-0-0-4             true
    kcm-system   aws-standalone-cp-0-0-5         true
    kcm-system   azure-aks-0-0-2                 true
    kcm-system   azure-hosted-cp-0-0-4           true
    kcm-system   azure-standalone-cp-0-0-5       true
    kcm-system   openstack-standalone-cp-0-0-2   true
    kcm-system   vsphere-hosted-cp-0-0-5         true
    kcm-system   vsphere-standalone-cp-0-0-5     true

    Putting it all together, your YAML would look something like this:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: my-cluster
      namespace: kcm-system
    spec:
      template: adopted-cluster-0-0-2
      credential: my-cluster-credential
      dryRun: false
    ```

5. Apply the `ClusterDeployment` configuration

    Once your configuration file is ready, apply it to the management cluster using `kubectl`:

    ```shell
    kubectl apply -f clusterdeployment.yaml
    ```

    This step submits the `ClusterDeployment` object to k0rdent, initiating the adoption process.

6. Check the Status of the `ClusterDeployment` Object

    To ensure the adoption process is progressing as expected, check the status of the `ClusterDeployment` object:

    ```shell
    kubectl -n <namespace> get clusterdeployment.kcm <cluster-name> -o=yaml
    ```

    The output includes the current state and any conditions (for example, errors or progress updates). Review 
    this information to confirm that the adoption is successful.

### What’s Happening Behind the Scenes?

When you adopt a cluster, k0rdent performs several actions:
1. It validates the credentials and configuration provided in the `ClusterDeployment` object.
2. It ensures network connectivity between the management cluster and the adopted cluster.
3. It registers the adopted cluster within the k0rdent system, enabling it to be monitored and managed like 
   any k0rdent-deployed cluster.

This process doesn’t change the adopted cluster’s existing workloads or configurations. Instead, it enhances your 
ability to manage the cluster through k0rdent.

### Additional Tips
- If you encounter issues, double-check that kubeconfig file you used for the adopted cluster is valid 
  and matches the cluster you’re trying to adopt.
- Use the [`dryRun`](appendix.md#understanding-the-dry-run) option during the first attempt to validate the configuration without making actual changes.

## Deploying a Hosted Control Plane

A hosted control plane is a Kubernetes setup in which the control plane components (such as the API server, 
etcd, and controllers) run inside the management cluster instead of separate controller nodes. This 
architecture centralizes control plane management, and improves scalability by sharing resources in the management cluster.
Hosted control planes are managed by [k0smotron](https://k0smotron.io/).

Instructions for setting up a hosted control plane vary slighting depending on the provider.

### AWS Hosted Control Plane Deployment

Follow these steps to set up a k0smotron-hosted control plane on AWS: 

1. Prerequisites

    Before proceeding, make sure you have the following:

    - A management Kubernetes cluster (Kubernetes v1.28 or later) deployed on AWS with [k0rdent installed](admin-installation.md).
    - A [default storage class](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/) configured on the management cluster to support Persistent Volumes.
    - The VPC ID where the worker nodes will be deployed.
    - The Subnet ID and Availability Zone (AZ) for the worker nodes.
    - The AMI ID for the worker nodes (Amazon Machine Image ID for the desired OS and Kubernetes version).

    > **Important:**  
    > All control plane components for your hosted cluster will reside in the management cluster. The management cluster 
      must have sufficient resources to handle these additional workloads.

2. Networking

    To deploy a hosted control plane, the necessary AWS networking resources must already exist or be created. If you're 
    using the same VPC and subnets as your management cluster, you can resuse these resources.

    If your management cluster was deployed using the Cluster API Provider AWS (CAPA), you can gather the required 
    networking details using the following commands:

    Retrieve the VPC ID:
    ```shell
    kubectl get awscluster <cluster-name> -o go-template='{{.spec.network.vpc.id}}'
    ```

    Retrieve Subnet ID:
    ```shell
    kubectl get awscluster <cluster-name> -o go-template='{{(index .spec.network.subnets 0).resourceID}}'
    ```

    Retrieve Availability Zone:
    ```shell
    kubectl get awscluster <cluster-name> -o go-template='{{(index .spec.network.subnets 0).availabilityZone}}'
    ```

    Retrieve Security Group:
    ```shell
    kubectl get awscluster <cluster-name> -o go-template='{{.status.networkStatus.securityGroups.node.id}}'
    ```

    Retrieve AMI ID:
    ```shell
    kubectl get awsmachinetemplate <cluster-name>-worker-mt -o go-template='{{.spec.template.spec.ami.id}}'
    ```

    > **Tip:**  
    > If you want to use different VPCs or regions for your management and hosted clusters, you’ll need to configure additional networking, such as [VPC peering](https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/vpc-peering.html), to allow communication between them.


3. Create the ClusterDeployment manifest

    Once you've collected all the necessary data, you can create the `ClusterDeployment` manifest. This file tells k0rdent how to 
    deploy and manage the hosted control plane. Below is an example:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: aws-hosted-cp
    spec:
      template: aws-hosted-cp-0-0-3
      credential: aws-credential
      config:
        vpcID: vpc-0a000000000000000
        region: us-west-1
        publicIP: true
        subnets:
          - id: subnet-0aaaaaaaaaaaaaaaa
            availabilityZone: us-west-1b
            isPublic: true
            natGatewayID: xxxxxx
            routeTableId: xxxxxx
          - id: subnet-1aaaaaaaaaaaaaaaa
            availabilityZone: us-west-1b
            isPublic: false
            routeTableId: xxxxxx
        instanceType: t3.medium
        securityGroupIDs:
          - sg-0e000000000000000
    ```

    > **Note:**  
    > The example above uses the `us-west-1` region, but you should use the region of your VPC.

    #### Generate the `ClusterDeployment` Manifest

    To simplify the creation of a `ClusterDeployment` manifest, you can use the following template, which dyamically 
    inserts the appropriate values:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: aws-hosted
    spec:
      template: aws-hosted-cp-0-0-3
      credential: aws-credential
      config:
        vpcID: "{{.spec.network.vpc.id}}"
        region: "{{.spec.region}}"
        subnets:
        {{- range $subnet := .spec.network.subnets }}
          - id: "{{ $subnet.resourceID }}"
            availabilityZone: "{{ $subnet.availabilityZone }}"
            isPublic: {{ $subnet.isPublic }}
            {{- if $subnet.isPublic }}
            natGatewayId: "{{ $subnet.natGatewayId }}"
            {{- end }}
            routeTableId: "{{ $subnet.routeTableId }}"
        {{- end }}
        instanceType: t3.medium
        securityGroupIDs:
          - "{{.status.networkStatus.securityGroups.node.id}}"
    ```

    Save this template as `clusterdeployment.yaml.tpl`, then generate your manifest using the following command:

    ```shell
    kubectl get awscluster <cluster-name> -o go-template="$(cat clusterdeployment.yaml.tpl)" > clusterdeployment.yaml
    ```

4. Apply the `ClusterTemplate`

    Nothing actually happens until you apply the `ClusterDeployment` manifest to create a new cluster deployment:

    ```shell
    kubectl apply -f clusterdeployment.yaml -n kcm-system
    ```

#### Deployment Tips

Here are some additional tips to help with deployment:

1. Controller and Template Availability:

   Make sure the kcm controller image and templates are available in a public or accessible repository.

2. Install Charts and Templates:

   If you're using a custom repository, run the following commands with the appropriate `kubeconfig`:

   ```shell
   KUBECONFIG=kubeconfig IMG="ghcr.io/k0rdent/kcm/controller-ci:v0.0.1-179-ga5bdf29" REGISTRY_REPO="oci://ghcr.io/k0rdent/kcm/charts-ci" make dev-apply
   KUBECONFIG=kubeconfig make dev-templates
   ```

3. Mark Infrastructure as Ready:

   To scale up the `MachineDeployment`, manually mark the infrastructure as ready:
   ```shell
   kubectl patch AWSCluster <hosted-cluster-name> --type=merge --subresource status --patch '{"status": {"ready": true}}' -n kcm-system
   ```
   For more details on why this is necessary, [click here](https://docs.k0smotron.io/stable/capi-aws/#:~:text=As%20we%20are%20using%20self%2Dmanaged%20infrastructure%20we%20need%20to%20manually%20mark%20the%20infrastructure%20ready.%20This%20can%20be%20accomplished%20using%20the%20following%20command).

### Azure Hosted Control Plane Deployment

Follow these steps to set up a k0smotron-hosted control plane on Azure:

1. Prerequisites

    Before you start, make sure you have the following:

    - A management Kubernetes cluster (Kubernetes v1.28+) deployed on Azure with [k0rdent installed](admin-installation.md).
    - A [default storage class](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/) configured 
      on the management cluster to support Persistent Volumes.

    > **Note:**  
    > All control plane components for managed clusters will run in the management cluster. Make sure the management cluster 
      has sufficient CPU, memory, and storage to handle the additional workload.

2.  Gather pre-existing resources

    In a hosted control plane setup, some Azure resources must exist before deployment and must be explicitly 
    provided in the `ClusterDeployment` configuration. These resources can also be reused by the management cluster.

    If you deployed your Azure Kubernetes cluster using the Cluster API Provider for Azure (CAPZ), you can retrieve 
    the required information using the following commands:

    Location:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{.spec.location}}'
    ```

    Subscription ID:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{.spec.subscriptionID}}'
    ```

    Resource Group:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{.spec.resourceGroup}}'
    ```

    VNet Name:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{.spec.networkSpec.vnet.name}}'
    ```

    Subnet Name:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{(index .spec.networkSpec.subnets 1).name}}'
    ```

    Route Table Name:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{(index .spec.networkSpec.subnets 1).routeTable.name}}'
    ```

    Security Group Name:
    ```shell
    kubectl get azurecluster <cluster-name> -o go-template='{{(index .spec.networkSpec.subnets 1).securityGroup.name}}'
    ```

3. Create the ClusterDeployment manifest

    After collecting the required data, create a `ClusterDeployment` manifest to configure the hosted control plane. It should
    look something like this:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: azure-hosted-cp
    spec:
      template: azure-hosted-cp-0-0-2
      credential: azure-credential
      config:
        location: "westus"
        subscriptionID: ceb131c7-a917-439f-8e19-cd59fe247e03
        vmSize: Standard_A4_v2
        resourceGroup: mgmt-cluster
        network:
          vnetName: mgmt-cluster-vnet
          nodeSubnetName: mgmt-cluster-node-subnet
          routeTableName: mgmt-cluster-node-routetable
          securityGroupName: mgmt-cluster-node-nsg
    ```

    #### Generate a `ClusterDeployment` Manifest

    To simplify the creation of a `ClusterDeployment` manifest, you can use the following template, which dyamically inserts 
    the appropriate values:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: azure-hosted-cp
    spec:
      template: azure-hosted-cp-0-0-2
      credential: azure-credential
      config:
        location: "{{.spec.location}}"
        subscriptionID: "{{.spec.subscriptionID}}"
        vmSize: Standard_A4_v2
        resourceGroup: "{{.spec.resourceGroup}}"
        network:
          vnetName: "{{.spec.networkSpec.vnet.name}}"
          nodeSubnetName: "{{(index .spec.networkSpec.subnets 1).name}}"
          routeTableName: "{{(index .spec.networkSpec.subnets 1).routeTable.name}}"
          securityGroupName: "{{(index .spec.networkSpec.subnets 1).securityGroup.name}}"
    ```

    Save this YAML as `clusterdeployment.yaml.tpl` and render the manifest with the following command:
    ```shell
    kubectl get azurecluster <management-cluster-name> -o go-template="$(cat clusterdeployment.yaml.tpl)" > clusterdeployment.yaml
    ```

4. Create the `ClusterDeployment`

    To actually create the cluster, apply the `ClusterDeployment` manifest to the management cluster, as in:

    ```shell
    kubectl apply clusterdeployment.yaml -n kcm-system
    ```
      
5. Manually update the `AzureCluster` object

    Due to a limitation on k0smotron, (see [k0sproject/k0smotron#668](https://github.com/k0sproject/k0smotron/issues/668)), 
    after applying the `ClusterDeployment` manifest, you must manually update the status of the `AzureCluster` object.

    Use the following command to set the `AzureCluster` object status to `Ready`:

    ```shell
    kubectl patch azurecluster <cluster-name> --type=merge --subresource status --patch '{"status": {"ready": true}}'
    ```

#### Important Notes on Cluster Deletion

Due to these same k0smotron limitations, you **must** take some manual steps in order to delete a cluster properly:

1. Add a Custom Finalizer to the AzureCluster Object:

   To prevent the `AzureCluster` object from being deleted too early, add a custom finalizer:

   ```shell
   kubectl patch azurecluster <cluster-name> --type=merge --patch '{"metadata": {"finalizers": ["manual"]}}'
   ```

2. Delete the ClusterDeployment:
   After adding the finalizer, delete the `ClusterDeployment` object as usual. Confirm that all `AzureMachines` objects have been deleted successfully.

3. Remove Finalizers from Orphaned AzureMachines:

   If any `AzureMachines` are left orphaned, delete their finalizers manually after confirming no VMs remain in Azure. Use this command to remove the finalizer:

   ```shell
   kubectl patch azuremachine <machine-name> --type=merge --patch '{"metadata": {"finalizers": []}}'
   ```

4. Allowing Updates to Orphaned Objects:

    If Azure admission controls prevent updates to orphaned objects, you must disable the associated `MutatingWebhookConfiguration` by deleting it:

    ```shell
    kubectl delete mutatingwebhookconfiguration <webhook-name>
    ```

### vSphere Hosted Control Plane Deployment

Follow these steps to set up a k0smotron-hosted control plane on vSphere. 

1. Prerequisites

    Before you start, make sure you have the following:

    - A management Kubernetes cluster (Kubernetes v1.28+) deployed on vSphere with [k0rdent installed](admin-installation.md).

    All control plane components for managed clusters will reside in the management cluster. Make sure the management 
    cluster has sufficient resources (CPU, memory, and storage) to handle these workloads.

2. Create the `ClusterDeployment` Manifest

The `ClusterDeployment` manifest for vSphere-hosted control planes is similar to standalone control plane deployments. 
For a detailed list of parameters, refer to the [template parameters](template.md) section.

> **Important:**  
> The vSphere provider requires you to specify the **control plane endpoint IP** before deploying the cluster. This IP 
> address must match the one assigned to the k0smotron load balancer (LB) service.  
> Use an annotation supported by your load balancer provider to assign the control plane endpoint IP to the k0smotron 
> service. For example, the manifest below includes a `kube-vip` annotation.

`ClusterDeployment`s for vSphere-based clusters include a `.spec.config.vsphere` object that contains vSphere-specific
parameters. For example:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: ClusterDeployment
metadata:
  name: cluster-1
spec:
  template: vsphere-hosted-cp-0-0-2
  credential: vsphere-credential
  config:
    vsphere:
      server: vcenter.example.com
      thumbprint: "00:00:00"
      datacenter: "DC"
      datastore: "/DC/datastore/DC"
      resourcePool: "/DC/host/vCluster/Resources/ResPool"
      folder: "/DC/vm/example"
    controlPlaneEndpointIP: "172.16.0.10"
    ssh:
      user: ubuntu
      publicKey: |
        ssh-rsa AAA...
    rootVolumeSize: 50
    cpus: 2
    memory: 4096
    vmTemplate: "/DC/vm/template"
    network: "/DC/network/Net"
    k0smotron:
      service:
        annotations:
          kube-vip.io/loadbalancerIPs: "172.16.0.10"
```

For more information on these parameters, see the [Reference Guide](template.md). 

## Updating a Single Standalone Cluster

k0rdent `ClusterTemplate`s are immutable, so the only way to change a `ClusterDeployment` is to change the template
that forms its basis. 

To update the `ClusterDeployment`, modify the `.spec.template` field to use the name of the new `ClusterTemplate`. 
This enables you to apply changes to the cluster configuration. These changes will then be applied to the actual 
cluster. For example, if the cluster currently uses `t2.large` instances, that will be specified in its current template. 
To change the cluster to use `t2.xlarge` instances, you would simply apply a template that references that new size; 
k0rdent will then realize the cluster is out of sync and will attempt to remedy the situation by updating the cluster.

Follow these steps to update the `ClusterDeployment`:

1. Patch the `ClusterDeployment` with the new template:

   Run the following command, replacing the placeholders with the appropriate values:

   ```shell
   kubectl patch clusterdeployment.kcm <cluster-name> -n <namespace> --patch '{"spec":{"template":"<new-template-name>"}}' --type=merge
   ```

2. Check the status of the `ClusterDeployment`:

   After applying the patch, verify the status of the `ClusterDeployment` object:

   ```shell
   kubectl get clusterdeployment.kcm <cluster-name> -n <namespace>
   ```

3. Inspect the detailed status:

   For more details, use the `-o=yaml` option to check the `.status.conditions` field:

   ```shell
   kubectl get clusterdeployment.kcm <cluster-name> -n <namespace> -o=yaml
   ```

Note that not all updates are possible; `ClusterTemplateChain` objects limit what templates can be applied.  Consider,
for example, this `ClusterTemplateChain`:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: ClusterTemplateChain
metadata:
  name: aws-standalone-cp-0.0.2
  namespace: kcm-system
spec:
  supportedTemplates:
    - name: aws-standalone-cp-0.0.1
      availableUpgrades:
        - name: aws-standalone-cp-0.0.2
    - name: aws-standalone-cp-0.0.2
```

As you can see from the `.spec`, the `aws-standalone-co-0.0.2` template can be applied to a cluster that also uses
the `aws-standalone-co-0.0.2` template, or it can be used as an upgrade from a cluster that uses `aws-standalone-co-0.0.1`.
You wouldn't be able to use this template to update a cluster that uses any other `ClusterTemplate`.

Similarly, the `AccessManagement` object must have properly configured `spec.accessRules` with a list of allowed 
`ClusterTemplateChain` object names and their namespaces. For more information, see [Template Life Cycle Management](template.md).

> **Future Note:**  
> Support for displaying all available `ClusterTemplates` for updates in the `ClusterDeployment` status is planned.

## Scaling a Cluster , <!-- TODO -->
## Upgrading a Single Standalone Cluster , <!-- TODO -->

## Cleanup

Especially when you are paying for cloud resources, it's crucial to clean up when you're finished.
Fortunately, k0rdent makes that straightforward.

Because a Kubernetes cluster is represented by a `ClusterDeployment`, when you delete that `ClusterDeployment`,
k0rdent deletes the cluster. For example:

```shell
kubectl delete clusterdeployment my-cluster-deployment -n kmc-system
```
```console
ClusterDeployment my-cluster-deployment deleted.
```

It takes time to delete these resources.