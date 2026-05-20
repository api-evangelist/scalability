---
title: "Kubernetes v1.36: Deprecation and removal of Service ExternalIPs"
url: "https://kubernetes.io/blog/2026/05/14/kubernetes-v1-36-deprecation-and-removal-of-service-externalips/"
date: "Thu, 14 May 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
The .spec.externalIPs field for Service was an early attempt to provide cloud-load-balancer-like functionality for non-cloud clusters. Unfortunately, the API assumes that every user in the cluster is fully trusted, and in any situation where that is not the case, it enables various security exploits, as described in CVE-2020-8554 . Since Kubernetes 1.21, the Kubernetes project has recommended that all users disable .spec.externalIPs .
