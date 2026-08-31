---
title: "Kubernetes v1.37: Pod Certificates and Cluster Trust Bundles"
url: "https://kubernetes.io/blog/2026/08/28/kubernetes-v1-37-pod-certificates-and-cluster-trust-bundles/"
date: "2026-08-28"
feed_url: "https://kubernetes.io/feed.xml"
---
Pod Certificate / Cluster Trust Bundles Blog Post Kubernetes brings a wealth of features that make it easy to run your production workloads securely and reliably. While aspects like scheduling, health checks and resource limits are probably at the front of your mind, one other important feature of Kubernetes is production identity — how your workload can authenticate to other systems in order to do its job. Up until now, the primary production identity mechanism built into Kubernetes has been service account JWTs (JSON Web Tokens).
