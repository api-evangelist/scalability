---
title: "Securing Production Debugging in Kubernetes"
url: "https://kubernetes.io/blog/2026/03/18/securing-production-debugging-in-kubernetes/"
date: "Wed, 18 Mar 2026 10:00:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
During production debugging, the fastest route is often broad access such as cluster-admin (a ClusterRole that grants administrator-level access), shared bastions/jump boxes, or long-lived SSH keys. It works in the moment, but it comes with two common problems: auditing becomes difficult, and temporary exceptions have a way of becoming routine. This post offers my recommendations for good practices applicable to existing Kubernetes environments with minimal tooling changes: Least privilege with RBAC Short-lived, identity-bound credentials An SSH-style handshake model for cloud native…
