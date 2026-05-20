---
title: "Cluster API v1.12: Introducing In-place Updates and Chained Upgrades"
url: "https://kubernetes.io/blog/2026/01/27/cluster-api-v1-12-release/"
date: "Tue, 27 Jan 2026 08:00:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Cluster API brings declarative management to Kubernetes cluster lifecycle, allowing users and platform teams to define the desired state of clusters and rely on controllers to continuously reconcile toward it. Similar to how you can use StatefulSets or Deployments in Kubernetes to manage a group of Pods, in Cluster API you can use KubeadmControlPlane to manage a set of control plane Machines, or you can use MachineDeployments to manage a group of worker Nodes. The Cluster API v1.12.0 release expands what is possible in Cluster API, reducing friction in common lifecycle operations by…
