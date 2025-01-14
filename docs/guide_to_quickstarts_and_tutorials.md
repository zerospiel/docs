# QuickStart

## Installing and setting up k0rdent

The following is a recipe for installing k0rdent and experiencing how it works. Emphasis is on quick and simple, not production-ready. If you are looking for best-practices used in creating a production-grade installation of k0rdent, please see the [Administrator Guide]().

Getting k0rdent ready begins with a few steps:

1. Setting up laptop pre-requisites
2. Creating a management cluster &mdash; the QuickStart using Kubernetes-in-Docker (KinD) for this
3. Installing k0rdent and necessary providers on the management cluster

k0rdent uses ClusterAPI (CAPI) to manage infrastructure and clusters. For our QuickStart and the tutorials that follow, we're providing recipe steps and make-based automation for setting up k0rdent to deploy and manage [k0s Kubernetes](https://k0sproject.io) clusters on AWS, Azure, and vSphere virtual machines.

Once initial steps are complete, we'll show how to add credentials you'll be able to deploy a non-production hosted cluster on AWS EC2. We will be adding QuickStart routes for Amazon EKS (that is, the Kubernetes target cluster is Amazon Elastic Kubernetes Service-flavored), Azure hosted (i.e., k0s Kubernetes on Azure VMs), Azure AKS (i.e., Azure Kubernetes Service-flavored target cluster), OpenStack hosted, VMware hosted, and other variants, very soon.

## Tutorials

Once you've completed setup and deployed a single cluster on AWS, additional tutorials are provided to walk you through important phases of using k0rdent, using AWS as target infrastructure. These include:

* Upgrade a Single Standalone Cluster (i.e., the AWS hosted cluster you just deployed)
* Install a ServiceTemplate into a single cluster (adding services to the cluster to create a more complete platform)
* Install a ServiceTemplate into multiple clusters (demonstrating k0rdent's ability to help you instantiate platforms at scale)

The above QuickStart and Tutorials focus on serving the needs of Platform Architects and Platform Engineering teams. They'll give you a feel of using k0rdent from first principles to define and instantiate Internal Development Platforms (IDPs) on clouds and infrastructure(s). 

## k0rdent workflows

Large organizations also need means of working at scale. For example, Platform Architects may use k0rdent to define platform templates, whereas Platform Leads and others will consume these templates to create and lifecycle manage platforms for their own teams and stakeholders.

Two tutorials demonstrate aspects of how k0rdent enables sharing for Platform Architects:

* Approve a ClusterTemplate & InfraCredentials for a separate namespace (i.e., how to safely share a cluster template with a Platform Lead, while constraining how it can be used)
* Approve a ServiceTemplate for a separate namespace (i.e., safely share a service template with others, while constraining how it can be used)

Finally, two companion tutorials demonstrate how shared templates can be consumed (e.g., by Platform Leads) within appropriate guardrails.

* Use an approved ClusterTemplate in a separate namespace
* Use an approved ServiceTemplate in a separate namespace

Ready? [Let's discover how to use k0rdent!]()

