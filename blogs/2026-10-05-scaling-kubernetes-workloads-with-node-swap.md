---
title: "Scaling Kubernetes Workloads with Node Swap"
url: "https://kubernetes.io/blog/2026/10/05/scaling-kubernetes-workloads-with-node-swap/"
date: "2026-10-05"
feed_url: "https://kubernetes.io/feed.xml"
---
Memory is often the first hard limit a Kubernetes cluster hits. Nodes run out of RAM long before they run out of CPU, and the new wave of agentic AI workloads makes this worse. These workloads demand large memory footprints to start up and run untrusted code, then sit idle waiting for the next prompt.
