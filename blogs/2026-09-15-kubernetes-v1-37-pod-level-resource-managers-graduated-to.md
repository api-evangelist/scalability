---
title: "Kubernetes v1.37: Pod-Level Resource Managers graduated to Beta"
url: "https://kubernetes.io/blog/2026/09/15/kubernetes-v1-37-pod-level-resource-managers-beta/"
date: "2026-09-15"
feed_url: "https://kubernetes.io/feed.xml"
---
With the release of Kubernetes v1.37, the Pod-Level Resource Managers feature has graduated to Beta status (disabled by default)! First introduced as an Alpha feature in Kubernetes v1.36 , this enhancement builds on Pod-Level Resources by equipping Kubelet's Topology Manager, CPU Manager, and Memory Manager to use Pod-level resource declarations ( .spec.resources ) directly when making hardware placement decisions. Bringing pod-level resources to node managers Before this feature, obtaining exclusive NUMA-aligned CPU cores or memory for latency-critical applications forced cluster operators in
