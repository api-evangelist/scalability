---
title: "Autoscaling Azure Pipelines agents with KEDA"
url: "https://keda.sh/blog/2021-05-27-azure-pipelines-scaler/"
date: "Thu, 27 May 2021 00:00:00 +0000"
author: ""
feed_url: "https://keda.sh/blog/index.xml"
---
With the addition of Azure Piplines support in KEDA, it is now possible to autoscale your Azure Pipelines agents based on the agent pool queue length. Self-hosted Azure Pipelines agents are the perfect workload for this scaler. By autoscaling the agents you can create a scalable CI/CD environment. 💡 The number of concurrent pipelines you can run is limited by your parallel jobs .
