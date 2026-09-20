---
title: "Kubernetes v1.37: Memory QoS Graduates to Beta"
url: "https://kubernetes.io/blog/2026/09/14/kubernetes-v1-37-memory-qos-graduates-to-beta/"
date: "2026-09-14"
feed_url: "https://kubernetes.io/feed.xml"
---
Memory QoS has graduated to Beta in Kubernetes v1.37 and is now enabled by default. On Linux nodes running cgroup v2, the feature uses the memory controller to give the kernel better guidance on how to treat container memory. It was first introduced as Alpha in v1.22, and expanded in v1.36 with tiered memory reservation.
