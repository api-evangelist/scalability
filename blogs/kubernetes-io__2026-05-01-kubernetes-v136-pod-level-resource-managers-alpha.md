---
title: "Kubernetes v1.36: Pod-Level Resource Managers (Alpha)"
url: "https://kubernetes.io/blog/2026/05/01/kubernetes-v1-36-feature-pod-level-resource-managers-alpha/"
date: "Fri, 01 May 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>Kubernetes v1.36 introduces
<a href="https://kubernetes.io/docs/concepts/workloads/resource-managers/#pod-level-resource-managers">Pod-Level Resource Managers</a>
as an alpha feature, bringing a more flexible and powerful resource management
model to performance-sensitive workloads. This enhancement extends the kubelet's
Topology, CPU, and Memory Managers to support pod-level resource specifications
(<code>.spec.resources</code>), evolving them from a strictly per-container allocation
model to a pod-centric one.</p>
<h2 id="why-do-we-need-pod-level-resource-managers">Why do we need pod-level resource managers?</h2>
<p>When running performance-critical workloads such as machine learning (ML)
training, high-frequency trading applications, or low-latency databases, you
often need exclusive, NUMA-aligned resources for your primary application
containers to ensure predictable performance.</p>
<p>However, modern Kubernetes pods rarely consist of just one container. They
frequently include sidecar containers for logging, monitoring, service meshes,
or data ingestion.</p>
<p>Before this feature, this created a trade-off, to get NUMA-aligned, exclusive
resources for your main application, you had to allocate exclusive,
integer-based CPU resources to <em>every</em> container in the pod. This might be
wasteful for lightweight sidecars. If you didn't do this, you forfeited the
pod's Guaranteed Quality of Service (QoS) class entirely, losing the performance
benefits.</p>
<h2 id="introducing-pod-level-resource-managers">Introducing pod-level resource managers</h2>
<p>Enabling pod-level resources support for the resource managers (via the
<code>PodLevelResourceManagers</code> and <code>PodLevelResources</code> feature gates) allows the
kubelet to create <strong>hybrid resource allocation models</strong>. This brings flexibility
and efficiency to high-performance workloads without sacrificing NUMA alignment.</p>
<h3 id="real-world-use-cases">Real-world use cases</h3>
<p>Here are a few practical scenarios demonstrating how this feature can be
applied, depending on the configured Topology Manager scope:</p>
<h4 id="1-tightly-coupled-database-topology-manager-s-pod-scope">1. Tightly-coupled database (Topology manager's pod scope)</h4>
<p>Consider a latency-sensitive database pod that includes a main database
container, a local metrics exporter, and a backup agent sidecar.</p>
<p>When configured with the <code>pod</code> Topology Manager scope, the kubelet performs a
single NUMA alignment based on the entire pod's budget. The database container
gets its exclusive CPU and memory slices from that NUMA node. The remaining
resources from the pod's budget form a new <strong>pod shared pool</strong>. The metrics
exporter and backup agent run in this pod shared pool. They share resources with
each other, but they are strictly isolated from the database's exclusive slices
and the rest of the node.</p>
<p>This allows you to safely co-locate auxiliary containers on the same NUMA node
as your primary workload without wasting dedicated cores on them.</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Pod<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>tightly-coupled-database<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># Pod-level resources establish the overall budget and NUMA alignment size.</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">requests</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"8"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"16Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"8"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"16Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">initContainers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>metrics-exporter<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>metrics-exporter:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">restartPolicy</span>:<span style="color: #bbb;"> </span>Always<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>backup-agent<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>backup-agent:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">restartPolicy</span>:<span style="color: #bbb;"> </span>Always<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">containers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>database<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>database:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># This Guaranteed container gets an exclusive 6 CPU slice from the pod's budget.</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># The remaining 2 CPUs and 4Gi memory form the pod shared pool for the sidecars.</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">requests</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"6"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"12Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"6"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"12Gi"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h4 id="2-ml-workload-with-infrastructure-sidecars-topology-manager-s-container-scope">2. ML workload with infrastructure sidecars (Topology manager's container scope)</h4>
<p>Imagine a pod running a GPU-accelerated ML training workload alongside a generic
service mesh sidecar.</p>
<p>Under the <code>container</code> Topology Manager scope, the kubelet evaluates each
container individually. You can grant the ML container exclusive, NUMA-aligned
CPUs and Memory for maximum performance. Meanwhile, the service mesh sidecar
doesn't need to be NUMA-aligned; it can run in the general node-wide shared
pool. The collective resource consumption is still safely bounded by the overall
pod limits, but you only allocate NUMA-aligned, exclusive resources to the
specific containers that actually require them.</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Pod<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>ml-workload<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># Pod-level resources establish the overall budget constraint.</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">requests</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"8Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"8Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">initContainers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>service-mesh-sidecar<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>service-mesh:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">restartPolicy</span>:<span style="color: #bbb;"> </span>Always<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">containers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>ml-training<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>ml-training:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># Under the 'container' scope, this Guaranteed container receives exclusive,</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># NUMA-aligned resources, while the sidecar runs in the node's shared pool.</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">requests</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"3"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"6Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"3"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"6Gi"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="cpu-quotas-cfs-and-isolation">CPU quotas (CFS) and isolation</h3>
<p>When running these mixed workloads within a pod, isolation is enforced
differently depending on the allocation:</p>
<ul>
<li><strong>Exclusive containers:</strong> Containers granted exclusive CPU slices have their
CPU CFS quota enforcement disabled at the container level, allowing them to
run without being throttled by the Linux scheduler.</li>
<li><strong>Pod shared pool containers:</strong> Containers falling into the pod shared pool
have CPU CFS quotas enforced at the pod level, ensuring they do not consume
more than the leftover pod budget.</li>
</ul>
<h2 id="how-to-enable-pod-level-resource-managers">How to enable Pod-Level Resource Managers</h2>
<p>Using this feature requires Kubernetes v1.36 or newer. To enable it, you must
configure the kubelet with the appropriate feature gates and policies:</p>
<ol>
<li>Enable the <code>PodLevelResources</code> and <code>PodLevelResourceManagers</code>
<a href="https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/">feature gates</a>.</li>
<li>Configure the
<a href="https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/#topology-manager-policies">Topology Manager</a>
with a policy other than <code>none</code> (i.e., <code>best-effort</code>, <code>restricted</code>, or
<code>single-numa-node</code>).</li>
<li>Set the
<a href="https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/#topology-manager-scopes">Topology Manager scope</a>
to either <code>pod</code> or <code>container</code> using the <code>topologyManagerScope</code> field in the
<a href="https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/"><code>KubeletConfiguration</code></a>.</li>
<li>Configure the
<a href="https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/">CPU Manager</a> with
the <code>static</code> policy.</li>
<li>Configure the
<a href="https://kubernetes.io/docs/tasks/administer-cluster/memory-manager/">Memory Manager</a> with the
<code>Static</code> policy.</li>
</ol>
<h2 id="observability">Observability</h2>
<p>To help cluster administrators monitor and debug these new allocation models, we
have introduced several new kubelet metrics when the feature gate is enabled:</p>
<ul>
<li><code>resource_manager_allocations_total</code>: Counts the total number of exclusive
resource allocations performed by a manager. The <code>source</code> label (&quot;pod&quot; or
&quot;node&quot;) distinguishes between allocations drawn from the node-level pool
versus a pre-allocated pod-level pool.</li>
<li><code>resource_manager_allocation_errors_total</code>: Counts errors encountered during
exclusive resource allocation, distinguished by the intended allocation
<code>source</code> (&quot;pod&quot; or &quot;node&quot;).</li>
<li><code>resource_manager_container_assignments</code>: Tracks the cumulative number of
containers running with specific assignment types. The <code>assignment_type</code>
label (&quot;node_exclusive&quot;, &quot;pod_exclusive&quot;, &quot;pod_shared&quot;) provides visibility
into how workloads are distributed.</li>
</ul>
<h2 id="current-limitations-and-caveats">Current limitations and caveats</h2>
<p>While this feature opens up new possibilities, there are a few things to keep in
mind during its alpha phase. Be sure to review the
<a href="https://kubernetes.io/docs/concepts/workloads/resource-managers/#limitations-and-caveats">Limitations and caveats</a>
in the official documentation for full details on compatibility, requirements,
and downgrade instructions.</p>
<h2 id="getting-started-and-providing-feedback">Getting started and providing feedback</h2>
<p>For a deep dive into the technical details and configuration of this feature,
check out the official concept documentation:</p>
<ul>
<li><a href="https://kubernetes.io/docs/concepts/workloads/resource-managers/#pod-level-resource-managers">Pod-level resource managers</a></li>
</ul>
<p>To learn more about the overall pod-level resources feature and how to assign
resources to pods, see:</p>
<ul>
<li><a href="https://kubernetes.io/docs/tasks/configure-pod-container/assign-pod-level-resources/">Assign Pod-level CPU and memory resources</a></li>
</ul>
<p>As this feature moves through Alpha, your feedback is invaluable. Please report
any issues or share your experiences via the standard Kubernetes communication
channels:</p>
<ul>
<li>Slack: <a href="https://kubernetes.slack.com/messages/sig-node">#sig-node</a></li>
<li><a href="https://groups.google.com/forum/#!forum/kubernetes-sig-node">Mailing list</a></li>
<li><a href="https://github.com/kubernetes/community/labels/sig%2Fnode">Open Community Issues/PRs</a></li>
</ul>
