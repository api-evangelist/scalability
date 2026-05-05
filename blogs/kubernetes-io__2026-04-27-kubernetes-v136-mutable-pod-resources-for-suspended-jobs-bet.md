---
title: "Kubernetes v1.36: Mutable Pod Resources for Suspended Jobs (beta)"
url: "https://kubernetes.io/blog/2026/04/27/kubernetes-v1-36-mutable-pod-resources-for-suspended-jobs/"
date: "Mon, 27 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>Kubernetes v1.36 promotes the ability to modify container resource requests and limits
in the pod template of a suspended Job to beta. First introduced as alpha in v1.35, this
feature allows queue controllers and cluster administrators to adjust CPU, memory, GPU,
and extended resource specifications on a Job while it is suspended, before it starts
or resumes running.</p>
<h2 id="why-mutable-pod-resources-for-suspended-jobs">Why mutable pod resources for suspended Jobs?</h2>
<p>Batch and machine learning workloads often have resource requirements that are not
precisely known at Job creation time. The optimal resource allocation depends on
current cluster capacity, queue priorities, and the availability of specialized hardware
like GPUs.</p>
<p>Before this feature, resource requirements in a Job's pod template were immutable once set.
If a queue controller like <a href="https://kueue.sigs.k8s.io/">Kueue</a> determined that a suspended
Job should run with different resources, the only option was to delete and recreate the Job,
losing any associated metadata, status, or history. This feature also provides a way
to let a specific Job instance for a CronJob progress slowly with reduced resources,
rather than outright failing to run if the cluster is heavily loaded.</p>
<p>Consider a machine learning training Job initially requesting 4 GPUs:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>batch/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Job<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>training-job-example-abcd123<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">labels</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">app.kubernetes.io/name</span>:<span style="color: #bbb;"> </span>trainer<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">suspend</span>:<span style="color: #bbb;"> </span><span style="color: #a2f; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">template</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">annotations</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">kubernetes.io/description</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"ML training, ID abcd123"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">containers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>trainer<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>example-registry.example.com/training:2026-04-23T150405.678<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">requests</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"8"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"32Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">example-hardware-vendor.com/gpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"8"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"32Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">example-hardware-vendor.com/gpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">restartPolicy</span>:<span style="color: #bbb;"> </span>Never<span style="color: #bbb;">
</span></span></span></code></pre></div><p>A queue controller managing cluster resources might determine that only 2 GPUs
are available. With this feature, the controller can update the Job's resource
requests before resuming it:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>batch/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Job<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>training-job-example-abcd123<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">labels</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">app.kubernetes.io/name</span>:<span style="color: #bbb;"> </span>trainer<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">suspend</span>:<span style="color: #bbb;"> </span><span style="color: #a2f; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">template</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">annotations</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">kubernetes.io/description</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"ML training, ID abcd123"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">containers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>trainer<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>example-registry.example.com/training:2026-04-23T150405.678<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">requests</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"16Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">example-hardware-vendor.com/gpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"2"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">limits</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">cpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">memory</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"16Gi"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">example-hardware-vendor.com/gpu</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"2"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">restartPolicy</span>:<span style="color: #bbb;"> </span>Never<span style="color: #bbb;">
</span></span></span></code></pre></div><p>Once the resources are updated, the controller resumes the Job by setting
<code>spec.suspend</code> to <code>false</code>, and the new Pods are created with the adjusted
resource specifications.</p>
<h2 id="how-it-works">How it works</h2>
<p>The Kubernetes API server relaxes the immutability constraint on pod template
resource fields specifically for suspended Jobs. No new API types have been introduced;
the existing Job and pod template structures accommodate the change through
relaxed validation.</p>
<p>The mutable fields are:</p>
<ul>
<li><code>spec.template.spec.containers[*].resources.requests</code></li>
<li><code>spec.template.spec.containers[*].resources.limits</code></li>
<li><code>spec.template.spec.initContainers[*].resources.requests</code></li>
<li><code>spec.template.spec.initContainers[*].resources.limits</code></li>
</ul>
<p>Resource updates are permitted when the following conditions are met:</p>
<ol>
<li>The Job has <code>spec.suspend</code> set to <code>true</code>.</li>
<li>For a Job that was previously running and then suspended, all active
Pods must have terminated (<code>status.active</code> equals 0) before resource
mutations are accepted.</li>
</ol>
<p>Standard resource validation still applies. For example, resource limits
must be greater than or equal to requests, and extended resources must be
specified as whole numbers where required.</p>
<h2 id="what-s-new-in-beta">What's new in beta</h2>
<p>With the promotion to beta in Kubernetes v1.36, the
<code>MutablePodResourcesForSuspendedJobs</code> feature gate is enabled by default.
This means clusters running v1.36 can use this feature without any additional
configuration on the API server.</p>
<h2 id="try-it-out">Try it out</h2>
<p>If your cluster is running Kubernetes v1.36 or later, this feature is available
by default. For v1.35 clusters, enable the <code>MutablePodResourcesForSuspendedJobs</code>
<a href="https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/">feature gate</a> on
the <code>kube-apiserver</code>.</p>
<p>You can test it by creating a suspended Job, updating its container resources
using <code>kubectl edit</code> or a controller, and then resuming the Job:</p>
<div class="highlight"><pre tabindex="0"><code class="language-shell"><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># Create a suspended Job</span>
</span></span><span style="display: flex;"><span>kubectl apply -f my-job.yaml --server-side
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># Edit the resource requests</span>
</span></span><span style="display: flex;"><span>kubectl edit job training-job-example-abcd123
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># Resume the Job</span>
</span></span><span style="display: flex;"><span>kubectl patch job training-job-example-abcd123 -p <span style="color: #b44;">'{"spec":{"suspend":false}}'</span>
</span></span></code></pre></div><h2 id="considerations">Considerations</h2>
<h3 id="running-jobs-that-are-suspended">Running Jobs that are suspended</h3>
<p>If you suspend a Job that was already running, you must wait for <strong>all</strong> of that Job's active
Pods to terminate before modifying resources. The API server rejects resource
mutations while <code>status.active</code> is greater than zero. This prevents inconsistency
between running Pods and the updated pod template.</p>
<h3 id="pod-replacement-policy">Pod replacement policy</h3>
<p>When using this feature with Jobs that may have failed Pods, consider setting
<code>podReplacementPolicy: Failed</code>. This ensures that replacement Pods are only
created after the previous Pods have fully terminated, preventing resource
contention from overlapping Pods.</p>
<h3 id="resourceclaims">ResourceClaims</h3>
<p>Dynamic Resource Allocation (DRA) <code>resourceClaimTemplates</code> remain immutable.
If your workload uses DRA, you must recreate the claim templates separately
to match any resource changes.</p>
<h2 id="getting-involved">Getting involved</h2>
<p>This feature was developed by <a href="https://github.com/kubernetes/community/tree/master/sig-apps">SIG Apps</a>
This feature was developed by <a href="https://www.kubernetes.dev/community/community-groups/sigs/apps/">SIG Apps</a>
with input from <a href="https://www.kubernetes.dev/community/community-groups/wg/batch/">WG Batch</a>. Both groups welcome feedback
as the feature progresses toward stable.</p>
<p>You can reach out through:</p>
<ul>
<li>Slack channel <a href="https://kubernetes.slack.com/archives/C18NZM5K9">#sig-apps</a>.</li>
<li>Slack channel <a href="https://kubernetes.slack.com/archives/C032ZE66A2X">#wg-batch</a>.</li>
<li>The <a href="https://kep.k8s.io/5440">KEP-5440</a> tracking issue.</li>
</ul>
