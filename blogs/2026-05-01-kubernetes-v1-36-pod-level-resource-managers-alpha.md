---
title: "Kubernetes v1.36: Pod-Level Resource Managers (Alpha)"
url: "https://kubernetes.io/blog/2026/05/01/kubernetes-v1-36-feature-pod-level-resource-managers-alpha/"
date: "Fri, 01 May 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Kubernetes v1.36 introduces Pod-Level Resource Managers as an alpha feature, bringing a more flexible and powerful resource management model to performance-sensitive workloads. This enhancement extends the kubelet's Topology, CPU, and Memory Managers to support pod-level resource specifications ( .spec.resources ), evolving them from a strictly per-container allocation model to a pod-centric one. Why do we need pod-level resource managers?
