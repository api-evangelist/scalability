---
title: "Kubernetes v1.37: Native Histograms Graduates to Beta"
url: "https://kubernetes.io/blog/2026/09/11/kubernetes-v1-37-native-histograms-beta/"
date: "2026-09-11"
feed_url: "https://kubernetes.io/feed.xml"
---
I'm excited to announce that native histogram support for Kubernetes metrics is graduating to Beta and is enabled by default in Kubernetes v1.37! Native histograms (previously introduced as Alpha in Kubernetes v1.36 under KEP-5808 ) bring high-resolution, low-cardinality observability to Kubernetes metrics. By adopting Prometheus Native Histograms , Kubernetes components now expose latency and duration metrics with far greater accuracy while significantly reducing telemetry storage and scraping overhead.
