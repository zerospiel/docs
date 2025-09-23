# {{{ docsVersionInfo.k0rdentName }}} Observability and FinOps

KOF transforms Kubernetes from an operational black box into a transparent, accountable, and scalable platform for both engineers and business leaders.

## Why KOF Exists

Running Kubernetes at scale introduces two universal problems:

1. **Observability is fragmented.** Teams struggle to piece together metrics, logs, and traces across clusters.
2. **Cloud bills are opaque.** Finance sees rising costs, but engineering can’t explain who or what is responsible.

**KOF (k0rdent Observability & FinOps)** solves both problems by unifying observability and cost management in a single, GitOps-driven platform.

## The Problems KOF Solves

KOF solves a number of problems that keep business from truly extracting the value from Kubernetes, such as:

* **Cluster sprawl:** KOF provides one control plane for metrics, logs, traces, and costs across all clusters.
* **Escalating cloud bills:** KOF presents namespace- and pod-level cost dashboards tied directly to workloads.
* **Slow troubleshooting:** KOF exposes logs, metrics, and distributed traces side-by-side for faster root-cause analysis.
* **Compliance pressure:** KOF enbales policy-driven retention and replication for metrics and logs (from 30–365+ days).
* **Operational drift:** KOF enables you to manage dashboards, alerts, and retention policies as code through CI/CD.

## Who Needs KOF

KOF simplifes Kubernetes usage and operations throughout the organization.

* **Platform Engineers:** Simplify multi-cluster observability, scale beyond Prometheus limits, enforce GitOps workflows.
* **DevOps Teams:** Debug faster with integrated logs, metrics, and tracing. Gain a single view across hybrid/multi-cloud deployments.
* **Finance / FinOps Teams:** Track real-time costs, run chargeback/showback, and forecast budgets with confidence.
* **Compliance & Security:** Retain data for audits, enforce RBAC, and secure communications by default.


# Pain Points & Solutions

KOF works to solve pratical problems involved in running Kubernetes in production.

## Cluster Sprawl and Fragmented Observability

**The problem:** As Kubernetes adoption grows, teams end up with dozens of clusters, each running its own Prometheus, Grafana, and logging stack. Dashboards drift, troubleshooting spans multiple tools, and SREs burn hours stitching together partial data.

**KOF solution:** KOF provides a **single control plane** for metrics, logs, traces, and cost data across management, regional, and child clusters. Dashboards, alerts, and retention rules are managed via GitOps, keeping environments consistent.

**Business impact:** Simplifies operations, reduces tool sprawl, and saves engineering time by providing one unified view of cluster health.

## Escalating Cloud Bills With No Accountability

**The problem:** Finance sees rising Kubernetes costs, but engineering can’t explain who’s responsible. Cost reports are high-level cloud invoices, not actionable insights.

**KOF solution:** KOF integrates **OpenCost**, surfacing pod-, namespace-, and team-level costs directly in Grafana. It enables **chargeback/showback** so teams can own their spend.

**Business impact:** Turns cloud bills into actionable insights, reduces waste, and promotes cost accountability across teams.

## Slow Troubleshooting in Microservices Environments

**The problem:** In distributed systems, issues hide across logs, metrics, and traces. Without a unified view, root cause analysis takes days, and downtime costs escalate.

**KOF solution:** KOF integrates **VictoriaMetrics, VictoriaLogs, and Jaeger** under one roof. Engineers can correlate metrics, logs, and traces from the same dashboard.

**Business impact:** Cuts mean-time-to-resolution (MTTR), reduces downtime costs, and increases developer confidence in production.

## Compliance and Audit Pressure

**The problem:** Regulations often require retaining logs and metrics for 90–365 days or more. DIY Prometheus/log stacks can’t scale without spiraling storage costs.

**KOF solution:** KOF supports **policy-driven retention and replication**, with scalable backends like VictoriaMetrics and VictoriaLogs. Configure once, enforce everywhere.

**Business impact:** Meets audit requirements cost-effectively while keeping engineers focused on delivery instead of compliance plumbing.

## Operational Drift Across Teams

**The problem:** Different teams edit Grafana dashboards, alert rules, or log retention policies by hand, causing environments to drift out of sync.

**KOF solution:** KOF enforces a **GitOps workflow**. Dashboards, alerts, and retention settings are stored in Git, deployed via CI/CD pipelines, and version-controlled.

**Business impact:** Eliminates configuration drift, improves collaboration, and ensures repeatability across dev, staging, and production.

