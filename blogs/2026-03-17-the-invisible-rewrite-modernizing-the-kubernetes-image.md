---
title: "The Invisible Rewrite: Modernizing the Kubernetes Image Promoter"
url: "https://kubernetes.io/blog/2026/03/17/image-promoter-rewrite/"
date: "Tue, 17 Mar 2026 00:00:00 +0000"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Every container image you pull from registry.k8s.io got there through kpromo , the Kubernetes image promoter. It copies images from staging registries to production, signs them with cosign , replicates signatures across more than 20 regional mirrors, and generates SLSA provenance attestations. If this tool breaks, no Kubernetes release ships.
