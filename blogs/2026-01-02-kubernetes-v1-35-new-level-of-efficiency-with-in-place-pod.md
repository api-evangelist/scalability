---
title: "Kubernetes v1.35: New level of efficiency with in-place Pod restart"
url: "https://kubernetes.io/blog/2026/01/02/kubernetes-v1-35-restart-all-containers/"
date: "Fri, 02 Jan 2026 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
The release of Kubernetes 1.35 introduces a powerful new feature that provides a much-requested capability: the ability to trigger a full, in-place restart of the Pod. This feature, Restart All Containers (alpha in 1.35), allows for an efficient way to reset a Pod's state compared to resource-intensive approach of deleting and recreating the entire Pod. This feature is especially useful for AI/ML workloads allowing application developers to concentrate on their core training logic while offloading complex failure-handling and recovery mechanisms to sidecars and declarative Kubernetes…
