---
title: "How CAST AI uses KEDA for Kubernetes autoscaling"
url: "https://keda.sh/blog/2021-08-04-keda-cast-ai/"
date: "Wed, 04 Aug 2021 00:00:00 +0000"
author: ""
feed_url: "https://keda.sh/blog/index.xml"
---
How CAST AI uses KEDA for Kubernetes autoscaling Kubernetes comes with several built-in autoscaling mechanisms - among them the Horizontal Pod Autoscaler (HPA). Scaling is essential for the producer-consumer workflow, a common use case in the IT world today. It’s especially useful for monthly reports and transactions with a huge load where teams need to spin up many workloads to process things faster and cheaper (for example, by using spot instances).
