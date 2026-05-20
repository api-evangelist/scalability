---
title: "Kubernetes v1.35: Introducing Workload Aware Scheduling"
url: "https://kubernetes.io/blog/2025/12/29/kubernetes-v1-35-introducing-workload-aware-scheduling/"
date: "Mon, 29 Dec 2025 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Scheduling large workloads is a much more complex and fragile operation than scheduling a single Pod, as it often requires considering all Pods together instead of scheduling each one independently. For example, when scheduling a machine learning batch job, you often need to place each worker strategically, such as on the same rack, to make the entire process as efficient as possible. At the same time, the Pods that are part of such a workload are very often identical from the scheduling perspective, which fundamentally changes how this process should look.
