---
title: "Kubernetes v1.35: Watch Based Route Reconciliation in the Cloud Controller Manager"
url: "https://kubernetes.io/blog/2025/12/30/kubernetes-v1-35-watch-based-route-reconciliation-in-ccm/"
date: "Tue, 30 Dec 2025 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Up to and including Kubernetes v1.34, the route controller in Cloud Controller Manager (CCM) implementations built using the k8s.io/cloud-provider library reconciles routes at a fixed interval. This causes unnecessary API requests to the cloud provider when there are no changes to routes. Other controllers implemented through the same library already use watch-based mechanisms, leveraging informers to avoid unnecessary API calls.
