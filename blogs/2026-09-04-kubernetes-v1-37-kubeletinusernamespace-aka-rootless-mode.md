---
title: "Kubernetes v1.37: KubeletInUserNamespace (aka Rootless mode) Graduates to Beta"
url: "https://kubernetes.io/blog/2026/09/04/kubernetes-v1-37-rootless-beta/"
date: "2026-09-04"
feed_url: "https://kubernetes.io/feed.xml"
---
Kubernetes v1.37 promotes the KubeletInUserNamespace feature gate to beta. With this feature enabled, all of the node components (kubelet, CRI and OCI runtimes, CNI plugins, and kube-proxy) can run as a non-root user on the host, using a Linux user namespace . This technique is also known as rootless mode .
