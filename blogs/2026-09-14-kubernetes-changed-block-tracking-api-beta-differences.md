---
title: "Kubernetes Changed Block Tracking API - Beta Differences"
url: "https://kubernetes.io/blog/2026/09/14/csi-changed-block-tracking-beta/"
date: "2026-09-14"
feed_url: "https://kubernetes.io/feed.xml"
---
Changed Block Tracking (CBT) support for CSI drivers shipped as Alpha in September 2025. With the March 2026 v1.0.0 release of the external-snapshot-metadata project, the feature moved to Beta . If you aren't yet familiar with changed block tracking for storage in Kubernetes, the Alpha announcement covers the motivation, the three primary components (the CSI SnapshotMetadata gRPC service, the SnapshotMetadataService CRD, and the external-snapshot-metadata sidecar), and a walkthrough of how to use the API.
