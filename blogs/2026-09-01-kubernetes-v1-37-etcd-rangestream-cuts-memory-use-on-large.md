---
title: "Kubernetes v1.37: etcd RangeStream Cuts Memory Use on Large List Reads"
url: "https://kubernetes.io/blog/2026/09/01/kubernetes-v1-37-etcd-range-stream/"
date: "2026-09-01"
feed_url: "https://kubernetes.io/feed.xml"
---
I am excited to announce that etcd RangeStream is graduating to beta in Kubernetes v1.37. Paired with etcd v3.7, it reduces the memory the API server and etcd need to read a large collection, and makes peak usage more predictable. The cost of large reads The API server serves most list and watch requests from its in-memory watch cache.
