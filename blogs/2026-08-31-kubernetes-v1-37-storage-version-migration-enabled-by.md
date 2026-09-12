---
title: "Kubernetes v1.37: Storage Version Migration Enabled by Default"
url: "https://kubernetes.io/blog/2026/08/31/kubernetes-v1-37-storage-version-migration-ga/"
date: "2026-08-31"
feed_url: "https://kubernetes.io/feed.xml"
---
I am excited that storage version migration (SVM) has graduated to General Availability (GA) in Kubernetes v1.37! After a number of releases of work and testing, the built-in StorageVersionMigration API ( storagemigration.k8s.io/v1 ) and control plane controller are now fully stable and enabled by default across all v1.37 Kubernetes clusters. The problem with stale storage versions In Kubernetes, stored API resources are written using a specific storage version (schema representation).
