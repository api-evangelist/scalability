---
title: "Kubernetes v1.36: Staleness Mitigation and Observability for Controllers"
url: "https://kubernetes.io/blog/2026/04/28/kubernetes-v1-36-staleness-mitigation-for-controllers/"
date: "Tue, 28 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Staleness in Kubernetes controllers is a problem that affects many controllers, and is something may affect controller behavior in subtle ways. It is usually not until it is too late, when a controller in production has already taken incorrect action, that staleness is found to be an issue due to some underlying assumption made by the controller author. Some issues caused by staleness include controllers taking incorrect actions, controllers not taking action when they should, and controllers taking too long to take action.
