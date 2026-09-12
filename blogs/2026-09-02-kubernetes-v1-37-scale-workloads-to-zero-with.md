---
title: "Kubernetes v1.37: Scale Workloads to Zero with HorizontalPodAutoscaler"
url: "https://kubernetes.io/blog/2026/09/02/kubernetes-v1-37-hpa-scale-to-zero-beta/"
date: "2026-09-02"
feed_url: "https://kubernetes.io/feed.xml"
---
Kubernetes v1.37 includes API support for horizontal autoscaling of workloads down to zero replicas. This feature is now Beta and enabled by default. A HorizontalPodAutoscaler (HPA) that uses a suitable object metric or external metric can now scale a workload to zero replicas, then bring it back when the metric changes.
