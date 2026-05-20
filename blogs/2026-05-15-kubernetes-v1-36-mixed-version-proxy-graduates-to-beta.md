---
title: "Kubernetes v1.36: Mixed Version Proxy Graduates to Beta"
url: "https://kubernetes.io/blog/2026/05/15/kubernetes-1-36-feature-mixed-version-proxy-beta/"
date: "Fri, 15 May 2026 10:00:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Back in Kubernetes 1.28, we introduced the Mixed Version Proxy (MVP) as an Alpha feature (under the feature gate UnknownVersionInteroperabilityProxy ) in a previous blog post . The goal was simple but critical: make cluster upgrades safer by ensuring that requests for resources not yet known to an older API server are correctly routed to a newer peer API server, instead of returning an incorrect 404 Not Found . We are excited to announce that the Mixed Version Proxy is moving to Beta in Kubernetes 1.36 and will be enabled by default!