Got it — here’s the next section, **“Who Benefits”**, written audience-first. Each role gets its own story of why KOF matters to them, in plain language that connects to their responsibilities.



# Who Benefits from KOF

## Platform Engineers & SREs

**Your challenge:** You manage Kubernetes clusters at scale, but every new region or cluster brings another Prometheus, another Grafana, another set of YAML to maintain. Observability stacks drift, upgrades break, and scale testing becomes painful.

**How KOF helps you:**

* One control plane across management, regional, and child clusters.
* Handles millions of metrics per second without federation hacks.
* GitOps-native: dashboards, alerts, and retention policies are versioned in Git and deployed via CI/CD.

**What this means for you:** Fewer moving parts, fewer late-night firefights, and the ability to scale observability as fast as you scale Kubernetes itself.

## DevOps & Application Teams

**Your challenge:** You’re on the hook for application performance, but your current tools split logs, metrics, and traces across silos. Finding the root cause of a production issue takes hours, sometimes days.

**How KOF helps you:**

* Unified dashboards that correlate metrics, logs, and traces.
* Jaeger-powered distributed tracing to follow requests across microservices.
* Instant cost visibility for the workloads you own.

**What this means for you:** Faster MTTR, fewer blind spots in production, and real-time cost accountability that’s baked into the same Grafana dashboards you already use.

## Finance & FinOps Teams

**Your challenge:** Cloud bills arrive as massive line items with little visibility into *who* or *what* drove the spend. You’re left begging engineers for explanations — and they can’t connect usage to dollars.

**How KOF helps you:**

* Namespace- and pod-level cost allocation via OpenCost.
* Chargeback and showback dashboards to attribute spend to teams, projects, or departments.
* Budget alerts tied directly to Kubernetes workloads.

**What this means for you:** Finally, cost visibility that matches how your business operates — and the leverage to hold teams accountable for their cloud footprint.

## Compliance, Security & Governance

**Your challenge:** You must meet strict audit and retention requirements, but current observability stacks aren’t built for compliance. Logs disappear too soon, retention is inconsistent, and auditors don’t care that Prometheus is “hard to scale.”

**How KOF helps you:**

* Policy-driven retention (30–365+ days) for logs and metrics.
* Secure by default: RBAC and TLS encryption baked in.
* Audit trails and dashboards that prove compliance.

**What this means for you:** Compliance without compromise — observability data is preserved, secured, and accessible when regulators come knocking.

Here’s the **“KOF in Action”** case study section, designed as a narrative that shows how different teams interact with KOF to solve a real-world problem. It reads like a story instead of a manual, which helps “sell” the platform.

## Core Capabilities (at a Glance)

k0rdent Observability and FinOps includes the everything you need to know what's going on inside your cluster, including:

* **Metrics:** Enterprise-grade time series storage (VictoriaMetrics).
* **Logs:** Centralized collection and analysis (VictoriaLogs).
* **Tracing:** Distributed tracing for microservices (Jaeger + OpenTelemetry).
* **Cost Management:** Kubernetes-aware cost visibility (OpenCost).
* **GitOps Integration:** Manage dashboards, alerts, and retention policies as code.
* **Compliance:** Retention policies, audit trails, RBAC, and TLS-secured communication.



## KOF in Action: A Day in the Life (and A Sudden Cost Spike)

It’s Monday morning. Finance notices that last week’s cloud bill is **20% higher than forecast**. The CFO wants answers before the leadership meeting tomorrow.

Without KOF, this would trigger days of Slack threads and finger-pointing. With KOF, here’s what happens instead:

### Step 1: Finance Spots the Anomaly

The **Finance team** opens the KOF FinOps dashboard in Grafana. They see that most of the increase came from the **US-East regional cluster**, specifically from workloads tagged to the “recommendation-engine” project.

*What used to be a mystery bill is now an actionable insight.*

### Step 2: DevOps Traces the Issue

The **DevOps team** takes over. They drill into the same dashboard and pull up **OpenCost metrics** for that namespace. One service shows a sharp increase in CPU and memory usage starting on Friday.

To understand why, they pivot into **Jaeger traces**. The culprit emerges: a new API endpoint is generating far more queries than expected.

*What used to take days of log-diving is discovered in minutes.*

### Step 3: Platform Engineers Validate the Fix

The **Platform engineering team** checks KOF’s **cluster overview dashboards** to confirm whether this workload is affecting other services. They see elevated resource usage but no system-wide instability.

They work with DevOps to roll out a fix via GitOps: right-sizing the deployment and setting an alert for future usage spikes.

