# Installation

k0rdent installation is pretty straightforward. As discussed in Architecture, k0rdent runs in a management Kubernetes cluster, so installation involves creating that cluster and adding k0rdent to it. Creating managed clusters is handled by k0rdent itself. This section of the guide covers:

- Creating the management cluster
- Preparing k0rdent to create managed clusters on multiple providers
- Creating and destroying managed clusters
- Creating, testing, and approving cluster templates for use by others
- Creating, testing, and approving service templates for use by others
- Extended management configuration
- Upgrading k0rdent

## Prerequisites

Before you can install k0rdent, you'll need to create the substrate in which it will run.

### Create the management cluster server

The k0rdent management cluster can run on a single Ubuntu server. Where that server resides isn't important, as long as it can reach the providers on which you want to run managed clusters. For example, you can run k0rdent in a management cluster on a VMware VM, and use it to create clusters on AWS, Azure, and so on. This platform independence makes it easier to manage multi-cloud installations. You can even run the management cluster on a (suitably performant) laptop.

What's more, while the management cluster can perform operations on the managed clusters, these managed clusters are essentially independent of it. Once they've been created, they are simply Kubernetes clusters; if the Management Cluster goes away, they will continue to function unimpeded.

For example, you can create and configure a series of edge clusters, then shut down the management cluster until you need to make changes at the edge.

The management cluster server should meet the following minimum requirements:

[[[[ INSERT MINIMUM REQUIREMENTS HERE. Include kubectl and Helm. ]]]]

Once this server is in place, move on to creating the actual management cluster.

### Create and prepare a Kubernetes cluster

If you already have a Kubernetes cluster into which you want to install k0rdent, skip to the next step. Otherwise, follow these steps to install and prepare the management cluster:

