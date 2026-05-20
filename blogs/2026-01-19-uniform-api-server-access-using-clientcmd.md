---
title: "Uniform API server access using clientcmd"
url: "https://kubernetes.io/blog/2026/01/19/clientcmd-apiserver-access/"
date: "Mon, 19 Jan 2026 10:00:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
If you've ever wanted to develop a command line client for a Kubernetes API, especially if you've considered making your client usable as a kubectl plugin, you might have wondered how to make your client feel familiar to users of kubectl . A quick glance at the output of kubectl options might put a damper on that: "Am I really supposed to implement all those options?" Fear not, others have done a lot of the work involved for you. In fact, the Kubernetes project provides two libraries to help you handle kubectl -style command line arguments in Go programs: clientcmd and cli-runtime (which…
