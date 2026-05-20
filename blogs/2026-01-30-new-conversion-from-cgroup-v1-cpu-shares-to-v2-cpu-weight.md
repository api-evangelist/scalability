---
title: "New Conversion from cgroup v1 CPU Shares to v2 CPU Weight"
url: "https://kubernetes.io/blog/2026/01/30/new-cgroup-v1-to-v2-cpu-conversion-formula/"
date: "Fri, 30 Jan 2026 08:00:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
I'm excited to announce the implementation of an improved conversion formula from cgroup v1 CPU shares to cgroup v2 CPU weight. This enhancement addresses critical issues with CPU priority allocation for Kubernetes workloads when running on systems with cgroup v2. Background Kubernetes was originally designed with cgroup v1 in mind, where CPU shares were derived from a container's CPU requests using the following formula: $$cpu.shares = milliCPU \times \frac{1024}{1000}$$ Note that the value 1024 in this formula is the default cpu.shares value in cgroup v1, and is unrelated to millicores.
