---
title: "Kubernetes v1.35: Job Managed By Goes GA"
url: "https://kubernetes.io/blog/2025/12/18/kubernetes-v1-35-job-managedby-for-jobs-goes-ga/"
date: "Thu, 18 Dec 2025 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
In Kubernetes v1.35, the ability to specify an external Job controller (through .spec.managedBy ) graduates to General Availability. This feature allows external controllers to take full responsibility for Job reconciliation, unlocking powerful scheduling patterns like multi-cluster dispatching with MultiKueue . Why delegate Job reconciliation?
