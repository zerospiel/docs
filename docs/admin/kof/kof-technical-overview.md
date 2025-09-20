
KOF Technical Overview

# Introduction

This document provides a technical overview of the KOF observability platform, detailing its core components and common usage scenarios.

  

KOF is a complete observability and cost management solution that provides:

-   Multi-cluster monitoring and observability
    
-   Financial operations (FinOps) for cost tracking and optimization
    
-   Automated deployment and management of observability infrastructure (via k0rdent)
    
-   Centralized data visualization and querying of metrics and logs
    
-   Support for Data Sovereignty
    

  

Key Features

-   Automation: Operator-driven deployment and configuration. Automatic label-based instrumentation.
    
-   Scalability: Distributed storage and query federation
    
-   Open-Source and Standards-based Technology
    
-   Cost Management (FinOps): OpenCost integration for cloud resource cost tracking
    
-   Multi-tenancy: Support for multiple clusters and namespaces
    

# Core Technologies

The KOF platform leverages a suite of well-tested open-source technologies to provide a complete observability solution.

## Component Roles

[[[ layers ]]]


## Summary

Technology

Capability / Role

[k0rdent](https://docs.k0rdent.io/next/)

Distributed Cluster and Service Orchestration

[OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/)

Fabric for the collection and distribution of traces, logs, and metrics

[OpenCost](https://opencost.io/docs/faq/)

Unified cost visibility across k0rdent Clusters

[Promxy](https://github.com/jacksontj/promxy)

Visibility across regional clusters; Interface for metrics queries via PromQL; Interface for recording and alert rule definition; Alert routing and notifications

[VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics)

Efficient, scalable and resilient storage for OpenTelemetry metrics

[VictoriaLogs](https://github.com/VictoriaMetrics/VictoriaLogs)

Efficient, scalable and resilient storage for OpenTelemetry logs

[Jaeger](https://www.jaegertracing.io/)

Destination for OpenTelemetry traces; dedicated UI for trace analysis and visualization

[Grafana](https://grafana.com/)

Unified presentation layer (Dashboards)

[DEX](https://github.com/dexidp/dex)

Enable SSO authentication for Grafana

## Component Detail

### k0rdent

Orchestration Layer

Multi-Cluster lifecycle management

-   Multi-cluster Orchestration: k0rdent handles cluster provisioning and lifecycle
    
-   Service Templates: k0rdent deploys components across clusters
    
-   Identity and RBAC: Leverages k0rdent's security mode for distribution of credentials
    
-   Configuration Management: k0rdent supports configuration through templates
    

### OpenTelemetry

Collection Layer

OpenTelemetry is the core data collection and observability engine for KOF, establishing a unified collection of metrics, logs, and traces across all k0rdent-managed clusters.

-   Vendor-neutral, standards-based telemetry collection mechanism.
    

-   Supports auto-instrumentation of applications via language-specific annotations and a custom resource
    

### OpenCost

Collection Layer

OpenCost is KOF's FinOps component; it provides cost monitoring capabilities across the k0rdent-managed Kubernetes clusters.

  

Example metrics exposed:

-   node_total_hourly_cost: Hourly cost per node
    
-   Cloud provider pricing: Real-time cloud resource costs obtained via cloud provider billing APIs
    
-   Resource allocation costs: CPU, memory, storage costs per namespace/pod
    
-   Cluster efficiency metrics: Resource utilization vs. cost ratios
    

  

The OpenCost dashboards below are included the kof-dashboards file structure and thus are automatically created in Grafana:

-   Resource-level cost breakdowns
    
-   Namespace cost allocation
    
-   Pod-level cost analysis
    
-   Historical cost trends
    

  

Using OpenCost, costs may be broken down by:

-   Per-cluster costs: Track costs across regional/child clusters
    
-   Per-namespace costs: Allocate costs to teams/applications
    
-   Per-workload costs: Understand application-level spending
    

  

FinOps optimizations:

-   Resource efficiency: Identify underutilized resources
    
-   Cluster rightsizing: Make informed scaling decisions
    
-   Budget monitoring: Set alerts on cost thresholds
    

### Promxy

Query Layer

Promxy enables KOF to query metrics from multiple clusters (identified as PromxyServerGroups) simultaneously. Promxy enables KOF to be Prometheus-compatible and improves scalability:

-   Replaces Prometheus for cross-cluster query federation and alerting
    
-   Aggregates metrics from multiple regional VictoriaMetrics clusters
    
-   Evaluates alerting rules across clusters
    

  

Thanks to Promxy’s Prometheus Protocol Compatibility:

-   KOF has full PromQL Support
    
-   Dashboards work with existing Prometheus queries
    
-   Recording and alerting rules use the Prometheus rule format
    

### VictoriaMetrics

Query Layer

Storage Layer

VictoriaMetrics serves as the time series database and query engine for OpenTelemetry collected metrics in KOF; it provides much better scalability than Prometheus while maintaining compatibility for API endpoints and PromQL support.

#### Why VictoriaMetrics vs. Prometheus?

-   More efficient long-term storage
    
-   Horizontal cluster scaling
    
-   Built-in clustering support
    
-   Better resource usage via optimized compression and metric buffering strategy
    

#### Retention

Storage retention periods can be set via the retentionPeriod field in VictoriaMetrics cluster configuration; different periods can be set at the KOF Management and KOF Regional-cluster level. Configuration can be managed via Helm values or ClusterDeployment annotations. VictoriaMetrics also has downsampling and retention filtering capabilities.

  

Retention recommendation:

-   Management Cluster: Shorter retention (1-30 days) for operational monitoring
    
-   Regional Storage: Longer retention (30-365+ days) for historical analysis
    

#### Region Awareness

Each regional KOF cluster gets its own VictoriaMetrics stack. This supports Data Sovereignty and Isolation.

#### Management Cluster Visibility

Metrics stored in VictoriaMetrics are aggregated across Region Clusters by Promxy (PromxyServerGroups).

### VictoriaLogs

Query Layer

Storage Layer

VictoriaLogs serves as the time series database and query engine for OpenTelemetry collected logs in KOF

#### Retention

Storage retention periods can be set via the retentionPeriod field in VictoriaLogs cluster configuration. Configuration can be managed via Helm values or ClusterDeployment annotations.

Retention recommendation:

-   Management Cluster: Shorter retention (1-30 days) for operational monitoring
    
-   Regional Storage: Longer retention (30-365+ days) for historical analysis
    

#### Region Awareness

Each regional KOF cluster gets its own VictoriaLogs stack. This supports Data Sovereignty and Isolation.

#### Management Cluster Visibility

Logs stored in VictoriaLogs are made visible to Grafana by GrafanaDatasources created on the Management Cluster by the kof-operator in the kof-mothership chart.

  

### Jaeger

Presentation Layer

Query Layer

Storage Layer

Jaeger serves as the distributed tracing backend that stores, queries, and visualizes traces collected via OpenTelemetry. Jaeger empowers developers to trace requests across services and identify performance bottlenecks.

#### Storage

Jaeger compliments VictoriaMetrics for storing traces. As application traces are a complex set of events and dependencies a different data model and storage schema is required. Unlike VictoriaMetrics, Jaeger does not support replication.

#### Retention

By default, KOF configures Jaeger to store a maximum of 100K traces.

  

(see [https://github.com/k0rdent/kof/blob/main/charts/kof-storage/values.yaml#L226](https://github.com/k0rdent/kof/blob/main/charts/kof-storage/values.yaml#L226))

storage:

type:  memory

options:

memory:

max-traces:  100000

  

Once the limit is reached, Jaeger will start evicting the oldest traces to make room for new ones (FIFO - First In, First Out). If Jaeger restarts, traces are lost as they're stored in memory.

#### Region Awareness

Jaeger is configured to have KOF regional awareness. KOF automatically configures child clusters to send traces to their corresponding regional Jaeger instance. Each regional KOF cluster maintains its own Jaeger instance which supports Data Sovereignty and Isolation.

  

It is necessary to connect to Jaeger via the Regional Clusters. There is no Management Cluster interface.

#### Systems Correlation

Cross-correlation between metrics and traces is possible through shared labels/metadata but would involve joining data outside of Grafana.

### Grafana

Presentation Layer

Grafana transforms KOF from a collection of observability tools into a cohesive, enterprise-ready observability platform that automatically scales and adapts to complex multi-cluster and multi-region Kubernetes environments.

  

-   Unified Interface: Single pane of glass for metrics, logs, traces, and events
    
-   Enterprise Features: SSO (via DEX OIDC provider), RBAC, multi-tenancy (via namespacing), and alerting
    
-   Cost Management: Integrated FinOps capabilities (see OpenCost section for list of FinOps-related pre-installed dashboards)
    

#### Dashboards

See [https://github.com/k0rdent/kof/tree/main/charts/kof-dashboards/files/dashboards](https://github.com/k0rdent/kof/tree/main/charts/kof-dashboards/files/dashboards)

  

KOF provides a comprehensive collection of dashboards covering the entire stack:

Infrastructure Dashboards:

-   Kubernetes cluster monitoring
    
-   Node exporter metrics
    
-   System API server monitoring
    
-   Networking and storage metrics
    
-   NVIDIA DCGM
    

  

Application Dashboards:

-   VictoriaMetrics cluster performance
    
-   VictoriaLogs operations
    
-   Jaeger tracing visualization
    
-   OpenCost financial monitoring
    

  

Service Mesh Dashboards:

-   Istio control plane monitoring
    
-   Service mesh performance
    
-   Traffic flow visualization
    

  

Platform Dashboards:

-   KCM (K0rdent Cluster Manager) operations
    
-   Cluster API monitoring
    
-   Sveltos lifecycle management
    

## DEX (SSO & OIDC)

Presentation Layer

DEX serves as the centralized Identity Provider (IdP) and Single Sign-On (SSO) gateway for KOF:

-   OAuth Flow: Enables OAuth2/OIDC authentication flow for Grafana dashboards
    
-   User Management: Handles user identity, profile information, and group membership
    

  

# Architecture (from repo docs)

See [Architecture - Documentation](https://docs.k0rdent.io/next/admin/kof/kof-architecture/)

  

KOF implements a hub-and-spoke model where:

-   Regional clusters collect metrics, logs, and traces from workloads
    
-   Storage clusters provide persistent storage for observability data
    
-   Management cluster aggregates data from all regions and provides centralized management
    
-   Promxy enables querying across multiple VictoriaMetrics instances
    

  

KOF is typically installed in 1 of 2 different topology configurations.

## Scenario 1: Production Installation w/ Regional Clusters

In this scenario, observability data generated by the Management Cluster is stored on the Management Cluster. Observability data generated by Clusters in a given Region is stored on the corresponding Regional Cluster.

[[[ regions ]]]


## Scenario 2: Development / QA Installation (Regionless)

In this scenario, there are no Regions defined, so the Management Cluster takes on the role of a Regional Cluster. Observability data from the Management Cluster is stored on the Management Cluster, alongside the observability data from all Child Clusters.

[[[ management and child ]]]


## Example Scenarios & Common Questions

This section addresses common questions and provides examples of how to achieve specific observability goals with KOF.

  

### Q: Describe "True Full Stack Observability"

KOF enables full-stack observability by integrating telemetry data from various layers of your application and infrastructure. OpenTelemetry agents collect traces, metrics, and logs from microservices, databases, and the underlying infrastructure. This data is then processed and stored in a scalable storage solution. OpenTelemetry metrics are stored in VictoriaMetrics, OpenTelemetry logs are stored in VictorlaLogs, and OpenTelemetry traces are stored in Jaeger. KOF ensures that both metrics and logs can be centrally accessed via Grafana for dashboarding. This unified view allows for rapid identification and resolution of issues across the entire stack.

  

### Q: How do I collect telemetry from a new target service?

  

Option 1: Prometheus Annotations

-   Add annotations to the service pods. KOF collectors have automatic pod discovery configured.
    

-   prometheus.io/scrape:  "true"
    
-   prometheus.io/port:  "8080"
    

  

Option 2: Enable KOF Auto-Instrumentation

-   Enabling the setting creates a custom resource in your cluster that the OpenTelemetry Operator will use for auto-instrumentation.
    

-   helm upgrade -install kof-collectors ./charts/kof-collectors --set kof.instrumentation.enabled=true --namespace kof
    

-   Next, annotate your application – set instrumentation.opentelemetry.io/inject-go: "true" (there are custom injection values for major languages)
    

  

### Q: How do I add new dashboards?

The Grafana UI has workflows for adding and deleting dashboards, however it is not recommended to make changes using the UI:

-   Using the UI for adhoc configuration changes is not aligned with k0rdent’s GitOps (Configuration as Code) philosophy
    
-   Changes made with the Grafana UI will not be tracked in version control, thus losing auditability and recoverability
    
-   Adhoc changes made through the UI will be overwritten during the next reconciliation cycle by the configuration defined in the Git repository
    

  

#### Recommended Approach: Helm Install/Upgrade via CI/CD or Manually

[[[ flows ]]]


#### Description of Process

-   First, commit a change, adding or updating dashboard YAML, to version control in the path of charts/kof-dashboards/files/dashboards/**
    
-   CI/CD solutions like Argo or GitHub Actions can detect the change and push the latest version of a new or revised chart to an OCI Registry. The CI/CD tool would then need to use helm to apply the new chart.
    

-   Example Helm Command:
    

helm upgrade --install kof-dashboards \

oci://ghcr.io/k0rdent/kof/charts/kof-dashboards \

--namespace  kof  \

--reset-values  \

--wait

  

-   If manually installing the chart from a copy of the version control repository on a filesystem, Helm can reference a file path
    

-   Example Helm Command:
    

helm  upgrade  --install  kof-dashboards  ./charts/kof-dashboards  \

--namespace  kof  \

--reset-values  \

--wait

-   The Helm kof-dashboards chart automatically finds and processes all dashboard files in the subdirectory structure. The folder structure will be reflected in Grafana to aid with organization. Helm templating is supported:
    

-   New Dashboards are rendered into GrafanaDashboard CRs
    
-   Existing GrafanaDashboard CRs are updated
    

-   The Grafana operator watches for changes to GrafanaDashboard CRDs and automagically reconciles them with dashboards currently available in Grafana
    

  

### Q: How do I delete a dashboard?

See the question above (How do I add a new dashboard?) for a discussion of why to avoid using the Grafana UI to create or manage dashboards.

#### Recommended Approach: Helm Install/Upgrade via CI/CD

-   Remove the dashboard from the path charts/kof-dashboards/files/dashboards/** in version control and commit.
    
-   If set up, CI/CD will recognize a change to the dashboard chart in version control and push a revised chart to the local registry. CI/CD then needs to trigger a reconciliation:
    

-   Example Helm Command:
    

helm  upgrade  --install  kof-dashboards  \

oci://ghcr.io/k0rdent/kof/charts/kof-dashboards \

--namespace kof \

--reset-values \  
--wait

-   Helm will delete resources that no longer exist in the chart
    

  

### Q: Multi-tenant capability – how do we store the data to avoid commingling different customers together?

  

Approach: Use different namespaces for per-tenant ClusterDeployments. KOF Collectors and Storage can be separated by different namespaces for isolation.

  

### Q: What types of telemetry are supported?

  

Types: KOF supports the three core types of observability:

-   Metrics: Numerical measurements collected over time (e.g., CPU utilization, request rates, error counts). OpenTelemetry is responsible for collecting Metrics and distributing them to VictoriaMetrics. Metrics can then be centrally accessed via a Grafana.
    
-   Logs: Timestamped records of discrete events (e.g., error messages, request logs, system events). OpenTelemetry is responsible for collecting Logs and distributing them to VictoriaLogs. Logs can then be centrally accessed via a Grafana.
    
-   Traces: End-to-end representations of requests as they flow through distributed systems, showing the sequence of operations and their latencies. OpenTelemetry is responsible for collecting Traces and distributing them to Jeager.
    

-   If the KOF Installation has Regions configured, Traces may be analyzed using the Jaeger endpoint on the Regional Cluster. If the KOF Installation is running in a Regionless configuration, Traces may be analyzed using a Jaeger instance on the Management Cluster.
    

  

### Q: If I define a new service via a template, what are the steps?

  

Steps:

-   When using a service template (e.g., a Kubernetes manifest or k0rdent service definition), the template typically includes pre-configured OpenTelemetry instrumentation and collector configurations.
    
-   Fill in service-specific parameters, and upon deployment via k0rdent, the service will automatically begin emitting telemetry that is picked up by the OpenTelemetry collectors.
    

  

### Q: What are the telemetry retention strategies available?

  

Granularity Capabilities

  
VictoriaMetrics and VictoriaLogs support:

-   Configurable storage limits
    
-   Configurable replication factor
    

-   Higher replication improves resiliency and ability to handle load
    

-   Configurable Retention Periods for both VictoriaMetrics and Victoria Logs.
    

Example:

  

victoriametrics:

vmcluster:

spec:

retentionPeriod: "30d"

replicationFactor: 2

vmstorage:

storage:

volumeClaimTemplate:

spec:

resources:

requests:

storage: 100Gi

  
  

victoria-logs-cluster:

vlstorage:

extraArgs:

retentionPeriod: "30d" # Days

persistentVolume:

size: "100Gi"