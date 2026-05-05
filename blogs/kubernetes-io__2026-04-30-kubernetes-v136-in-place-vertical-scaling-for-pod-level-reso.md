---
title: "Kubernetes v1.36: In-Place Vertical Scaling for Pod-Level Resources Graduates to Beta"
url: "https://kubernetes.io/blog/2026/04/30/kubernetes-v1-36-inplace-pod-level-resources-beta/"
date: "Thu, 30 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>Following the graduation of Pod-Level Resources to Beta in v1.34 and the General Availability (GA) of In-Place Pod Vertical Scaling in v1.35, the Kubernetes community is thrilled to announce that <strong>In-Place Pod-Level Resources Vertical Scaling has graduated to Beta in v1.36!</strong></p>
<p>This feature is now enabled by default via the <code>InPlacePodLevelResourcesVerticalScaling</code> feature gate. It allows users to update the aggregate Pod resource budget (<code>.spec.resources</code>) for a running Pod, <strong>often without requiring a container restart.</strong></p>
<h2 id="why-pod-level-in-place-resize">Why Pod-level in-place resize?</h2>
<p>The Pod-level resource model simplified management for complex Pods (such as those with sidecars) by allowing containers to share a collective pool of resources. In v1.36, you can now adjust this aggregate boundary on-the-fly.</p>
<p>This is particularly useful for Pods where containers do not have individual limits defined. These containers automatically scale their effective boundaries to fit the newly resized Pod-level dimensions, allowing you to expand the shared pool during peak demand without manual per-container recalculations.</p>
<h2 id="resource-inheritance-and-the-resizepolicy">Resource inheritance and the <code>resizePolicy</code></h2>
<p>When a Pod-level resize is initiated, the Kubelet treats the change as a resize event for every container that inherits its limits from the Pod-level budget. To determine whether a restart is required, the Kubelet consults the <code>resizePolicy</code> defined within individual containers:</p>
<ul>
<li><strong>Non-disruptive Updates:</strong> If a container's <code>restartPolicy</code> is set to <code>NotRequired</code>, the Kubelet attempts to update the cgroup limits dynamically via the Container Runtime Interface (CRI).</li>
<li><strong>Disruptive Updates:</strong> If set to <code>RestartContainer</code>, the container will be restarted to apply the new aggregate boundary safely.</li>
</ul>
<blockquote>
<p><strong>Note:</strong> Currently, <code>resizePolicy</code> is not supported at the Pod level. The Kubelet always defers to individual container settings to decide if an update can be applied in-place or requires a restart.</p>
</blockquote>
<h2 id="example-scaling-a-shared-resource-pool">Example: Scaling a shared resource pool</h2>
<p>In this scenario, a Pod is defined with a 2 CPU pod-level limit. Because the individual containers do not have their own limits defined, they share this total pool.</p>
<h3 id="1-initial-pod-specification">1. Initial Pod specification</h3>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Pod<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>shared-pool-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># Pod-level limits</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"2"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">containers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>main-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>my-app:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resizePolicy</span>:<span style="color: #bbb;"> </span>[{<span style="color: #008000; font-weight: bold;">resourceName</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"cpu"</span><span style="color: #008000; font-weight: bold;">, restartPolicy</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"NotRequired"</span>}]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>sidecar<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>logger:v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resizePolicy</span>:<span style="color: #bbb;"> </span>[{<span style="color: #008000; font-weight: bold;">resourceName</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"cpu"</span><span style="color: #008000; font-weight: bold;">, restartPolicy</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"NotRequired"</span>}]<span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="2-the-resize-operation">2. The resize operation</h3>
<p>To double the CPU capacity to 4 CPUs, apply a patch using the <code>resize</code> subresource:</p>
<div class="highlight"><pre tabindex="0"><code class="language-bash"><span style="display: flex;"><span>kubectl patch pod shared-pool-app --subresource resize --patch <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> <span style="color: #b44;">'{"spec":{"resources":{"limits":{"cpu":"4"}}}}'</span>
</span></span></code></pre></div><h2 id="node-level-reality-feasibility-and-safety">Node-Level reality: feasibility and safety</h2>
<p>Applying a resize patch is only the first step. The Kubelet performs several checks and follows a specific sequence to ensure node stability:</p>
<h3 id="1-the-feasibility-check">1. The feasibility check</h3>
<p>Before admitting a resize, the Kubelet verifies if the new aggregate request fits within the Node's allocatable capacity. If the Node is overcommitted, the resize is not ignored; instead, the <code>PodResizePending</code> condition will reflect a <code>Deferred</code> or <code>Infeasible</code> status, providing immediate feedback on why the &quot;envelope&quot; hasn't grown.</p>
<h3 id="2-update-sequencing">2. Update sequencing</h3>
<p>To prevent resource &quot;overshoot&quot;, the Kubelet coordinates the cgroup updates in a specific order:</p>
<ul>
<li><strong>When Increasing:</strong> The Pod-level cgroup is expanded first, creating the &quot;room&quot; before the individual container cgroups are enlarged.</li>
<li><strong>When Decreasing:</strong> The container cgroups are throttled first, and only then is the aggregate Pod-level cgroup shrunken.</li>
</ul>
<h2 id="observability-tracking-resize-status">Observability: tracking resize status</h2>
<p>With the move to Beta, Kubernetes uses <strong>Pod Conditions</strong> to track the lifecycle of a resize:</p>
<ul>
<li><strong><code>PodResizePending</code></strong>: The spec is updated, but the Node hasn't admitted the change (e.g., due to capacity).</li>
<li><strong><code>PodResizeInProgress</code></strong>: The Node has admitted the resize (<code>status.allocatedResources</code>) but the changes aren't yet fully applied to the cgroups (<code>status.resources</code>).</li>
</ul>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">status</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allocatedResources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">conditions</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>PodResizeInProgress<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">status</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"True"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h2 id="constraints-and-requirements">Constraints and requirements</h2>
<ul>
<li><strong>cgroup v2 Only:</strong> Required for accurate aggregate enforcement.</li>
<li><strong>CRI Support:</strong> Requires a container runtime that supports the <code>UpdateContainerResources</code> CRI call (e.g., containerd v2.0+ or CRI-O).</li>
<li><strong>Feature Gates:</strong> Requires <code>PodLevelResources</code>, <code>InPlacePodVerticalScaling</code>, <code>InPlacePodLevelResourcesVerticalScaling</code>, and <code>NodeDeclaredFeatures</code>.</li>
<li><strong>Linux Only:</strong> Currently exclusive to Linux-based nodes.</li>
</ul>
<h2 id="what-s-next">What's next?</h2>
<p>As we move toward General Availability (GA), the community is focusing on <strong>Vertical Pod Autoscaler (VPA) Integration</strong>, enabling VPA to issue Pod-level resource recommendations and trigger in-place actuation automatically.</p>
<h2 id="getting-started-and-providing-feedback">Getting started and providing feedback</h2>
<p>We encourage you to test this feature and provide feedback via the standard Kubernetes communication channels:</p>
<ul>
<li>Slack: <a href="https://kubernetes.slack.com/messages/sig-node">#sig-node</a></li>
<li><a href="https://groups.google.com/forum/#!forum/kubernetes-sig-node">Mailing list</a></li>
<li><a href="https://github.com/kubernetes/community/labels/sig%2Fnode">Open Community Issues/PRs</a></li>
</ul>
