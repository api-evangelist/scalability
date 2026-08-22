# Scalability

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

A subject-matter collection covering APIs, tools, frameworks, and data sources related to application scalability, infrastructure scaling, performance optimization, and elastic resource management. This topic spans cloud provider auto-scaling, event-driven autoscaling (KEDA), load balancing, database scaling, and observability for scale.

**URL:** [https://raw.githubusercontent.com/api-evangelist/scalability/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/scalability/refs/heads/main/apis.yml)

## Tags

Auto Scaling, Cloud Computing, DevOps, Distributed Systems, Elasticity, High Availability, Infrastructure, Load Balancing, Performance, Scalability

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-05-02

## APIs

### KEDA (Kubernetes Event-Driven Autoscaling) API
CNCF graduate project for fine-grained event-driven autoscaling of Kubernetes workloads, supporting 70+ built-in scalers and scale-to-zero.

**Human URL:** [https://keda.sh/](https://keda.sh/)

#### Tags

Auto Scaling, CNCF, Event-Driven, Kubernetes, Scale To Zero

#### Properties

- [Documentation](https://keda.sh/docs/)
- [GitHub](https://github.com/kedacore/keda)
- [Changelog](https://github.com/kedacore/keda/releases)
- [Blog](https://keda.sh/blog/)

### AWS Auto Scaling API
Amazon Web Services Auto Scaling for EC2 instances, ECS services, DynamoDB tables, Lambda concurrency, and more.

**Human URL:** [https://aws.amazon.com/autoscaling/](https://aws.amazon.com/autoscaling/)

#### Tags

Amazon Web Services, Auto Scaling, Cloud, EC2, Elasticity

#### Properties

- [Documentation](https://docs.aws.amazon.com/autoscaling/)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/autoscaling/2011-01-01/openapi.yaml)
- [Pricing](https://aws.amazon.com/autoscaling/pricing/)

### Google Cloud Compute Engine Autoscaler API
Google Cloud Autoscaler for managed instance groups and GKE cluster autoscaling.

**Human URL:** [https://cloud.google.com/compute/docs/autoscaler](https://cloud.google.com/compute/docs/autoscaler)

#### Tags

Auto Scaling, Cloud, GKE, Google Cloud, Instance Groups

#### Properties

- [Documentation](https://cloud.google.com/compute/docs/autoscaler)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/googleapis.com/compute/v1/openapi.yaml)
- [Pricing](https://cloud.google.com/compute/pricing)
- [SDK](https://cloud.google.com/sdk/docs)

### Azure Autoscale REST API
Microsoft Azure Autoscale for VM Scale Sets, App Service, and Container Apps.

**Human URL:** [https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-overview](https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-overview)

#### Tags

Auto Scaling, Azure, Cloud, Microsoft, Virtual Machine Scale Sets

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-overview)
- [OpenAPI](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/main/specification/monitor/resource-manager/Microsoft.Insights/stable/2022-10-01/autoScale_API.json)
- [SDK](https://learn.microsoft.com/en-us/azure/developer/)

### CloudWatch Application Signals API
Amazon CloudWatch APM for detecting and diagnosing performance issues to inform scaling decisions.

**Human URL:** [https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals.html)

#### Tags

Amazon Web Services, Observability, APM, Monitoring, Performance

#### Properties

- [Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals.html)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/monitoring/2010-08-01/openapi.yaml)

### Prometheus HTTP API
CNCF graduate open-source monitoring toolkit; the de-facto metrics source for scalability observability and custom autoscaling triggers.

**Human URL:** [https://prometheus.io/](https://prometheus.io/)

#### Tags

CNCF, Metrics, Monitoring, Observability, Open Source, Prometheus, Time Series

#### Properties

- [Documentation](https://prometheus.io/docs/prometheus/latest/querying/api/)
- [GitHub](https://github.com/prometheus/prometheus)

### Grafana HTTP API
Open-source analytics and observability platform for visualizing scalability metrics, dashboards, and alerts.

**Human URL:** [https://grafana.com/](https://grafana.com/)

#### Tags

Dashboards, Grafana, Metrics, Monitoring, Observability, Open Source

#### Properties

- [Documentation](https://grafana.com/docs/grafana/latest/developers/http_api/)
- [GitHub](https://github.com/grafana/grafana)
- [Pricing](https://grafana.com/pricing/)

## Schemas

| Artifact | Description |
|---|---|
| [Scaling Policy Schema](json-schema/scalability-scaling-policy-schema.json) | JSON Schema for auto-scaling policies covering KEDA ScaledObjects, triggers, min/max replicas, and scaling behavior rules. |
| [Load Balancer Schema](json-schema/scalability-load-balancer-schema.json) | JSON Schema for load balancer configuration including algorithms, backends, health checks, and session affinity. |

## Structures

| Artifact | Description |
|---|---|
| [Scaling Policy Structure](json-structure/scalability-scaling-policy-structure.json) | Hierarchical field documentation for scaling policy objects. |
| [Load Balancer Structure](json-structure/scalability-load-balancer-structure.json) | Hierarchical field documentation for load balancer configurations including backend pools, health checks, and session affinity. |

## Linked Data

| Artifact | Description |
|---|---|
| [Scalability Context](json-ld/scalability-context.jsonld) | JSON-LD context mapping scalability vocabulary to schema.org and KEDA namespaces. |

## Examples

| Artifact | Description |
|---|---|
| [KEDA ScaledObject Example](examples/scalability-keda-scaled-object-example.json) | Example KEDA ScaledObject for a Kafka consumer with scale-to-zero and rate-limit policies. |
| [Load Balancer Example](examples/scalability-load-balancer-example.json) | Example L7 load balancer configuration with least-connections algorithm, HTTPS, TLS termination, three backends, and health checks. |

## Vocabulary

| Artifact | Description |
|---|---|
| [Scalability Vocabulary](vocabulary/scalability-vocabulary.yml) | Normative vocabulary for auto scaling, load balancing, Kubernetes scaling, observability, and resilience patterns. |

## Common Properties

- [GitHub Organization](https://github.com/kedacore)
- [CNCF Landscape](https://landscape.cncf.io/card-mode?category=auto-scaling)
- [Blog](https://kubernetes.io/blog/)

## Maintainers

**API Evangelist** — [kin@apievangelist.com](mailto:kin@apievangelist.com) — [https://apievangelist.com](https://apievangelist.com)
