---
title: "Kubernetes v1.35: Restricting executables invoked by kubeconfigs via exec plugin allowList added to kuberc"
url: "https://kubernetes.io/blog/2026/01/09/kubernetes-v1-35-kuberc-credential-plugin-allowlist/"
date: "Fri, 09 Jan 2026 10:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
Did you know that kubectl can run arbitrary executables, including shell scripts, with the full privileges of the invoking user, and without your knowledge? Whenever you download or auto-generate a kubeconfig , the users[n].exec.command field can specify an executable to fetch credentials on your behalf. Don't get me wrong, this is an incredible feature that allows you to authenticate to the cluster with external identity providers.
