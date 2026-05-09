---
title: "Kubernetes v1.36: Moving Volume Group Snapshots to GA"
url: "https://kubernetes.io/blog/2026/05/08/kubernetes-v1-36-volume-group-snapshot-ga/"
date: "2026-05-08"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Volume group snapshots were introduced as an Alpha feature with the Kubernetes v1.27 release, moved to Beta in v1.32, and to a second Beta in v1.34. We are excited to announce that in the Kubernetes v1.36 release, support for volume group snapshots has reached General Availability (GA). The support for volume group snapshots relies on a set of extension APIs for group snapshots. These APIs allow users to take crash-consistent snapshots for a set of volumes. Behind the scenes, Kubernetes uses a label selector to group multiple PersistentVolumeClaim objects for snapshotting.
