---
title: "How Zapier uses KEDA"
url: "https://keda.sh/blog/2022-03-10-how-zapier-uses-keda/"
date: "Thu, 10 Mar 2022 00:00:00 +0000"
author: ""
feed_url: "https://keda.sh/blog/index.xml"
---
RabbitMQ is at the heart of Zap processing at Zapier . We enqueue messages to RabbitMQ for each step in a Zap. These messages get consumed by our backend workers, which run on Kubernetes .
