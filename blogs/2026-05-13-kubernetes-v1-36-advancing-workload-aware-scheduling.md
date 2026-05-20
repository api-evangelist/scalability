---
title: "Kubernetes v1.36: Advancing Workload-Aware Scheduling"
url: "https://kubernetes.io/blog/2026/05/13/kubernetes-v1-36-advancing-workload-aware-scheduling/"
date: "Wed, 13 May 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
AI/ML and batch workloads introduce unique scheduling challenges that go beyond simple Pod-by-Pod scheduling. In Kubernetes v1.35, we introduced the first tranche of workload-aware scheduling improvements, featuring the foundational Workload API alongside basic gang scheduling support built on a Pod-based framework, and an opportunistic batching feature to efficiently process identical Pods. Kubernetes v1.36 introduces a significant architectural evolution by cleanly separating API concerns: the Workload API acts as a static template, while the new PodGroup API handles the runtime state.
