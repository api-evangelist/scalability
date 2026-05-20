---
title: "SELinux Volume Label Changes goes GA (and likely implications in v1.37)"
url: "https://kubernetes.io/blog/2026/04/22/breaking-changes-in-selinux-volume-labeling/"
date: "Wed, 22 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
If you run Kubernetes on Linux with SELinux in enforcing mode, plan ahead: a future release (anticipated to be v1.37) is expected to turn the SELinuxMount feature gate on by default. This makes volume setup faster for most workloads, but it can break applications that still depend on the older recursive relabeling model in subtle ways (for example, sharing one volume between privileged and unprivileged Pods on the same node). Kubernetes v1.36 is the right release to audit your cluster and fix or opt out of this change.
