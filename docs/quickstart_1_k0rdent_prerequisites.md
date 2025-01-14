# QuickStart 1 - Install k0rdent prerequisites

The Quickstart installation has a couple of dependencies, so let’s get them installed before we start. 

## Install make

We’ll be using Makefiles to execute a lot of these instructions, so make sure that you have make installed:

```shell
sudo apt-get install make
```

## Install Docker

Our k0rdent management cluster for this QuickStart will run on Kubernetes-in-Docker (KinD), so let’s go ahead and get Docker installed if you don’t have it already. You can find the full instructions [here](https://docs.docker.com/engine/install/ubuntu/), but here’s the short version:

### First uninstall older, potentially conflicting versions:

```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### Use the Docker convenience script to deploy Docker:

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Install kubectl

The script will need the Kubernetes command line client so let’s get that installed next. You can find the full instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/), but assuming you’re on an x86 machine, the basic instructions are:

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## Install clusterawsadm

To simplify dealing with AWS, you’ll need the clusterawsadm tool. You can find all of the releases here, but assuming you’re working with Ubuntu on x86 hardware, it would be:

```shell
curl -LO https://github.com/kubernetes-sigs/cluster-api-provider-aws/releases/download/v2.7.1/clusterawsadm-linux-amd64
sudo install -o root -g root -m 0755 clusterawsadm-linux-amd64 /usr/local/bin/clusterawsadm
```

## Install jq

You’ll need the jq utility to monitor the installation, so let’s install that now:

```shell
sudo apt install jq
```

## Clone the Getting Started repo
All of the scripts you’ll need to get started are in the [Getting Started repo](ttps://github.com/Mirantis/2a-demos) at https://github.com/Mirantis/2a-demos so let’s go ahead and clone that repo:

```shell
git clone https://github.com/Mirantis/2a-demos
cd 2a-demos
```

Now we’re ready to get started.

# Create the management cluster

Now we’re ready to create the k0rdent management cluster so we have somewhere to install the k0rdent components. Fortunately, the makefile takes care of that for you. Because we’re going to need to access Docker as root, let’s go ahead and change over to root:

```shell
sudo su
```

Now go ahead and bootstrap the management cluster:

```shell
make bootstrap-kind-cluster
```

If you want to specify a particular name for this cluster, you can specify the `KIND_CLUSTER_NAME` environment variable, as in:

```shell
export KIND_CLUSTER_NAME=k0rdentdemo
sudo make bootstrap-kind-cluster
```

This command also sets up your `KUBECONFIG` and makes sure kubectl is pointing to the proper context, as you can see from the output:

```shell
No kind clusters found.
Creating cluster "hmc-management-local" ...
 ✓ Ensuring node image (kindest/node:v1.31.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-hmc-management-local"
You can now use your cluster with:
kubectl cluster-info --context kind-hmc-management-local
Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
Switched to context "kind-hmc-management-local".
```

You can see that everything is in place:

```shell
kubectl get pods --all-namespaces
NAMESPACE            NAME                                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-7c65d6cfc9-c9vg6                                     1/1     Running   0          107s
kube-system          coredns-7c65d6cfc9-q7tds                                     1/1     Running   0          107s
kube-system          etcd-hmc-management-local-control-plane                      1/1     Running   0          113s
kube-system          kindnet-m98qq                                                1/1     Running   0          107s
kube-system          kube-apiserver-hmc-management-local-control-plane            1/1     Running   0          113s
kube-system          kube-controller-manager-hmc-management-local-control-plane   1/1     Running   0          113s
kube-system          kube-proxy-vh4mw                                             1/1     Running   0          107s
kube-system          kube-scheduler-hmc-management-local-control-plane            1/1     Running   0          114s
local-path-storage   local-path-provisioner-57c5987fd4-4jdgf                      1/1     Running   0          107s
```

# Install k0rdent into the management cluster

You’re finally ready to install k0rdent!  Fortunately, that’s easy too:

```shell
make deploy-2a
```

The actual deployment can take 5 or 10 minutes, so you’ll want to monitor the installation:

```shell
PATH=$PATH:./bin kubectl get management hmc -o go-template='{{range $key, $value := .status.components}}{{$key}}: {{if $value.success}}{{$value.success}}{{else}}{{$value.error}}{{end}}{{"\n"}}{{end}}'
```

When k0rdent installation has succeeded, the output will look like this:


```shell
capi: true
cluster-api-provider-aws: true
cluster-api-provider-azure: true
cluster-api-provider-vsphere: true
hmc: true
k0smotron: true
projectsveltos: true
```

# Install the Getting Started Helm Repo into the management cluster

Helm charts for deploying the various Getting Started functions are in their own OCI repo, so let’s get that set up on the management cluster:

```shell
make setup-helmrepo
```

As you can see, this makefile deploys a local OCI Helm registry and adds a HelmRepository resource to the cluster so we can use it to deploy the various objects such as child clusters and templates:

```shell
deployment.apps/helm-registry created
persistentvolumeclaim/helm-registry-storage created
service/helm-registry created
helmrepository.source.toolkit.fluxcd.io/2a-demos created
Now add the Helm charts with custom Cluster and Service Templates:
make push-helm-charts
```

Now all the pre-requisites are in place, we can move on to setting up to [deploy our managed cluster on AWS](quickstart_2_aws_infra_setup.md).

