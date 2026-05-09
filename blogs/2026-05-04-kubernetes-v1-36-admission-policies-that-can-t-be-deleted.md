---
title: "Kubernetes v1.36: Admission Policies That Can't Be Deleted"
url: "https://kubernetes.io/blog/2026/05/04/kubernetes-v1-36-manifest-based-admission-control/"
date: "2026-05-04"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
If you've ever tried to enforce a security policy across a fleet of Kubernetes clusters, you've probably run into a frustrating chicken-and-egg problem. Your admission policies are API objects, which means they don't exist until someone creates them, and they can be deleted by anyone with the right permissions. There's always a window during cluster bootstrap where your policies aren't active yet, and there's no way to prevent a privileged user from removing them. Kubernetes v1.36 introduces an alpha feature that addresses this: manifest-based admission control.
