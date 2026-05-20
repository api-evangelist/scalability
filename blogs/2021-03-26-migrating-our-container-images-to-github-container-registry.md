---
title: "Migrating our container images to GitHub Container Registry"
url: "https://keda.sh/blog/2021-03-26-migrating-to-github-container-registry/"
date: "Fri, 26 Mar 2021 00:00:00 +0000"
author: ""
feed_url: "https://keda.sh/blog/index.xml"
---
We provide various ways to deploy KEDA in your cluster including by using Helm chart , Operator Hub and raw YAML specifications. These deployment options all rely on the container images that we provide which are available on Docker Hub , the industry standard for public container images . However, we have found that Docker Hub is no longer the best place for our container images and are migrating to GitHub Container Registry (Preview).
