# Scalability

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
