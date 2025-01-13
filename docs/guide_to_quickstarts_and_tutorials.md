# QuickStart

## Installing and setting up k0rdent

The following is a recipe for installing k0rdent and experiencing how it works. Emphasis is on quick and simple, not production-ready. If you are looking for best-practices used in creating a production-grade installation of k0rdent, please see the [Administrator Guide]().

Getting k0rdent ready for use begins with a few steps:

1. Setting up a management cluster - k0rdent runs on CNCF-certified Kubernetes 1.28+.
2. Setting up laptop pre-requisites.
3. Installing k0rdent main components on the management cluster.

## Choosing a Kubernetes distro and target infrastructure

Next, you'll need to select a Kubernetes distro, and choose a target cloud or infrastructure on which you wish to deploy and manage clusters. For more about this, please see the [Administrator Guide]().

For our QuickStart and the tutorials that follow, however, we're eliminating the need to make these choices -- at least to start. Instead, we're providing recipe steps and make-based automation for setting up k0rdent to deploy and manage basic [k0s Kubernetes](https://k0sproject.io) clusters on AWS EC2 virtual machines. Here are the steps you'll follow: 

1. Install ClusterAPI providers for distro and target cloud or infrastructure.
2. Securely provide credentials and other required configuration info.

Once these steps are complete, you'll be able to deploy a non-production hosted cluster on AWS EC2. We will be adding QuickStart routes for Amazon EKS (that is, the Kubernetes target cluster is Amazon Elastic Kubernetes Service-flavored), Azure hosted (i.e., k0s Kubernetes on Azure VMs), Azure AKS (i.e., Azure Kubernetes Service-flavored target cluster), OpenStack hosted, VMware hosted, and other variants, very soon.

## Tutorials

Once you've completed setup (for AWS hosted cluster management), we've provided a series of simple tutorials to walk you through important phases of using k0rdent. These include:

* Upgrade a Single Standalone Cluster (i.e., the AWS hosted cluster you just deployed)
* Install a ServiceTemplate into a single cluster (adding services to the cluster to create a more complete platform)
* Install a ServiceTemplate into multiple clusters (demonstrating k0rdent's ability to help you instantiate platforms at scale)

The above QuickStart and Tutorials are aimed at Platform Architects and Platform Engineering teams. They'll give you a feel of using k0rdent from first principles to define and instantiate Internal Development Platforms (IDPs) and workload-hosting platforms on your infrastructure(s). But k0rdent also serves Platform Leads and others who will consume the templates Platform Architects provide, and use them to create and lifecycle manage platforms for their own teams and stakeholders. We have two additional tutorials demonstrating aspects of how k0rdent enables this process:

* Approve a ClusterTemplate & InfraCredentials for a separate namespace (i.e., how to safely share a cluster template with a Platform Lead, while constraining how it can be used)
* Use an approved ClusterTemplate in a separate namespace (i.e., how a Platform Lead can use a cluster template within appropriate guardrails)

[Let's discover how to use k0rdent!]()

