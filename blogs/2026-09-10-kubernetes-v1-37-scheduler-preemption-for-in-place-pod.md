---
title: "Kubernetes v1.37: Scheduler Preemption for In-Place Pod Resize (Alpha)"
url: "https://kubernetes.io/blog/2026/09/10/kubernetes-v1-37-scheduler-preemption-for-in-place-pod-resize-alpha/"
date: "2026-09-10"
feed_url: "https://kubernetes.io/feed.xml"
---
In Kubernetes, resource allocation has historically been a static decision made during a Pod's initial scheduling and placement. With the graduation of the core in-Place Pod resize feature to General Availability in v1.35, application developers and cluster operators gained the powerful ability to dynamically adjust CPU and memory allocations of running containers without incurring disruptive restarts or application downtime. However, in-place resizing introduced a unique resource scheduling gap: if a running Pod requested a resource scale-up that exceeded the host node's allocatable headroom,
