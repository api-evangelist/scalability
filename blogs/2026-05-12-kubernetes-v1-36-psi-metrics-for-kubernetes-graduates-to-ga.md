---
title: "Kubernetes v1.36: PSI Metrics for Kubernetes Graduates to GA"
url: "https://kubernetes.io/blog/2026/05/12/kubernetes-v1-36-psi-metrics-ga/"
date: "Tue, 12 May 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Since its original implementation in the Linux kernel in 2018, Pressure Stall Information (PSI) has provided users with the high-fidelity signals needed to identify resource saturation before it becomes an outage. Unlike traditional utilization metrics, PSI tells the story of tasks stalled and time lost, all in nicely-packaged percentages of time across the CPU, memory, and I/O. With the recent release of Kubernetes v1.36, users across the ecosystem have a stable, reliable interface to observe resource contention at the node, pod, and container levels.
