---
title: "The Shift to cgroup v2 in Kubernetes: What You Need to Know"
url: "https://kubernetes.io/blog/2026/10/06/kubernetes-cgroups-v2-shift/"
date: "2026-10-06"
feed_url: "https://kubernetes.io/feed.xml"
---
In Linux, cgroups (control groups) are a kernel feature used for managing system resources. Kubernetes uses cgroups to allocate resources like CPU and memory to containers, ensuring that applications run smoothly without interfering with each other. With the release of Kubernetes v1.31, support for v1 cgroup management moved into maintenance mode .
