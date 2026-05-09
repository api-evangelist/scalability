---
title: "Kubernetes v1.36: Server-Side Sharded List and Watch"
url: "https://kubernetes.io/blog/2026/05/06/kubernetes-v1-36-server-side-sharded-list-and-watch/"
date: "2026-05-06"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
As Kubernetes clusters grow to tens of thousands of nodes, controllers that watch high-cardinality resources like Pods face a scaling wall. Every replica of a horizontally scaled controller receives the full stream of events from the API server, paying the CPU, memory, and network cost to deserialize everything, only to discard the objects it is not responsible for. Scaling out the controller does not reduce per-replica cost; it multiplies it. Kubernetes v1.36 introduces server-side sharded list and watch as an alpha feature (KEP-5866).
