---
title: "Kubernetes v1.36: Tiered Memory Protection with Memory QoS"
url: "https://kubernetes.io/blog/2026/04/29/kubernetes-v1-36-memory-qos-tiered-protection/"
date: "Wed, 29 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>On behalf of SIG Node, we are pleased to announce updates to the Memory QoS
feature (alpha) in Kubernetes v1.36. Memory QoS uses the cgroup v2 memory
controller to give the kernel better guidance on how to treat container memory.
It was first introduced in v1.22 and updated in v1.27. In Kubernetes v1.36, we're introducing: opt-in memory reservation, tiered
protection by QoS class, observability metrics, and kernel-version warning for <code>memory.high</code>.</p>
<h2 id="what-s-new-in-v1-36">What's new in v1.36</h2>
<h3 id="opt-in-memory-reservation-with-memoryreservationpolicy">Opt-in memory reservation with <code>memoryReservationPolicy</code></h3>
<p>v1.36 separates throttling from reservation. Enabling the feature gate turns on
<code>memory.high</code> throttling (the kubelet sets <code>memory.high</code> based on
<code>memoryThrottlingFactor</code>, default 0.9), but memory reservation is now controlled
by a separate kubelet configuration field:</p>
<ul>
<li><strong><code>None</code></strong> (default): no <code>memory.min</code> or <code>memory.low</code> is written. Throttling
via <code>memory.high</code> still works.</li>
<li><strong><code>TieredReservation</code></strong>: the kubelet writes tiered memory protection based on the Pod's
<a href="https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/">QoS class</a>:</li>
</ul>
<p><strong>Guaranteed</strong> Pods get hard protection via <code>memory.min</code>. For example, a
Guaranteed Pod requesting 512 MiB of memory results in:</p>
<pre tabindex="0"><code class="language-none">$ cat /sys/fs/cgroup/kubepods.slice/kubepods-pod6a4f2e3b_1c9d_4a5e_8f7b_2d3e4f5a6b7c.slice/memory.min
536870912
</code></pre><p>The kernel will not reclaim this memory under any circumstances. If it cannot
honor the guarantee, it invokes the OOM killer on other processes to free pages.</p>
<p><strong>Burstable</strong> Pods get soft protection via <code>memory.low</code>. For the same 512 MiB
request on a Burstable Pod:</p>
<pre tabindex="0"><code class="language-none">$ cat /sys/fs/cgroup/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod8b3c7d2e_4f5a_6b7c_9d1e_3f4a5b6c7d8e.slice/memory.low
536870912
</code></pre><p>The kernel avoids reclaiming this memory under normal pressure, but may reclaim
it if the alternative is a system-wide OOM.</p>
<p><strong>BestEffort</strong> Pods get neither <code>memory.min</code> nor <code>memory.low</code>. Their memory
remains fully reclaimable.</p>
<h4 id="comparison-with-v1-27-behavior">Comparison with v1.27 behavior</h4>
<p>In earlier versions, enabling the MemoryQoS feature gate immediately set <code>memory.min</code> for every container with a memory request. <code>memory.min</code> is a hard reservation that the kernel will not reclaim, regardless of memory pressure.</p>
<p>Consider a node with 8 GiB of RAM where Burstable Pod requests total 7 GiB. In earlier versions, that 7 GiB would be locked as <code>memory.min</code>, leaving little headroom for the kernel, system daemons, or BestEffort workloads and increasing the risk of OOM kills.</p>
<p>With v1.36 tiered reservation, those Burstable requests map to <code>memory.low</code> instead of <code>memory.min</code>. Under normal pressure, the kernel still protects that memory, but under extreme pressure it can reclaim part of it to avoid system-wide OOM. Only Guaranteed Pods use <code>memory.min</code>, which keeps hard reservation lower.</p>
<p>With <code>memoryReservationPolicy</code> in v1.36, you can enable throttling first, observe workload behavior, and opt into reservation when your node has enough headroom.</p>
<h3 id="observability-metrics">Observability metrics</h3>
<p>Two alpha-stability metrics are exposed on the kubelet <code>/metrics</code> endpoint:</p>
<table>
<thead>
<tr>
<th>Metric</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>kubelet_memory_qos_node_memory_min_bytes</code></td>
<td>Total <code>memory.min</code> across Guaranteed Pods</td>
</tr>
<tr>
<td><code>kubelet_memory_qos_node_memory_low_bytes</code></td>
<td>Total <code>memory.low</code> across Burstable Pods</td>
</tr>
</tbody>
</table>
<p>These are useful for capacity planning. If <code>kubelet_memory_qos_node_memory_min_bytes</code>
is creeping toward your node's physical memory, you know hard reservation is
getting tight.</p>
<pre tabindex="0"><code class="language-none">$ curl -sk https://localhost:10250/metrics | grep memory_qos
# HELP kubelet_memory_qos_node_memory_min_bytes [ALPHA] Total memory.min in bytes for Guaranteed pods
kubelet_memory_qos_node_memory_min_bytes 5.36870912e+08
# HELP kubelet_memory_qos_node_memory_low_bytes [ALPHA] Total memory.low in bytes for Burstable pods
kubelet_memory_qos_node_memory_low_bytes 2.147483648e+09
</code></pre><h3 id="kernel-version-check">Kernel version check</h3>
<p>On kernels older than 5.9, <code>memory.high</code> throttling can trigger the
<a href="https://lore.kernel.org/all/a4e23b59e9ef499b575ae73a8120ee089b7d3373.1594640214.git.chris@chrisdown.name/">kernel livelock</a> issue. The bug was fixed
in kernel 5.9. In v1.36, when the feature gate is enabled, the kubelet checks the
kernel version at startup and logs a warning if it is below 5.9. The feature
continues to work — this is informational, not a hard block.</p>
<h3 id="how-kubernetes-maps-memory-qos-to-cgroup-v2">How Kubernetes maps Memory QoS to cgroup v2</h3>
<p>Memory QoS uses four cgroup v2 memory controller interfaces:</p>
<ul>
<li><strong><code>memory.max</code></strong>: hard memory limit — unchanged from previous versions</li>
<li><strong><code>memory.min</code></strong>: hard memory protection — with <code>TieredReservation</code>, set only for Guaranteed Pods</li>
<li><strong><code>memory.low</code></strong>: soft memory protection — set for Burstable Pods with <code>TieredReservation</code></li>
<li><strong><code>memory.high</code></strong>: memory throttling threshold — unchanged from previous versions</li>
</ul>
<p>The following table shows how Kubernetes container resources map to cgroup v2
interfaces when <code>memoryReservationPolicy: TieredReservation</code> is configured.
With the default <code>memoryReservationPolicy: None</code>, no <code>memory.min</code> or
<code>memory.low</code> values are set.</p>
<table>
<tr>
<th>QoS Class</th>
<th><tt>memory.min</tt></th>
<th><tt>memory.low</tt></th>
<th><tt>memory.high</tt></th>
<th><tt>memory.max</tt></th>
</tr>
<tr>
<td><b>Guaranteed</b></td>
<td>Set to <code>requests.memory</code><br />(hard protection)</td>
<td>Not set</td>
<td>Not set<br />(requests == limits, so throttling is not useful)</td>
<td>Set to <code>limits.memory</code></td>
</tr>
<tr>
<td><b>Burstable</b></td>
<td>Not set</td>
<td>Set to <code>requests.memory</code><br />(soft protection)</td>
<td>Calculated based on<br />formula with throttling factor</td>
<td>Set to <code>limits.memory</code><br />(if specified)</td>
</tr>
<tr>
<td><b>BestEffort</b></td>
<td>Not set</td>
<td>Not set</td>
<td>Calculated based on<br />node allocatable memory</td>
<td>Not set</td>
</tr>
</table>
<h3 id="cgroup-hierarchy">Cgroup hierarchy</h3>
<p>cgroup v2 requires that a parent cgroup's memory protection is at least as
large as the sum of its children's. The kubelet maintains this by setting
<code>memory.min</code> on the kubepods root cgroup to the sum of all Guaranteed and
Burstable Pod memory requests, and <code>memory.low</code> on the Burstable QoS cgroup
to the sum of all Burstable Pod memory requests. This way the kernel can
enforce the per-container and per-pod protection values correctly.</p>
<p>The kubelet manages pod-level and QoS-class cgroups directly using the runc
libcontainer library, while container-level cgroups are managed by the
container runtime (containerd or CRI-O).</p>
<h2 id="how-do-i-use-it">How do I use it?</h2>
<h3 id="prerequisites">Prerequisites</h3>
<ol>
<li>Kubernetes v1.36 or later</li>
<li>Linux with cgroup v2. Kernel 5.9 or higher is recommended — earlier kernels
work but may experience the livelock issue. You can verify cgroup v2 is
active by running <code>mount | grep cgroup2</code>.</li>
<li>A container runtime that supports cgroup v2 (containerd 1.6+, CRI-O 1.22+)</li>
</ol>
<h3 id="configuration">Configuration</h3>
<p>To enable Memory QoS with tiered protection:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>kubelet.config.k8s.io/v1beta1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>KubeletConfiguration<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">featureGates</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">MemoryQoS</span>:<span style="color: #bbb;"> </span><span style="color: #a2f; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">memoryReservationPolicy: TieredReservation # Options</span>:<span style="color: #bbb;"> </span>None (default), TieredReservation<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">memoryThrottlingFactor: 0.9 # Optional</span>:<span style="color: #bbb;"> </span>default is 0.9<span style="color: #bbb;">
</span></span></span></code></pre></div><p>If you want <code>memory.high</code> throttling without memory protection, omit
<code>memoryReservationPolicy</code> or set it to <code>None</code>:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>kubelet.config.k8s.io/v1beta1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>KubeletConfiguration<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">featureGates</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">MemoryQoS</span>:<span style="color: #bbb;"> </span><span style="color: #a2f; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">memoryReservationPolicy</span>:<span style="color: #bbb;"> </span>None <span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># This is the default</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h2 id="how-can-i-learn-more">How can I learn more?</h2>
<ul>
<li><a href="https://kep.k8s.io/2570">KEP-2570: Memory QoS</a></li>
<li><a href="https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/">Pod Quality of Service Classes</a></li>
<li><a href="https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/">Managing Resources for Containers</a></li>
<li><a href="https://kubernetes.io/docs/concepts/architecture/cgroups/">Kubernetes cgroups v2 support</a></li>
<li><a href="https://docs.kernel.org/admin-guide/cgroup-v2.html">Linux kernel cgroups v2 documentation</a></li>
</ul>
<h2 id="getting-involved">Getting involved</h2>
<p>This feature is driven by <a href="https://github.com/kubernetes/community/tree/master/sig-node">SIG Node</a>.
If you are interested in contributing or have feedback, you can find us on
<a href="https://kubernetes.slack.com/messages/sig-node">Slack</a> (#sig-node), the
<a href="https://groups.google.com/forum/#!forum/kubernetes-sig-node">mailing list</a>,
or at the regular
<a href="https://github.com/kubernetes/community/tree/master/sig-node#meetings">SIG Node meetings</a>.
Please file bugs at <a href="https://github.com/kubernetes/kubernetes/issues">kubernetes/kubernetes</a>
and enhancement proposals at
<a href="https://github.com/kubernetes/enhancements/issues/2570">kubernetes/enhancements</a>.</p>
