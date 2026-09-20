---
title: "Kubernetes v1.37: Hardening Container Storage with Bind Mount Options and EmptyDir Permissions"
url: "https://kubernetes.io/blog/2026/09/16/kubernetes-v1-37-hardening-container-storage/"
date: "2026-09-16"
feed_url: "https://kubernetes.io/feed.xml"
---
Kubernetes v1.37 brings important storage security features: emptyDir permission modes and bind mount options. They help application programmers and security professionals implement rigorous security policies, for example, prohibiting deletion of files across containers or execution of arbitrary binaries from writable volumes, directly in Kubernetes without any complicated circumvention. Linux storage and permission fundamentals Before diving into the new Kubernetes features, let us briefly review the low-level Linux security mechanisms that make them possible.
