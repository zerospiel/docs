# Guide to QuickStarts and Tutorials

The following is a recipe for quickly installing k0rdent on a small laptop Kubernetes cluster and experiencing how it works. Emphasis is on quick and simple, not production-ready. Setting up k0rdent for production is detailed in the [Administrator Guide]().

**Limitations and constraints** &mdash; our Quickstart and subsequent Tutorials assume:

* You have access to a Linux laptop or VM running recent Ubuntu or another Debian distribution.
* You have administrative-level access to an Amazon Web Services account &mdash; Our QuickStart setup will use Amazon Web Services as target infrastructure and create [k0s Kubernetes](https://k0sproject.io) clusters on AWS EC2 virtual machines, and subsequent Tutorials will re-use this setup.

**Coming soon** &mdash; QuickStarts and Tutorials for other Kubernetes distros, clouds, and environments k0rdent supports, including:

* **AWS hosted** &mdash; k0s Kubernetes on AWS virtual machines (this QuickStart)
* **AWS EKS managed** &mdash; Amazon Elastic Kubernetes Service 
* **Azure hosted** &mdash; k0s Kubernetes on Azure virtual machines
* **Azure AKS managed** &mdash; Azure Kubernetes Service
* **vSphere hosted** &mdash; k0s Kubernetes on vSphere virtual machines
* **OpenStack hosted** &mdash; k0s Kubernetes on OpenStack virtual machines

## QuickStart

Our QuickStart has three parts:

* [QuickStart 1: k0rdent prerequisites](quickstart_1_k0rdent_prerequisites.md) &mdash; Here, we install pre-requisite applications, create a small laptop Kubernetes management cluster with KinD, clone the Getting Started repository and install it in the management cluster, and install k0rdent itself.
* [QuickStart 2: AWS infrastructure setup](quickstart_2_aws_infra_setup.md) &mdash; Here, we set up the AWS roles, policies, users, and credentials k0rdent and and CAPA (ClusterAPI for AWS) need.
* [QuickStart 3: Deploy a managed cluster on AWS](quickstart_3_deploy_managed_cluster_aws.md) &mdash; Here, we deploy your first clusters on AWS. We'll actually be deploying two small clusters to start, so that in the tutorials, below, we can demonstrate how k0rdent can perform multi-cluster operations. In fact, k0rdent can, in principle, enable control of dozens, hundreds, or thousands of clusters, distributed across multiple clouds and infrastructures.

## Tutorials

Once you've completed the QuickStart and deployed your first clusters on AWS, we re-use the same setup in _additional Tutorials_ that walk you through important workflow steps for using k0rdent. These include:

* [Tutorial 1 - Upgrade a Single Standalone Cluster](tutorial_1_upgrade_single_standalone_cluster.md) &mdash; Apply a rolling upgrade to one of your newly-deployed clusters
* [Tutorial 2 - Install a ServiceTemplate into a single standalone cluster](tutorial_2_install_service_template_into_single_standalone_cluster.md) &mdash; Add services to a cluster to create a more complete platform
* [Tutorial 3 - Install a ServiceTemplate into multiple clusters](tutorial_3_install_service_template_into_multiple_clusters.md) &mdash; Leverage k0rdent to instantiate standardized platforms at scale

Initial Tutorials focus on serving the needs of Platform Architects and Platform Engineering teams. They'll give you a feel of using k0rdent from first principles to define and instantiate Internal Development Platforms (IDPs) on clouds and infrastructure(s). 

## k0rdent workflows

Large organizations also need means of working safely at scale &mdash; sharing innovative methods along 'golden paths,' enabling far-flung operations, keeping things simple and robust for innovation consumers (i.e., hiding complexity), and ensuring compliance, security, and standards. k0rdent is being engineered to enable at-scale operations much simpler. For example, Platform Architects can use k0rdent to define platform templates and approve these for use by others with conditional safeguards. Platform Leads and other authorized users can consume these templates (within constraints established by the Platform Architects) to create and lifecycle manage platforms for their own teams and stakeholders.

Two tutorials demonstrate aspects of how k0rdent enables sharing for Platform Architects:

* Approve a ClusterTemplate & InfraCredentials for a separate namespace (i.e., safely share a cluster template _and credentials enabling its use_ without making the template modifiable or the relevant credentials readable)
* Approve a ServiceTemplate for a separate namespace (i.e., safely share a service template with others, while constraining how it can be used)

Finally, two companion tutorials demonstrate how shared templates can be consumed (e.g., by Platform Leads) within appropriate guardrails.

* Use an approved ClusterTemplate in a separate namespace (e.g., to instantiate a cluster for your team)
* Use an approved ServiceTemplate in a separate namespace (e.g., to install services on such a cluster in a constrained way)

Ready? [Let's discover how to use k0rdent!]()

