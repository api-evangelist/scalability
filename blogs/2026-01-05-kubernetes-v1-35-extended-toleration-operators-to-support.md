---
title: "Kubernetes v1.35: Extended Toleration Operators to Support Numeric Comparisons (Alpha)"
url: "https://kubernetes.io/blog/2026/01/05/kubernetes-v1-35-numeric-toleration-operators/"
date: "Mon, 05 Jan 2026 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Many production Kubernetes clusters blend on-demand (higher-SLA) and spot/preemptible (lower-SLA) nodes to optimize costs while maintaining reliability for critical workloads. Platform teams need a safe default that keeps most workloads away from risky capacity, while allowing specific workloads to opt-in with explicit thresholds like "I can tolerate nodes with failure probability up to 5%". Today, Kubernetes taints and tolerations can match exact values or check for existence, but they can't compare numeric thresholds.
