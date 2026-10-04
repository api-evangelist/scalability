---
title: "Kubernetes v1.37: Tracking When a PersistentVolumeClaim Was Last Used (Beta)"
url: "https://kubernetes.io/blog/2026/09/21/kubernetes-v1-37-pvc-last-used-time/"
date: "2026-09-21"
feed_url: "https://kubernetes.io/feed.xml"
---
Kubernetes v1.37 promotes the PersistentVolumeClaimUnusedSinceTime feature gate to Beta (enabled by default). With this feature, the PersistentVolumeClaim (PVC) protection controller adds an Unused condition to each PVC, telling you whether any running pod currently references it — no custom tooling or cross-referencing required. For the API definition of PVC conditions, see the PersistentVolumeClaim API reference .