1. Install Docker

    Kubernetes, of course, requires Docker. You can find the [full instructions for installing Docker](https://docs.docker.com/engine/install/ubuntu/ "Docker installation instructions") at the Docker site, but here's a simplified version. 

    First uninstall older, potentially conflicting versions:

    ```shell
    for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
    ```

    Then use the Docker convenience script to deploy Docker:

    ```shell
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

    It's not necessary to prepare Docker for non-root usage.

2. Deploy a Kubernetes cluster

    The next step is to create the actual cluster itself. Note that the actual distribution used for the management cluster isn't important, as long as it's a CNCF-compliant distribution. That means you can use an existing EKS cluster, or whatever is your normal corporate standard. To make things simple this guide uses k0s, a small, convenient, and fully-functional distribution:

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://get.k0s.sh | sudo sh
    sudo k0s install controller --single
    sudo k0s start
    ```

    k0s includes its own preconfigured version of `kubectl` so make sure the cluster is running:

    ```shell
    sudo k0s kubectl get nodes
    ```

    You should see a single node with a status of `Ready`, as in:

    ```shell
    NAME              STATUS   ROLES    AGE   VERSION
    ip-172-31-29-61   Ready    <none>   46s   v1.31.2+k0s
    ```

3. Install kubectl

    Everything you do in k0rdent is done by creating and manipulating Kubernetes objects, so you'll need to have `kubectl` installed:

    ```shell
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
    sudo apt-get update
    sudo apt-get install -y kubectl
    ```

4. Get the kubeconfig

    In order to access the management cluster you will, of course, need the kubeconfig. Again, if you're using another Kubernetes distribution follow those instructions to get the kubeconfig, but for k0s, the process involves simply copying the existing file and adding it to an environment varable so `kubectl` knows where to find it.

    ```shell
    sudo cp /var/lib/k0s/pki/admin.conf KUBECONFIG
    sudo chmod +r KUBECONFIG
    export KUBECONFIG=./KUBECONFIG
    ```
    Now you should be able to use the non-k0s `kubectl` to see the status of the cluster:

    ```shell
    kubectl get nodes
    ```

    Again, you should see the single k0s node, but by this time it should have had its role assigned, as in:

    ```console
    NAME              STATUS   ROLES           AGE   VERSION
    ip-172-31-29-61   Ready    control-plane   25m   v1.31.2+k0s
    ```

    Now the cluster is ready for installation, which we'll do using Helm.

5. Install Helm

    The easiest way to install k0rdent is through its Helm chart, so let's get Helm installed:

    ```shell
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh
    ```

    Helm will be installed into `/usr/local/bin/helm`.

## Install k0rdent

The actual management cluster is a Kubernetes cluster with the k0rdent application installed. The simplest way to install k0rdent is through its Helm chart:

```shell
helm install kcm oci://ghcr.io/k0rdent/kcm/charts/kcm --version 0.0.7 -n kcm-system --create-namespace
```
```console
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: ./KUBECONFIG
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: ./KUBECONFIG
Pulled: ghcr.io/mirantis/hmc/charts/hmc:0.0.3
Digest: sha256:1f75e8e55c44d10381d7b539454c63b751f9a2ec6c663e2ab118d34c5a21087f
NAME: hmc
LAST DEPLOYED: Mon Dec  9 00:32:14 2024
NAMESPACE: hmc-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

While the installation process appears to be done at this point, it's not; multiple pods and templates are being created in the background, and the entire process takes a few minutes.  

To understand whether installation is complete, start by making sure all pods are ready in the `kcm-system` namespace. There should be 15:

```shell
kubectl get pods -n kcm-system
```
```console
NAME                                                           READY   STATUS
azureserviceoperator-controller-manager-86d566cdbc-rqkt9       1/1     Running
capa-controller-manager-7cd699df45-28hth                       1/1     Running
capi-controller-manager-6bc5fc5f88-hd8pv                       1/1     Running
capv-controller-manager-bb5ff9bd5-7dsr9                        1/1     Running
capz-controller-manager-5dd988768-qjdbl                        1/1     Running
helm-controller-76f675f6b7-4d47l                               1/1     Running
kcm-cert-manager-7c8bd964b4-nhxnq                              1/1     Running
kcm-cert-manager-cainjector-56476c46f9-xvqhh                   1/1     Running
kcm-cert-manager-webhook-69d7fccf68-s46w8                      1/1     Running
kcm-cluster-api-operator-79459d8575-2s9jc                      1/1     Running
kcm-controller-manager-64869d9f9d-zktgw                        1/1     Running
k0smotron-controller-manager-bootstrap-6c5f6c7884-d2fqs        2/2     Running
k0smotron-controller-manager-control-plane-857b8bffd4-zxkx2    2/2     Running
k0smotron-controller-manager-infrastructure-7f77f55675-tv8vb   2/2     Running
source-controller-5f648d6f5d-7mhz5                             1/1     Running
```

State management is handled by Project Sveltos, so you'll want to make sure that all 9 pods are running in the `projectsveltos` namespace:

```shell
kubectl get pods -n projectsveltos
```
```console
NAME                                     READY   STATUS    RESTARTS   AGE
access-manager-cd49cffc9-c4q97           1/1     Running   0          16m
addon-controller-64c7f69796-whw25        1/1     Running   0          16m
classifier-manager-574c9d794d-j8852      1/1     Running   0          16m
conversion-webhook-5d78b6c648-p6pxd      1/1     Running   0          16m
event-manager-6df545b4d7-mbjh5           1/1     Running   0          16m
hc-manager-7b749c57d-5phkb               1/1     Running   0          16m
sc-manager-f5797c4f8-ptmvh               1/1     Running   0          16m
shard-controller-767975966-v5qqn         1/1     Running   0          16m
sveltos-agent-manager-56bbf5fb94-9lskd   1/1     Running   0          15m
```

If any of these pods are missing, simply give k0rdent more time. If there's a problem, you'll see pods crashing and restarting, and you can review the logs at [[[ ADD LOG LOCATION HERE ]]]. As long as that's not happening, you just need to wait a few minutes.

Next verify whether the kcm templates have been successfully installed and reconciled.  Start with the `ProviderTemplate` objects:

[[[ ADD ACTUAL VERSION NUMBERS TO THE OUTPUT ]]]

```shell
kubectl get providertemplate -n kcm-system
```
```console
NAME                                 VALID
cluster-api-X-Y-Z                    true
cluster-api-provider-aws-X-Y-Z       true
cluster-api-provider-azure-X-Y-Z     true
cluster-api-provider-vsphere-X-Y-Z   true
kcm-X-Y-Z                            true
k0smotron-X-Y-Z                      true
projectsveltos-X-Y-Z                 true
```

Make sure that all templates are not just installed, but valid. Again, this may take a few minutes.

You'll also want to make sure the `ClusterTemplate` objects are installed and valid:

[[[ ADD ACTUAL VERSION NUMBERS TO THE OUTPUT ]]]

```shell
kubectl get clustertemplate -n kcm-system
```
```console
NAME                                VALID
aws-eks-X-Y-Z                       true
aws-hosted-cp-X-Y-Z                 true
aws-standalone-cp-X-Y-Z             true
azure-hosted-cp-X-Y-Z               true
azure-standalone-cp-X-Y-Z           true
vsphere-hosted-cp-X-Y-Z             true
vsphere-standalone-cp-X-Y-Z         true
```

Finally, make sure the `ServiceTemplate` objects are installed and valid:

```shell
kubectl get servicetemplate -n kcm-system
```
```console
NAME                      VALID
cert-manager-1-16-2       true
dex-0-19-1                true
external-secrets-0-11-0   true
ingress-nginx-4-11-0      true
ingress-nginx-4-11-3      true
kyverno-3-2-6             true
velero-8-1-0              true
```


### Air-gapped Installation

> WARNING:
> Currently only the vSphere infrastructure provider supports full air-gapped
> installation.

#### Prerequisites

In order to install k0rdent in an air-gapped environment, you need will need the
following:

- A k0s cluster installed on vSphere. This cluster will act as the management cluster. (While k0rdent works with any certified Kubernetes distribution, k0s implements an OCI image bundle watcher that enables k0s to easily utilize a bundle of management cluster images. You can follow the [Airgapped Installation](https://docs.k0sproject.io/head/airgap-install/#airgap-install) documentation to create this cluster if necessary.
- The `KUBECONFIG` of the management cluster.
- A registry that is accessible from the airgapped hosts in which to store the k0rdent images. If you don't have a registry, you can deploy a [local Docker registry](https://distribution.github.io/distribution/) or use [mindthegap](https://github.com/mesosphere/mindthegap?tab=readme-ov-file#serving-a-bundle-supports-both-image-or-helm-chart) to provide one.

    > WARNING:
    > If using a local Docker registry, ensure the registry URL is added to
    > the `insecure-registries` key within the Docker `/etc/docker/daemon.json`
    > file, as in:
    > ```json
    > {
    >   "insecure-registries": ["<registry-url>"]
    > }
    > ```

- A registry and associated chart repository for hosting kcm charts.  At this
  time, all k0rdent charts MUST be hosted in a single OCI chart repository.  See
  [Use OCI-based registries](https://helm.sh/docs/topics/registries/) in the
  Helm documentation for more information.
- [jq](https://jqlang.github.io/jq/download/), Helm and Docker binaries installed on the machine from which the `airgap-push.sh` script (see below) will run. [[[ THIS IS THE SAME MACHINE WHERE THE CLUSTER IS RUNNING, NO? ]]]


#### Installation

1. Download the kcm airgap bundle from [[[ INSERT URL HERE ]]]. This bundle contains:

    - `images/kcm-images-<version>.tgz` - The image bundle tarball for the
      management cluster. This bundle is loaded into the management
      cluster.
    - `images/kcm-extension-images-<version>.tgz` - The image bundle tarball for
      the managed clusters. This bundle will be pushed to a registry where the
      images can be accessed by managed clusters when deployed.
    - `charts` - Contains the kcm Helm chart, dependency charts and k0s
      extensions charts within the `extensions` directory.  All of these charts
      get pushed to a chart repository within a registry.
    - `scripts/airgap-push.sh` - A script that will aid in re-tagging and
      pushing the `ClusterDeployment` required charts and images to a desired
      registry.

2. Log into the container and chart registries:

    ```shell
    docker login
    helm registry login
    ```

2. Extract and use the `airgap-push.sh` script to push the `extensions` images
   and `charts` contents to the registry:  

     ```shell
     tar xvf kcm-airgap-<version>.tgz scripts/airgap-push.sh
     ./scripts/airgap-push.sh -r <registry> -c <chart-repo> -a kcm-airgap-<version>.tgz
     ```

3. Extract the `management` bundle tarball and sync the images to the
   target k0s management cluster.  See [Sync the Bundle File](https://docs.k0sproject.io/head/airgap-install/#2a-sync-the-bundle-file-with-the-airgapped-machine-locally)
   for more information.

     > NOTE:
     > Multiple image bundles can be placed in the `/var/lib/k0s/images`
     > directory for k0s to use; you don't need to merge the existing `k0s` airgap bundle
     > into the `kcm-images-<version>.tgz` bundle.

     ```shell
     tar -C /var/lib/k0s -xvf kcm-airgap-<version>.tgz "images/kcm-images-<version>.tgz"
     ```

4. At this point the chart registry includes the k0rdent kcm charts. The kcm controller image is loaded as part of the airgap `management` bundle and does not need to be customized within the Helm chart, but the default chart repository configured via `controller.defaultRegistryURL` should be set to reference the repository:

      ```shell
      helm install kcm oci://<chart-repository>/kcm \
        --version <version> \
        -n kcm-system \
        --create-namespace \
        --set controller.defaultRegistryURL=oci://<chart-repository>
      ```

5. Edit the `Management` object to add the airgap parameters.

     The resulting yaml should look like this:

      ```yaml
      apiVersion: k0rdent.mirantis.com/v1alpha1
      kind: Management
      metadata:
        name: kcm
      spec:
        core:
          capi:
            config:
              airgap: true
          kcm:
            config:
              controller:
                defaultRegistryURL: oci://<registry-url>
                insecureRegistry: true
        providers:
        - config:
            airgap: true
          name: k0smotron
        - config:
            airgap: true
          name: cluster-api-provider-vsphere
        - name: projectsveltos
        release: <release name>
      ```
	 > NOTE:
	 > Use the `insecureRegistry` parameter only if you have a plain HTTP (rather than HTTPS)
	 > registry.


6. k0s components must be downloaded at each node upon creation. To make them available, you can either place the k0s binary and airgap bundle on an internal server so they are available over HTTP, or make them available as a Kubernetes `Deployment` as follows:

    > NOTE:
    > k0s image version is the same that the default defined in the vSphere
    > template. [[[ NOT SURE WHAT THIS SENTENCE MEANS ]]]

  ```yaml
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: k0s-ag-image
    labels:
      app: k0s-ag-image
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: k0s-ag-image
    template:
      metadata:
        labels:
          app: k0s-ag-image
      spec:
        containers:
        - name: k0s-ag-image
        image: k0s-ag-image:v1.31.1-k0s.1
          ports:
          - containerPort: 80
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: k0s-ag-image
  spec:
    ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
    selector:
      app: k0s-ag-image
    type: NodePort
  ```
    [[[ EXPLAIN WHAT IS GOING ON HERE ]]]

#### Creation of the ClusterDeployment

The last step is to make sure that `ClusterDeployment`s have access to the Kubernetes bundle. To do that, you need to add additional information to the `.spec.config` of the `ClusterDeployment` object.

Specifically, you need to specify the custom image registry and chart repository to be used. These are the same URLs to which you pushed the `extensions` bundle and charts earlier. Finally, provide the endpoint where the k0s binary and airgap bundle can be downloaded, as you specified in step 7 of the [installation procedure](#installation)):

```yaml
spec:
 config:
   airgap: true
   k0s:
     downloadURL: "http://<k0s binary endpoint>/k0s"
     bundleURL: "http://<k0s binary endpoint>/k0s-airgap-bundle"
   extensions:
    imageRepository: ${IMAGE_REPOSITORY}
    chartRepository: ${CHART_REPOSITORY}
```
## Managed Cluster Target Platforms

Managed clusters can be hosted on a number of different platforms. At the time of this writing, those platforms include:

- Amazon Web Services
- Microsoft Azure
- OpenStack
- VMware


### AWS

k0rdent is able to deploy managed clusters as both EC2-based Kubernetes clusters and EKS clusters. In both cases, you'll need to create the relevant credentials, and to do that you'll need to configure an IAM user. Follow these steps to make it possible to deploy to AWS:

1. Install k0rdent

    Follow the instructions in [[[ add link ]]] Install k0rdent to create a management cluster with k0rdent running.

1. Install `clusterawsadm`

    k0rdent uses the Cluster API (CAPI) to marshal clouds and infrastructures. For AWS, this means using the components from the Cluster API Provider AWS (CAPA) project. clusterawsadm, a CLI tool created by CAPA project, helps with AWS-specific tasks such as creating IAM roles and policies, as well as credential configuration. To install clusterawsadm on Ubuntu on x86 hardware, execute these commands:

    ```shell
    curl -LO https://github.com/kubernetes-sigs/cluster-api-provider-aws/releases/download/v2.7.1/clusterawsadm-linux-amd64
    sudo install -o root -g root -m 0755 clusterawsadm-linux-amd64 /usr/local/bin/clusterawsadm
    ```

2. Configure AWS IAM

    Next you'll need to create the IAM policies and service account k0rdent will use to take action within the AWS infrastructure. (Note that you only need to do this once.)

    The first step is to crete the IAM CloudFormation stack based on your admin user. Start by specifying the environment variables clusterawsadmin will use as AWS credentials:

    ```shell
    export AWS_REGION=<aws-region>
    export AWS_ACCESS_KEY_ID=<admin-user-access-key>
    export AWS_SECRET_ACCESS_KEY=<admin-user-secret-access-key>
    export AWS_SESSION_TOKEN=<session-token> # Optional. If you are using Multi-Factor Auth.
    ```

3. Create the IAM CloudFormation 

	Now use clusterawsadm to create the IAM CloudFormation stack:

	```shell
	clusterawsadm bootstrap iam create-cloudformation-stack
	```
	
4. Install the AWS CLI
 
	With the stack in place you can create the AWS IAM user. You can do this in the UI, but it's also possible to do it from the command line using the aws CLI tool.  Start by installing it, if you haven't already:

	```shell
	sudo apt install unzip
	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
	unzip awscliv2.zip 
	sudo ./aws/install
	```

	The tool recognizes the environment variables you created earlier, so there's no need to login.

5. Create the IAM user. 

	The actual `user-name` parameter is arbitrary; you can specify it as anything you like:

	```shell
	aws iam create-user --user-name k0rdentQuickstart
	```
	```console
	{
	  "User": {
		  "Path": "/",
		  "UserName": "k0rdentQuickstart",
		  "UserId": "AIDA2XXXXXXXXXXXXXXXX",
		  "Arn": "arn:aws:iam::743175908171:user/k0rdentQuickstart",
		  "CreateDate": "2025-01-18T08:15:27+00:00"
	  }
	}
	```

6. Assign the relevant policies
 
	You'll need to assign the following policies to the user you just created:

	```shell
	control-plane.cluster-api-provider-aws.sigs.k8s.io
	controllers.cluster-api-provider-aws.sigs.k8s.io
	nodes.cluster-api-provider-aws.sigs.k8s.io
	```
	To do that, you'll need the ARNs for each policy.  You can get them with the `list-policies` command, as in:

	```shell
	aws iam list-policies --scope Local
	```
	```console
	{
	  "Policies": [
		  {
			  "PolicyName": "controllers-eks.cluster-api-provider-aws.sigs.k8s.io",
			  "PolicyId": "ANPA22CF4NNF3VUDTMH3N",
			  "Arn": "arn:aws:iam::743175908171:policy/controllers-eks.cluster-api-provider-aws.sigs.k8s.io",
			  "Path": "/",
			  "DefaultVersionId": "v1",
			  "AttachmentCount": 2,
			  "PermissionsBoundaryUsageCount": 0,
			  "IsAttachable": true,
			  "CreateDate": "2025-01-01T18:47:43+00:00",
			  "UpdateDate": "2025-01-01T18:47:43+00:00"
		  },
		  {
			  "PolicyName": "nodes.cluster-api-provider-aws.sigs.k8s.io",
			  "PolicyId": "ANPA22CF4NNF5TAKL44PU",
			  "Arn": "arn:aws:iam::743175908171:policy/nodes.cluster-api-provider-aws.sigs.k8s.io",
			  "Path": "/",
			  "DefaultVersionId": "v1",
			  "AttachmentCount": 3,
			  "PermissionsBoundaryUsageCount": 0,
			  "IsAttachable": true,
			  "CreateDate": "2025-01-01T18:47:44+00:00",
			  "UpdateDate": "2025-01-01T18:47:44+00:00"
		  },
		  {
			  "PolicyName": "controllers.cluster-api-provider-aws.sigs.k8s.io",
			  "PolicyId": "ANPA22CF4NNFVO6OHIQOE",
			  "Arn": "arn:aws:iam::743175908171:policy/controllers.cluster-api-provider-aws.sigs.k8s.io",
			  "Path": "/",
			  "DefaultVersionId": "v1",
			  "AttachmentCount": 3,
			  "PermissionsBoundaryUsageCount": 0,
			  "IsAttachable": true,
			  "CreateDate": "2025-01-01T18:47:43+00:00",
			  "UpdateDate": "2025-01-01T18:47:43+00:00"
		  },
		  {
			  "PolicyName": "control-plane.cluster-api-provider-aws.sigs.k8s.io",
			  "PolicyId": "ANPA22CF4NNFY4FJ3DA2E",
			  "Arn": "arn:aws:iam::743175908171:policy/control-plane.cluster-api-provider-aws.sigs.k8s.io",
			  "Path": "/",
			  "DefaultVersionId": "v1",
			  "AttachmentCount": 2,
			  "PermissionsBoundaryUsageCount": 0,
			  "IsAttachable": true,
			  "CreateDate": "2025-01-01T18:47:43+00:00",
			  "UpdateDate": "2025-01-01T18:47:43+00:00"
		  }
	  ]
	}
	```

	Now you can add the policies using the `attach-user-policy` command and the ARNs you retrieved in the previous step:

	```shell
	aws iam attach-user-policy --user-name k0rdentQuickstart --policy-arn arn:aws:iam::743175908171:policy/controllers-eks.cluster-api-provider-aws.sigs.k8s.io
	aws iam attach-user-policy --user-name k0rdentQuickstart --policy-arn arn:aws:iam::743175908171:policy/controllers.cluster-api-provider-aws.sigs.k8s.io
	aws iam attach-user-policy --user-name k0rdentQuickstart --policy-arn arn:aws:iam::743175908171:policy/control-plane.cluster-api-provider-aws.sigs.k8s.io
	```
	
8. Create an access key and secret

	To access AWS as this new user, you'll need to create an access key:

	```shell
	aws iam create-access-key --user-name k0rdentQuickstart 
	```
	```console
	{
	  "AccessKey": {
		  "UserName": "k0rdentQuickstart",
		  "AccessKeyId": "AKIA2XXXXXXXXXXXXXXX",
		  "Status": "Active",
		  "SecretAccessKey": "nWtQIsXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
		  "CreateDate": "2025-01-18T08:33:35+00:00"
	  }
	}
	```

9. Create the IAM Credentials Secret on the k0rdent Management Cluster

	Create a YAML file called aws-cluster-identity-secret.yaml and add the following text, including the `AccessKeyId` and `SecretAccessKey` you created in the previous step:

	```yaml
	apiVersion: v1
	kind: Secret
	metadata:
	name: aws-cluster-identity-secret
	namespace: kcm-system
	type: Opaque
	stringData:
	AccessKeyID: EXAMPLE_ACCESS_KEY_ID
	SecretAccessKey: EXAMPLE_SECRET_ACCESS_KEY
	```

	Apply the YAML to your cluster, making sure to add it to the namespace where the CAPA provider is running (currently kcm-system) so the controller can read it:

	```shell
	kubectl apply -f aws-cluster-identity-secret.yaml -n kcm-system
	```

11. Create the `AWSClusterStaticIdentity`

	Create the `AWSClusterStaticIdentity` object in a file named aws-cluster-identity.yaml:

	```shell
	kind: AWSClusterStaticIdentity
	metadata:
	name: aws-cluster-identity
	spec:
	secretRef: aws-cluster-identity-secret
	allowedNamespaces:
	  selector:
		matchLabels: {}
	```

	Notice that the `secretRef` references the `Secret` you created in the previous step.

	Apply the YAML to your cluster, again adding it to the `kcm-system` namespace.

	```shell
	kubectl apply -f aws-cluster-identity.yaml  -n kcm-system
	```

12. Create the `Credential`

	Finally, create the kcm `Credential` object, making sure to reference the `AWSClusterStaticIdentity` you just created:

	```yaml
	apiVersion: k0rdent.mirantis.com/v1alpha1
	kind: Credential
	metadata:
	name: aws-cluster-identity-cred
	namespace: kcm-system
	spec:
	description: "Credential Example"
	identityRef:
	  apiVersion: infrastructure.cluster.x-k8s.io/v1beta2
	  kind: AWSClusterStaticIdentity
	  name: aws-cluster-identity
	```
	Apply the YAML to your cluster, again keeping it in the `kcm-system` namespace:

	```shell
	kubectl apply -f aws-cluster-identity-cred.yaml -n kcm-system
	```

13. Deploy a cluster

	Make sure everything is configured properly by creating a `ClusterDeployment`. Start with a YAML file specifying the `ClusterDeployment`, as in:

	```yaml
	apiVersion: k0rdent.mirantis.com/v1alpha1
	kind: ClusterDeployment
	metadata:
	name: my-aws-clusterdeployment1
	namespace: kcm-system
	spec:
	template: aws-standalone-cp-0-0-5
	credential: aws-cluster-identity-cred
	config:
	  region: us-east-2
	  controlPlane:
		instanceType: t3.small
	  worker:
		instanceType: t3.small
	```

	A couple of things to notice:
	- You're giving it an arbitrary name (my-aws-clusterdeployment1)
	- You're referencing the credential you created in the previous step. This enables you to set up a system where users can take advantage of having access to the credentials to the AWS account without actually having those credentials in hand.
	- You need to choose a template to use for the cluster. You can get a list of available templates using:
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

	Apply the YAML to your management cluster:

	```shell
	kubectl apply -f my-aws-clusterdeployment1.yaml
	```
	```console
	clusterdeployment.k0rdent.mirantis.com/my-aws-clusterdeployment1 created
	```
	As before, there will be a delay as the cluster finishes provisioning. Follow the provisioning process with:

	```shell
	kubectl -n kcm-system get clusterdeployment.k0rdent.mirantis.com my-aws-clusterdeployment1 --watch
	```
	```console
	NAME                        READY   STATUS
	my-aws-clusterdeployment1   True    ClusterDeployment is ready
	```

	When the cluster is `Ready`, you can access it via the kubeconfig, as in:

	```shell
	kubectl -n kcm-system get secret my-aws-clusterdeployment1-kubeconfig -o jsonpath='{.data.value}' | base64 -d > my-aws-clusterdeployment1-kubeconfig.kubeconfig
	KUBECONFIG="my-aws-clusterdeployment1-kubeconfig.kubeconfig" kubectl get pods -A
	```

1. Cleanup

	When you've established that it's working properly. you can delete the managed cluster and its AWS objects:

	```shell
	kubectl delete ClusterDeployment my-aws-clusterdeployment1 
	```

### Azure

Standalone clusters can be deployed on Azure instances.

#### Software prerequisites

Before using k0rdent to deploy managed Kubernetes clusters on Azure, make sure you have the following components installed and configured:

1. Install k0rdent

    Follow the instructions in [[[ add link ]]] Install k0rdent to create a management cluster with k0rdent running.

1. The Azure CLI

    The Azure CLI (az) is required to interact with Azure resources. You can install it on Ubuntu as follows:

    ```shell
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

2. Log in to Azure

    Run the `az login` command to authenticate your session with Azure:

    ```shell
    az login
    ```

3. Register resource providers

    In order for k0rdent to deploy and manage clusters, it needs to be able to work with Azure resources such as 
    compute, network, and identity. Make sure the subscription you're using has the following resource providers registered:

    - `Microsoft.Compute`
    - `Microsoft.Network`
    - `Microsoft.ContainerService`
    - `Microsoft.ManagedIdentity`
    - `Microsoft.Authorization`

    To register these providers, run the following commands in the Azure CLI:

    ```shell
    az provider register --namespace Microsoft.Compute
    az provider register --namespace Microsoft.Network
    az provider register --namespace Microsoft.ContainerService
    az provider register --namespace Microsoft.ManagedIdentity
    az provider register --namespace Microsoft.Authorization
    ```

#### Credentials

Creating a managed cluster requires a structure of credentials that link to user identities on the provider system without
exposing the actual username and password to users. You can find more information on k0rdent [[[ LINK TO CREDENTIALS FILE ]]] 
Credentials, but for Azure, this involves creating an `AzureClusterIdentity` and a 
Service Principal (SP) to let CAPZ (Cluster API Azure) communicate with the cloud provider. To Set up credentials, follow these steps:

1. Find Your Subscription ID

    Your Azure must have at least one subscription for you to use it with k0rdent, so if you're working with a new
    account make sure to (create a new subscription with billing information)[] before you start.

    To get the information you need, list all your Azure subscriptions:https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/initial-subscriptions

    ```shell
    az account list -o table
    ```
    ```console
    Name                     SubscriptionId                        TenantId
    -----------------------  -------------------------------------  --------------------------------
    My Azure Subscription    12345678-1234-5678-1234-567812345678  87654321-1234-5678-1234-12345678
    ```

    Make note of the `SubscriptionId` for the subscription you want to use.

2. Create a Service Principal (SP)

    The Service Principal is like a password-protected user that CAPZ will use to manage resources on Azure.
    Create the Service Principal, making sure to replace <subscription-id> with the `SubscriptionId` from step 1.

    ```shell
    az ad sp create-for-rbac --role contributor --scopes="/subscriptions/<subscription-id>"
    ```
    ```console
    {
    "appId": "12345678-7848-4ce6-9be9-a4b3eecca0ff",
    "displayName": "azure-cli-2024-10-24-17-36-47",
    "password": "12~34~I5zKrL5Kem2aXsXUw6tIig0M~3~1234567",
    "tenant": "12345678-959b-481f-b094-eb043a87570a"
    }
    ```
    Note that this information gives you access to your Azure account, so make sure to treat these strings 
    like passwords. Do not share them or check them into a repository.

3. Use the password to create a Secret object

    The `Secret` stores the `clientSecret` (password) from the Service Principal.
    Save the `Secret` YAML in a file called azure-cluster-identity-secret.yaml:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: azure-cluster-identity-secret
      namespace: kcm-system
    stringData:
      clientSecret: <password> # Password retrieved from the Service Principal
    type: Opaque
    ```

    You can then apply the YAML to your cluster:

    ```shell
    kubectl apply -f azure-cluster-identity-secret.yaml
    ```

4. Create the `AzureClusterIdentity` objects

    The `AzureClusterIdentity` object defines the credentials CAPZ uses to manage Azure resources. 
    It references the `Secret` you just created, so make sure that `.spec.clientSecret.name` matches 
    the name of that `Secret`.

    Save the following YAML into a file named azure-cluster-identity.yaml:

    ```yaml
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
    kind: AzureClusterIdentity
    metadata:
      labels:
        clusterctl.cluster.x-k8s.io/move-hierarchy: "true"
      name: azure-cluster-identity
      namespace: kcm-system
    spec:
      allowedNamespaces: {}
      clientID: <appId> # The App ID retrieved from the Service Principal above in Step 2
      clientSecret:
        name: azure-cluster-identity-secret
        namespace: kcm-system
      tenantID: <tenant> # The Tenant ID retrieved from the Service Principal above in Step 2
      type: ServicePrincipal
    ```

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f azure-cluster-identity.yaml
    ```
    ```console
    azureclusteridentity.infrastructure.cluster.x-k8s.io/azure-cluster-identity created
    ```

5. Create the k0rdent `Credential` Object

    Create the YAML for the specification of the `Credential` and save it as azure-cluster-identity-cred.yaml.

    ```YAML
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: Credential
    metadata:
      name: azure-cluster-identity-cred
      namespace: kcm-system
    spec:
      identityRef:
        apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
        kind: AzureClusterIdentity
        name: azure-cluster-identity
        namespace: kcm-system
    ```

    You're referencing the `AzureClusterIdentity` object you just created, so make sure that `.spec.name` matches 
    `.metadata.name` of that object. Also, note that while the overall object's `kind` is `Credential`, the 
    `.spec.identityRef.kind` must be `AzureClusterIdentity` to match that object.

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f azure-cluster-identity-cred.yaml
    ```
    ```console
    credential.k0rdent.mirantis.com/azure-cluster-identity-cred created
    ```

Now you're ready to deploy the cluster.

#### Create your first ClusterDeployment

To test the configuration, deploy a managed cluster by following these steps:

1. Determine the region

    First get a list of available locations/regions:

    ```shell
    az account list-locations -o table
    ```
    ```console
    DisplayName               Name                 RegionalDisplayName
    ------------------------  -------------------  -------------------------------------
    East US                   eastus               (US) East US
    South Central US          southcentralus       (US) South Central US
    West US 2                 westus2              (US) West US 2
    West US 3                 westus3              (US) West US 3
    Australia East            australiaeast        (Asia Pacific) Australia East
    …
    ```

    Make note of the location you want to use, such as `eastus`.

2. Create the cluster specification

    To create a managed cluster, create a `ClusterDeployment` that references the appropriate template
    as well as the location, credentials, and `subscriptionId`.

    You can see the available templates by listing them:

    ```shell
    kubectl get clustertemplate -n kcm-system
    ```

    Create the yaml:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: my-azure-clusterdeployment1
      namespace: kcm-system
    spec:
      template: azure-standalone-cp-0-0-5
      credential: azure-cluster-identity-cred
      config:
        location: "westus" # Select your desired Azure Location (find it via `az account list-locations -o table`)
        subscriptionID: <subscription-id> # Enter the Subscription ID used earlier
        controlPlane:
          vmSize: Standard_A4_v2
        worker:
          vmSize: Standard_A4_v2
    ```

    Apply the YAML to your management cluster:

    ```shell
    kubectl apply -f my-azure-clusterdeployment1.yaml
    ```
    ```console
    clusterdeployment.k0rdent.mirantis.com/my-azure-clusterdeployment1 created
    ```

    Note that although the `ClusterDeployment` object has been created, there will be a delay as actual Azure instances
    are provisioned and added to the cluster. You can follow the provisioning process:

    ```shell
    kubectl -n kcm-system get clusterdeployment.k0rdent.mirantis.com my-azure-clusterdeployment1 --watch
    ```

    If the provisioning process continues for a more than a few minutes, check to make sure k0rdent isn't trying to
    exceed your quotas. If you are near the top of your quotas, requesting an increase can "unstick" the provisioning process.

    After the cluster is `Ready`, you can access it via the kubeconfig:

    ```shell
    kubectl -n kcm-system get secret my-azure-clusterdeployment1-kubeconfig -o jsonpath='{.data.value}' | base64 -d > my-azure-clusterdeployment1-kubeconfig.kubeconfig
    KUBECONFIG="my-azure-clusterdeployment1-kubeconfig.kubeconfig" kubectl get pods -A
    ```

#### Cleanup

To clean up Azure resources, delete the managed cluster by deleting the `ClusterDeployment`:

```shell
kubectl get ClusterDeployments -A
```
```console
NAMESPACE    NAME                          READY   STATUS
kcm-system   my-azure-clusterdeployment1   True    ClusterDeployment is ready
```
```shell
kubectl delete ClusterDeployment my-azure-clusterdeployment1 -n kcm-system
```
```console
clusterdeployment.k0rdent.mirantis.com "my-azure-clusterdeployment1" deleted
```

### OpenStack

Much of the following includes the process of setting up credentials for OpenStack.
To better understand how k0rdent uses credentials, read the
[Credential system](../credential/main.md).

To configure and test k0rdent's ability to create OpenStack managed clusters, follow these steps:

1. Install k0rdent

    Follow the instructions in [install k0rdent](admin-install.md) to create a management cluster with k0rdent running.

2. OpenStack CLI (optional)

    If you plan to access OpenStack directly, go ahead and 
    [install the OpenStack CLI](https://docs.openstack.org/newton/user-guide/common/cli-install-openstack-command-line-clients.html).

3. Configure the OpenStack Application Credential

    This credential should include:

    - `OS_AUTH_URL`
    - `OS_APPLICATION_CREDENTIAL_ID`
    - `OS_APPLICATION_CREDENTIAL_SECRET`
    - `OS_REGION_NAME`
    - `OS_INTERFACE`
    - `OS_IDENTITY_API_VERSION` (commonly `3`)
    - `OS_AUTH_TYPE` (e.g., `v3applicationcredential`)

    While it's possible to use a username and password instead of the Application Credential--adjust your YAML accordingly--an 
    Application Credential is strongly recommended because it limits scope and improves security over a raw username/password approach.

4. Create the OpenStack Credentials Secret

    Create a Kubernetes Secret containing the `clouds.yaml` that defines your OpenStack environment, substituting real values
    where appropriate. Save this as `openstack-cloud-config.yaml`:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: openstack-cloud-config
      namespace: kcm-system
    stringData:
      clouds.yaml: |
        clouds:
          openstack:
            auth:
              auth_url: <OS_AUTH_URL>
              application_credential_id: <OS_APPLICATION_CREDENTIAL_ID>
              application_credential_secret: <OS_APPLICATION_CREDENTIAL_SECRET>
            region_name: <OS_REGION_NAME>
            interface: <OS_INTERFACE>
            identity_api_version: <OS_IDENTITY_API_VERSION>
            auth_type: <OS_AUTH_TYPE>
    ```

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f openstack-cloud-config.yaml
    ```

5. Create the k0rdent Credential Object

    Next, define a Credential that references the Secret from the previous step.
    Save this as `openstack-cluster-identity-cred.yaml`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: Credential
    metadata:
      name: openstack-cluster-identity-cred
      namespace: kcm-system
    spec:
      description: "OpenStack credentials"
      identityRef:
        apiVersion: v1
        kind: Secret
        name: openstack-cloud-config
        namespace: kcm-system
    ```

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f openstack-cluster-identity-cred.yaml
    ```

    Note that `.spec.identityRef.name` must match the `Secret` you created in the previous step, and 
    `.spec.identityRef.namespace` must be the same as the `Secret`’s namespace (`kcm-system`).

6. Create Your First Managed Cluster

    To test the configuration, create YAML with the specification of your Managed Cluster and save it as
    `my-openstack-cluster-deployment.yaml`.  Note that you can see the available templates by listing them:

    ```shell
    kubectl get clustertemplate -n kcm-system
    ```

    Here is an example:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: ClusterDeployment
    metadata:
      name: my-openstack-cluster-deployment
      namespace: kcm-system
    spec:
      template: openstack-standalone-cp-0-0-1
      credential: openstack-cluster-identity-cred
      config:
        controlPlaneNumber: 1
        workersNumber: 1
        controlPlane:
          flavor: m1.medium
          image:
            filter:
              name: ubuntu-22.04-x86_64
        worker:
          flavor: m1.medium
          image:
            filter:
              name: ubuntu-22.04-x86_64
        authURL: <OS_AUTH_URL>
    ```
    You can adjust `flavor`, `image` name, and `authURL` to match your OpenStack environment. For more information about the configuration options, see the [OpenStack Template Parameters](../clustertemplates/openstack/template-parameters.md).

    Apply the YAML to your management cluster:

    ```shell
    kubectl apply -f my-openstack-cluster-deployment.yaml
    ```

    There will be a delay as the cluster finishes provisioning. You can follow the
    provisioning process:

    ```shell
    kubectl -n kcm-system get clusterdeployment.k0rdent.mirantis.com my-openstack-cluster-deployment --watch
    ```

    After the cluster is `Ready`, you can access it via the kubeconfig, just like any other Kubernetes cluster:

    ```shell
    kubectl -n kcm-system get secret my-openstack-cluster-deployment-kubeconfig -o jsonpath='{.data.value>' | base64 -d > my-openstack-cluster-deployment-kubeconfig.kubeconfig
    KUBECONFIG="my-openstack-cluster-deployment-kubeconfig.kubeconfig" kubectl get pods -A
    ```

7. Cleanup

    To clean up OpenStack resources, delete the managed cluster by deleting the `ClusterDeployment`:

    ```shell
    kubectl get ClusterDeployments -A
    ```
    ```console
    NAMESPACE    NAME                          READY   STATUS
    kcm-system   my-openstack-cluster-deployment   True    ClusterDeployment is ready
    ```
    ```shell
    kubectl delete ClusterDeployment my-openstack-cluster-deployment -n kcm-system
    ```
    ```console
    clusterdeployment.k0rdent.mirantis.com "my-openstack-cluster-deployment" deleted
    ```


### VSphere

Much of the following includes the process of setting up credentials for vSphere.
To better understand how k0rdent uses credentials, read the
[Credential System](../credential/main.md).

#### Prerequisites

Make sure you have the following installed:

1. A k0rdent management cluster

    Follow the instructions in [install k0rdent](admin-install.md) to create a management cluster with k0rdent running.

1. The `kubectl` CLI, installed locally.
2. A vSphere instance version `6.7.0` or higher.

4. [Image template](#image-template).
5. [vSphere network](#vsphere-network) with DHCP enabled.

3. vSphere account with appropriate privileges

    To function properly, the user assigned to the vSphere Provider should be able
    to manipulate vSphere resources. The user should have the following 
    required privileges:

    - `Virtual machine`: Full permissions are required
    - `Network`: `Assign network` is sufficient
    - `Datastore`: The user should be able to manipulate virtual machine files and metadata

    In addition to that, specific CSI driver permissions are required. See
    [the official doc](https://docs.vmware.com/en/VMware-vSphere-Container-Storage-Plug-in/2.0/vmware-vsphere-csp-getting-started/GUID-0AB6E692-AA47-4B6A-8CEA-38B754E16567.html)
    for more information on CSI-specific permissions.

4. Image template

    You can use pre-built image templates from the
    [CAPV project](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/main/README.md#kubernetes-versions-with-published-ovas)
    or build your own.

    When building your own image, make sure that VMware tools and cloud-init are
    installed and properly configured.

    You can follow the [official open-vm-tools guide](https://docs.vmware.com/en/VMware-Tools/11.0.0/com.vmware.vsphere.vmwaretools.doc/GUID-C48E1F14-240D-4DD1-8D4C-25B6EBE4BB0F.html)
    on how to correctly install VMware tools.

    When setting up cloud-init, you can refer to the [official docs](https://cloudinit.readthedocs.io/en/latest/index.html)
    and specifically the [VMware datasource docs](https://cloudinit.readthedocs.io/en/latest/reference/datasources/vmware.html)
    for extended information regarding cloud-init on vSphere.

5. vSphere network

    When creating a network, make sure that it has the DHCP service.

    Also, ensure that part of your network is out of the DHCP range (for example, the network
    `172.16.0.0/24` should have a DHCP range of `172.16.0.100-172.16.0.254` only) so that LoadBalancer services will not create any IP conflicts in the
    network.



#### Create credentials

To enable k0rdent to access vSphere resources, create the appropriate credentials objects:

1. Create a `Secret` Object with the username and password

    The `Secret` stores the username and password for your vSphere instance. Save the `Secret` YAML in a file named `vsphere-cluster-identity-secret.yaml`:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: vsphere-cluster-identity-secret
      namespace: kcm-system
    stringData:
      username: <user>
      password: <password>
    type: Opaque
    ```

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f vsphere-cluster-identity-secret.yaml
    ```

2. Create the `VSphereClusterIdentity` Object

    The `VSphereClusterIdentity` object defines the credentials CAPV will use to manage vSphere resources.

    Save the `VSphereClusterIdentity` YAML into a file named `vsphere-cluster-identity.yaml`:

    ```yaml
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
    kind: VSphereClusterIdentity
    metadata:
      name: vsphere-cluster-identity
    spec:
      secretName: vsphere-cluster-identity-secret
      allowedNamespaces:
        selector:
          matchLabels: {}
    ```

    The `VSpehereClusterIdentity` object references the `Secret` you created in the previous step, so `.spec.secretName` 
    needs to match the `.metadata.name` for the `Secret`.

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f vsphere-cluster-identity.yaml
    ```

3. Create the `Credential` Object

    Create a YAML with the specification of our credential and save it as
    `vsphere-cluster-identity-cred.yaml`

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1alpha1
    kind: Credential
    metadata:
      name: vsphere-cluster-identity-cred
      namespace: kcm-system
    spec:
      identityRef:
        apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
        kind: VSphereClusterIdentity
        name: vsphere-cluster-identity
    ```
    Again, `.spec.identityRef.name` must match the `.metadata.name` of the `VSphereClusterIdentity` object you just created.

    Apply the YAML to your cluster:

    ```shell
    kubectl apply -f vsphere-cluster-identity-cred.yaml
    ```

#### Create your first Cluster Deployment

Test the configuration by deploying a cluster. Create a YAML document with the specification of 
your Cluster Deployment and save it as `my-vsphere-clusterdeployment1.yaml`.

You can get a list of available templates:

```shell
kubectl get clustertemplate -A
```

Here is an example of a `ClusterDeployment` YAML file. Make sure the replace the placeholders with your
specific information:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: ClusterDeployment
metadata:
  name: my-vsphere-clusterdeployment1
  namespace: kcm-system
spec:
  template: vsphere-standalone-cp-0-0-5
  credential: vsphere-cluster-identity-cred
  config:
    vsphere:
      server: <VSPHERE_SERVER>
      thumbprint: <VSPHERE_THUMBPRINT>
      datacenter: <VSPHERE_DATACENTER>
      datastore: <VSPHERE_DATASTORE>
      resourcePool: <VSPHERE_RESOURCEPOOL>
      folder: <VSPHERE_FOLDER>
    controlPlaneEndpointIP: <VSPHERE_CONTROL_PLANE_ENDPOINT>
    controlPlane:
      ssh:
        user: ubuntu
        publicKey: <VSPHERE_SSH_KEY>
      vmTemplate: <VSPHERE_VM_TEMPLATE>
      network: <VSPHERE_NETWORK>
    worker:
      ssh:
        user: ubuntu
        publicKey: <VSPHERE_SSH_KEY>
      vmTemplate: <VSPHERE_VM_TEMPLATE>
      network: <VSPHERE_NETWORK>
```

For more information about the available configuration options, see the [vSphere Template Parameters](../clustertemplates/vsphere/template-parameters.md).

Apply the YAML to your management cluster:

```shell
kubectl apply -f my-vsphere-clusterdeployment1.yaml
```

There will be a delay as the cluster finishes provisioning. Follow the
provisioning process with the following command:

```shell
kubectl -n kcm-system get clusterdeployment.k0rdent.mirantis.com my-vsphere-clusterdeployment1 --watch
```

After the cluster is `Ready`, you can access it via the kubeconfig, like this:

```shell
kubectl -n kcm-system get secret my-vsphere-clusterdeployment1-kubeconfig -o jsonpath='{.data.value}' | base64 -d > my-vsphere-clusterdeployment1-kubeconfig.kubeconfig
KUBECONFIG="my-vsphere-clusterdeployment1-kubeconfig.kubeconfig" kubectl get pods -A
```

To delete provisioned cluster and free consumed vSphere resources run:

```shell
kubectl -n kcm-system delete cluster my-vsphere-clusterdeployment1
```
## Finding Releases

Releases are tagged in the GitHub repository and can be found [here](https://github.com/k0rdent/kcm/tags).

## Extended Management Configuration

k0rdent is deployed with the following default configuration, which may vary
depending on the release version:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: Management
metadata:
  name: kcm
spec:
  providers:
  - name: k0smotron
  - name: cluster-api-provider-aws
  - name: cluster-api-provider-azure
  - name: cluster-api-provider-vsphere
  - name: projectsveltos
release: kcm-0-0-7
```
To see what is included in a specific release, look at the `release.yaml` file in the tagged release.
For example, here is the [v0.0.7 release.yaml](https://github.com/k0rdent/kcm/releases/download/v0.0.7/release.yaml).

There are two options to override the default management configuration of k0rdent:

1. Update the `Management` object after the k0rdent installation using `kubectl`:

    `kubectl --kubeconfig <path-to-management-kubeconfig> edit management`

2. Deploy k0rdent skipping the default `Management` object creation and provide your
   own `Management` configuration:

	- Create `management.yaml` file and configure core components and providers.
	- Specify `--create-management=false` controller argument and install k0rdent:
	  If installing using `helm` add the following parameter to the `helm
	  install` command:

		`--set="controller.createManagement=false"`

	- Create the `kcm` `Management` object after the k0rdent installation:

    ```bash
    kubectl --kubeconfig <path-to-management-kubeconfig> create -f management.yaml
    ```
    
### Cleanup

Before you delete the `Management` object, make sure that you've deleted all `ClusterDeployment`s,
as you won't be able to delete them afterwards.

1. Remove the Management object:

  ```bash
	kubectl delete management kcm
  ```

2. Remove the `kcm` Helm release:

  ```bash
	helm uninstall kcm -n kcm-system
  ```

3. Remove the `kcm-system` namespace:

  ```bash
	kubectl delete ns kcm-system
  ```
