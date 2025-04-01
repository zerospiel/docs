# Welcome to Mirantis k0rdent Enterprise

## Introduction

Mirantis k0rdent Enterprise is a Kubernetes-native platform engineering solution that provides open source composability to simplify the creation of use case-specific developer platforms at scale. It delivers the open source k0rdent Distributed Container Management Environment (DCME) via a secure software supply chain, with components fully validated by Mirantis and enhanced with additional [Enterprise-class features](addons/index.md) and [24/7 Enterprise or Fully-Managed Support](support/index.md).

Mirantis k0rdent Enterprise (and k0rdent) are based on the premise that:

* Kubernetes and its ecosystem are mature and inherently stable.
* Large scale adoption of Kubernetes means that {{{ docsVersionInfo.k0rdentName }}} can run anywhere.
* Community standards and open source projects ensure support and reduce adoption risk.

The goal of {{{ docsVersionInfo.k0rdentName }}} is to provide platform engineers with means
to deliver a distributed container management environment (DCME) and enable them to
compose unique internal developer platforms (IDP) to support a diverse range
of complex modern application workloads.

Another way to think of {{{ docsVersionInfo.k0rdentName }}} is as a "super control plane" designed to ensure the
consistent provisioning and lifecycle management of kubernetes clusters and the
services that make them useful.

In short:
Kubernetes clusters at scale, managed centrally, template driven, based on open
community driven standards, enabling Golden Paths ... {{{ docsVersionInfo.k0rdentName }}} aspires to do all of that.

Whether you want to manage Kubernetes clusters on-premises, in the cloud,
or a combination of both, {{{ docsVersionInfo.k0rdentName }}} provides a consistent way to do it. With
full life-cycle management, including provisioning, configuration, and
maintenance, {{{ docsVersionInfo.k0rdentName }}} is designed to be a repeatable and secure way to
manage your Kubernetes clusters in a central location.

## Structure and History

As noted above, Mirantis k0rdent Enterprise delivers open source k0rdent (with additional features) for Enterprise use, via a tightly-controlled software supply chain and validation process. k0rdent comprises:

* **k0rdent**: the overall project

    * **k0rdent Cluster Manager (KCM)**

        Deployment and life-cycle management of Kubernetes clusters, including
        configuration, updates, and other CRUD operations.

    * **k0rdent State Manager (KSM)**

        Installation and life-cycle management of [beach-head services](appendix/glossary.md#beach-head-services),
        policy, Kubernetes API configurations, and more.

          * This is currently rolled into kcm, but may be split out in the future
          * ksm leverages [Project Sveltos](https://github.com/projectsveltos/sveltos)
            for an increasing amount of functionality

    * **k0rdent Observability and FinOps (KOF)**

        Cluster and beach-head services monitoring, events and log management.

There are a few historical names that may show up in the code and in older docs, including:

* **Project 2A**: the original codename of k0rdent, 2A references the hexadecimal 0x2A, 
  or 42, which encompasses our hopes for the project.
* **HMC or hmc**: the original repository name for k0rdent and KCM
  development
* **motel**: the original repository and codename for KOF

## Quick Start

See the [{{{ docsVersionInfo.k0rdentName }}} Quick Start Guide](quickstarts/index.md) to get started with a small deployment of Mirantis k0rdent Enterprise.

## Supported Providers

{{{ docsVersionInfo.k0rdentName }}} leverages the Cluster API provider ecosystem; the following
providers have had `ProviderTemplates` created and validated, and more are
in the works. 

* [AWS](admin/installation/prepare-mgmt-cluster/aws.md)
* [Azure](admin/installation/prepare-mgmt-cluster/azure.md)
* [vSphere](admin/installation/prepare-mgmt-cluster/vmware.md)
* [OpenStack](admin/installation/prepare-mgmt-cluster/openstack.md)
* [GCP](admin/installation/prepare-mgmt-cluster/gcp.md)

{{{ docsVersionInfo.k0rdentName }}} also includes a way to add custom providers, so it's possible to integrate it with other hyperscalers, 
or even into an existing local infrastructure.

## Development Documentation

Documentation related to the development process and developer-specific notes is
located in the [main k0rdent open source repository](https://github.com/k0rdent/kcm/blob/main/docs/dev.md).

## Where to go from here

This documentation covers all aspects of administering, using, and contributing to {{{ docsVersionInfo.k0rdentName }}}, documentation for Mirantis k0rdent Enterprise AddOns, plus where to get support and services. It includes:

- [{{{ docsVersionInfo.k0rdentName }}} concepts](concepts/index.md)
- [QuickStarts](quickstarts/index.md)
- [Administrator Guide](admin/index.md)
- [User Guide](user/index.md)
- [{{{ docsVersionInfo.k0rdentName }}} Templates Reference](reference/template/index.md)
- [Troubleshooting](troubleshooting/index.md)
- [Glossary](appendix/glossary.md)
- [Appendix](appendix/index.md)

... plus:

- [Addons](addons/index.md) - Composable components and solutions, validated by Mirantis, ready for use with Mirantis k0rdent Enterprise
- [Services, Support, and Contact](support/index.md) - Enterprise support for Mirantis k0rdent Enterprise
