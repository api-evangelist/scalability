---
title: "Building a Custom Metrics Exporter for Kubernetes"
url: "https://kubernetes.io/blog/2026/07/14/custom-metrics-exporter-kubernetes/"
date: "2026-07-14"
feed_url: "https://kubernetes.io/feed.xml"
---
Kubernetes ships with built-in awareness of CPU and memory, but most real-world scaling decisions depend on signals that live entirely outside that narrow window: how many messages are waiting in a queue, how long the last batch job took, how many active WebSocket connections a pod is holding. When the built-in metrics are not enough, a metrics exporter bridges that gap. This post walks through writing one from scratch, packaging it as a container, and wiring it into a cluster so that Prometheus — and ultimately the HorizontalPodAutoscaler — can consume it.
