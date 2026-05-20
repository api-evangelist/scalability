---
title: "Kubernetes v1.35: A Better Way to Pass Service Account Tokens to CSI Drivers"
url: "https://kubernetes.io/blog/2026/01/07/kubernetes-v1-35-csi-sa-tokens-secrets-field-beta/"
date: "Wed, 07 Jan 2026 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
If you maintain a CSI driver that uses service account tokens, Kubernetes v1.35 brings a refinement you'll want to know about. Since the introduction of the TokenRequests feature , service account tokens requested by CSI drivers have been passed to them through the volume_context field. While this has worked, it's not the ideal place for sensitive information, and we've seen instances where tokens were accidentally logged in CSI drivers.
