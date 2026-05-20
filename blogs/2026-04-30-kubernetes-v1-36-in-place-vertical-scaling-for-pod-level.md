---
title: "Kubernetes v1.36: In-Place Vertical Scaling for Pod-Level Resources Graduates to Beta"
url: "https://kubernetes.io/blog/2026/04/30/kubernetes-v1-36-inplace-pod-level-resources-beta/"
date: "Thu, 30 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Following the graduation of Pod-Level Resources to Beta in v1.34 and the General Availability (GA) of In-Place Pod Vertical Scaling in v1.35, the Kubernetes community is thrilled to announce that In-Place Pod-Level Resources Vertical Scaling has graduated to Beta in v1.36! This feature is now enabled by default via the InPlacePodLevelResourcesVerticalScaling feature gate. It allows users to update the aggregate Pod resource budget ( .spec.resources ) for a running Pod, often without requiring a container restart.