*What used to be firefighting is now controlled, repeatable operations.*

### Step 4: Finance Closes the Loop

By the end of the day, Finance refreshes the **chargeback dashboard**. Costs for the “recommendation-engine” namespace are already trending down. The CFO gets a simple, confident answer for tomorrow’s meeting:

"We found the root cause, fixed it, and costs are back under control."

### The Outcome

* **Finance**: Cost visibility and accountability.
* **DevOps**: Faster root cause analysis, no wasted hours in silos.
* **Platform Engineers**: A clean GitOps workflow to fix the issue.
* **Leadership**: Confidence that teams are in control of performance *and* costs.

In a single day, KOF turned a vague cost spike into a resolved issue, with every team playing their part in one unified system.
# Getting Started

KOF lets you get started with some quick wins. For example, you can easily:

* Spin up a single cluster and access Grafana dashboards in under 10 minutes.
* Attribute cloud costs to teams immediately using OpenCost metrics.
* Configure alerts for both performance and budget thresholds.
* Start retaining logs and metrics for compliance right out of the box.




## Managing KOF as Code

We recommend keeping a dedicated Git repository for your KOF deployment, separate from the {{{ docsVersionInfo.k0rdentName }}} repo itself and separate from application workload repos.

That repository should contain the Helm chart directories such as:

```
charts/
  kof-storage/
  kof-operators/
  kof-dashboards/
```

With this setup you can:
- Track all KOF configuration changes in Git.
- Use CI/CD pipelines to run `helm upgrade --install` against your management and regional clusters.
- Manage dashboards, retention policies, and observability settings as code for consistency across environments.

Other sections in this documentation, such as Dashboard Lifecycle and Retention assume you have such a repo and a CI/CD pipeline in place.

## DIY Stack vs. KOF

| Dimension                  | DIY Stack (Prometheus, Grafana, OpenCost, Jaeger, etc.)                               | **KOF**                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Setup & Integration**    | Assemble 4–6 separate projects, each with its own configs, upgrades, and quirks.      | One integrated platform: metrics, logs, traces, and costs shipped together.                          |
| **Scale & Performance**    | Prometheus federation slows at millions of samples; log systems require heavy tuning. | VictoriaMetrics/Logs designed for **millions of samples/sec**, fast queries, and long-term storage.  |
| **Consistency**            | Dashboards, alerts, and retention vary per cluster; drift happens quickly.            | **GitOps-native**: dashboards, alerts, and policies stored in Git and deployed via CI/CD.            |
| **Troubleshooting**        | Metrics in one tool, logs in another, traces in a third. Correlation is manual.       | Unified Grafana dashboards: metrics, logs, and traces correlated automatically.                      |
| **Cost Management**        | OpenCost must be bolted on; engineers rarely look at it.                              | Costs embedded in Grafana dashboards engineers already use; supports chargeback/showback.            |
| **Multi-Cluster Support**  | Each cluster runs its own observability stack; federation is fragile.                 | **Single control plane** for management, regional, and child clusters.                               |
| **Compliance & Retention** | Long-term retention requires custom S3/Elasticsearch setups, costly and brittle.      | Policy-driven retention (30–365+ days) and replication built in.                                     |
| **Security & Governance**  | Role-based access is piecemeal, TLS often manual.                                     | RBAC and secure communication enforced by default.                                                   |
| **Upgrades**               | Upgrade each component independently; breakage risk is high.                          | Guided version-to-version upgrades with clear migration paths.                                       |
| **Business Impact**        | Tools work, but require constant integration effort; finance rarely gets visibility.  | Engineers debug faster, finance gets cost accountability, compliance gets retention — in one system. |


## Guides

Get started with the basic documentation:

- [Architecture](kof-architecture.md)
- [Installing KOF](kof-install.md)
- [Verifying the KOF installation](kof-verification.md)
- [Storing KOF data](kof-storing.md)
- [Using KOF](kof-using.md)
- [KOF Alerts](kof-alerts.md)
- [KOF Tracing](kof-tracing.md)
- [Retention and Replication ](kof-retention.md)
- [Limits](kof-limits.md)
- [Scaling KOF](kof-scaling.md)
- [Maintaining KOF](kof-maintainence.md)
- [Upgrading KOF](kof-upgrade.md)
- [FAQ](kof-faq.md)

Once you have KOF up and running,
check [k0rdent/kof/docs](https://github.com/k0rdent/kof/tree/v{{{ extra.docsVersionInfo.kofVersions.kofDotVersion }}}/docs)
for advanced guides.
