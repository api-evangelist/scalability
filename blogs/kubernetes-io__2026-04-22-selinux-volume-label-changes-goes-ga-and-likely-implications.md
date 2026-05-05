---
title: "SELinux Volume Label Changes goes GA (and likely implications in v1.37)"
url: "https://kubernetes.io/blog/2026/04/22/breaking-changes-in-selinux-volume-labeling/"
date: "Wed, 22 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>If you run Kubernetes on Linux with SELinux in enforcing mode, plan ahead: a future release (anticipated to be v1.37) is
expected to turn the <code>SELinuxMount</code> <a href="https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/">feature gate</a> on by default. This makes volume setup faster
for most workloads, but <strong>it can break applications</strong> that still depend on the older recursive relabeling
model in subtle ways (for example, sharing one volume between privileged and unprivileged Pods on the same node).
Kubernetes v1.36 is the right release to audit your cluster and fix or opt out of this change.</p>
<p>If your nodes do not use SELinux, nothing changes for you: the kubelet skips the whole
SELinux logic when SELinux is unavailable or disabled in the Linux kernel. You can skip this article completely.</p>
<p>This blog builds on the earlier work described in the
<a href="https://kubernetes.io/blog/2023/04/18/kubernetes-1-27-efficient-selinux-relabeling-beta/">Kubernetes 1.27: Efficient SELinux Relabeling (Beta)</a>
post, where the <code>SELinuxMountReadWriteOncePod</code> feature gate was described. The problem to be addressed remains
the same, however, this blog extends that same approach to all volumes.</p>
<h2 id="the-problem">The problem</h2>
<p>Linux systems with Security Enhanced Linux (SELinux) enabled use labels attached to objects
(for example, files and network sockets) to make access control decisions.
Historically, the container runtime applies SELinux labels to a Pod and all its volumes. Kubernetes only passes the SELinux label from a Pod's <code>securityContext</code> fields
to the container runtime.</p>
<p>The container runtime then recursively changes the SELinux label on all files that
are visible to the Pod's containers. This can be time-consuming if there are
many files on the volume, especially when the volume is on a remote filesystem.</p>
<div class="alert alert-caution"><h4 class="alert-heading">Caution:</h4>If a container uses <code>subPath</code> of a volume, only that <code>subPath</code> of the whole
volume is relabeled. This allows two Pods that have two different SELinux labels
to use the same volume, as long as they use different subpaths of it.</div>
<p>If a Pod does not have any SELinux label assigned in the Kubernetes API, the
container runtime assigns a unique random label, so a process that potentially
escapes the container boundary cannot access data of any other container on the
host. The container runtime still recursively relabels all Pod volumes with this
random SELinux label.</p>
<h2 id="what-kubernetes-is-improving">What Kubernetes is improving</h2>
<p>Where the stack supports it, the kubelet can mount the volume with <code>-o context=&lt;label&gt;</code> so the kernel
applies the correct label for all inodes on that mount without a recursive inode traversal. That path is
gated by feature flags and requires, among other things, that the Pod expose enough of an SELinux
label (for example <code>spec.securityContext.seLinuxOptions.level</code>) and that the volume driver opts in (for CSI,
CSIDriver field <code>spec.seLinuxMount: true</code>).</p>
<p>The project rolled this out in phases:</p>
<ul>
<li>ReadWriteOncePod volumes were handled under the <code>SELinuxMountReadWriteOncePod</code> feature gate, on by default since v1.28 and GA in v1.36.</li>
<li>Broader coverage was handled under the <code>SELinuxMount</code> flag, paired with the <code>spec.securityContext.seLinuxChangePolicy</code> field on Pods.</li>
</ul>
<!-- a heavily edited copy from the previous blog + docs in https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ -->
<p>If a Pod and its volume meet <strong>all</strong> of the following conditions, Kubernetes will
mount the volume directly with the right SELinux label. Such a mount will happen
in a constant time and the container runtime will not need to recursively
relabel any files on it. For such a mount to happen:</p>
<ol>
<li>
<p>The operating system must support SELinux. Without SELinux support detected, the kubelet and the container runtime do not
do anything with regard to SELinux.</p>
</li>
<li>
<p>The <a href="https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/">feature gate</a>
<code>SELinuxMountReadWriteOncePod</code> must be enabled.
If you're running Kubernetes v1.36, the feature is enabled unconditionally.</p>
</li>
<li>
<p>The Pod must use a PersistentVolumeClaim with applicable <code>accessModes</code>:</p>
<ul>
<li>Either the volume has <code>accessModes: [&quot;ReadWriteOncePod&quot;]</code></li>
<li>or the volume can use any other access mode(s), provided that the feature gates
<code>SELinuxChangePolicy</code> and <code>SELinuxMount</code> are both enabled
<strong>and</strong> the Pod has <code>spec.securityContext.seLinuxChangePolicy</code> set to nil (default) or as <code>MountOption</code>.</li>
</ul>
<p>The feature gate <code>SELinuxMount</code> is Beta and disabled by default in Kubernetes 1.36.
All other SELinux-related feature gates are now General Availability (GA).</p>
<p>With any of these feature gates disabled, SELinux labels will always be
applied by the container runtime via recursively traversing through the volume
(or its subPaths).</p>
</li>
<li>
<p>The Pod must have at least <code>seLinuxOptions.level</code> assigned in its
<a href="https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context">security context</a>
or all containers in that Pod must have it set in their container-level <a href="https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context-1">security contexts</a>.
Kubernetes will read the default <code>user</code>, <code>role</code> and <code>type</code> from the operating
system defaults (typically <code>system_u</code>, <code>system_r</code> and <code>container_t</code>).</p>
<p>Without Kubernetes knowing at least the SELinux <code>level</code>, the container
runtime will assign a random level after the volumes are mounted. The
container runtime will still relabel the volumes recursively in that case.</p>
</li>
<li>
<p>The volume plugin or the CSI driver responsible for the volume supports
mounting with SELinux mount options.</p>
<p>These in-tree volume plugins support mounting with SELinux mount options:
<code>fc</code> and <code>iscsi</code>.</p>
<p>CSI drivers that support mounting with SELinux mount options must declare this capability in their
<a href="https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/csi-driver-v1/">CSIDriver</a>
instance by setting the <code>seLinuxMount</code> field.</p>
<p>Volumes managed by other volume plugins or CSI drivers that do not
set <code>seLinuxMount: true</code> will be recursively relabeled by the container
runtime.</p>
</li>
</ol>
<h2 id="the-breaking-change">The breaking change</h2>
<p>The <code>SELinuxMount</code> feature gate changes what volumes can be shared among multiple Pods in a subtle way.</p>
<p>Both of these cases work with recursive relabeling:</p>
<ol>
<li>Two Pods with different SELinux labels share the same volume, but each of them uses a different <code>subPath</code> to the volume.</li>
<li>A privileged Pod and an unprivileged Pod share the same volume.</li>
</ol>
<p>The above scenarios will not work with modern, target behavior for Kubernetes mounting when SELinux is active. Instead, one of these Pods will be stuck in <code>ContainerCreating</code> until the other Pod is terminated.</p>
<p>The first case is very niche and hasn't been seen in practice.
Although the second case is still quite rare, this setup has been observed in applications.
Kubernetes v1.36 offers metrics and events to identify these Pods and allows cluster administrators to opt out of the
mount option through the Pod field <code>spec.securityContext.seLinuxChangePolicy</code>.</p>
<h3 id="selinuxchangepolicy"><code>seLinuxChangePolicy</code></h3>
<p>The new Pod field <code>spec.securityContext.seLinuxChangePolicy</code> specifies how the SELinux label is applied to all Pod volumes.
In Kubernetes v1.36, this field is part of the stable Pod API.</p>
<p>There are three choices available:</p>
<dl>
<dt><em>field not set</em> (default)</dt>
<dd>In Kubernetes v1.36, the behavior depends on whether the <code>SELinuxMount</code> feature gate is enabled. By default that feature gate is not enabled, and the SELinux label is applied recursively. If you enable that feature gate in your cluster, and <a href="#what-kubernetes-is-improving">all other conditions</a> are met, labelling will be applied using the mount option.</dd>
<dt><code>Recursive</code></dt>
<dd>the SELinux label is applied recursively. This opts out from using the mount option.</dd>
<dt><code>MountOption</code></dt>
<dd>the SELinux label is applied using the mount option, if <a href="#what-kubernetes-is-improving">all other conditions</a> are met.
This choice is available only when the <code>SELinuxMount</code> feature gate is enabled.</dd>
</dl>
<h2 id="selinux-warning-controller">SELinux warning controller (optional)</h2>
<p>Kubernetes v1.36 provides a new controller within the control plane, <code>selinux-warning-controller</code>.
This controller runs within the kube-controller-manager controller.
To use it, you pass <code>--controllers=*,selinux-warning-controller</code> on the kube-controller-manager command line;
you also must not have explicitly overridden the <code>SELinuxChangePolicy</code> feature gate to be disabled.</p>
<p>The controller watches all Pods in the cluster and emits an Event when it finds two Pods that share the same
volume in a way that is not compatible with the <code>SELinuxMount</code> feature gate.
All such conflicting Pods will receive an event, such as:</p>
<div class="highlight"><pre tabindex="0"><code class="language-console"><span style="display: flex;"><span><span style="color: #888;">SELinuxLabel "system_u:system_r:container_t:s0:c98,c99" conflicts with pod my-other-pod that uses the same volume as this pod with SELinuxLabel "system_u:system_r:container_t:s0:c0,c1". If both pods land on the same node, only one of them may access the volume.
</span></span></span></code></pre></div><p>The actual Pod name may be censored when the conflicting Pods run in different namespaces to prevent leaking information across namespace boundaries.</p>
<p>The controller reports such an event even when these Pods don't run on the same node, to make sure all Pods work
regardless of the Kubernetes scheduler decision. They could run on the same node next time.</p>
<p>In addition, the controller emits the metric <code>selinux_warning_controller_selinux_volume_conflict</code> that lists all current conflicts among Pods.
The metric has labels that identify the conflicting Pods and their SELinux labels, such as:</p>
<pre tabindex="0"><code>selinux_warning_controller_selinux_volume_conflict{pod1_name="my-other-pod",pod1_namespace="default",pod1_value="system_u:object_r:container_file_t:s0:c0,c1",pod2_name="my-pod",pod2_namespace="default",pod2_value="system_u:object_r:container_file_t:s0:c0,c2",property="SELinuxLabel"} 1
</code></pre><p>There is a security consequence from enabling this opt-in controller: it may reveal namespace names, which are always present in the metric.
The Kubernetes project assumes only cluster administrators can access kube-controller-manager metrics.</p>
<h2 id="suggested-upgrade-path">Suggested upgrade path</h2>
<p>To ensure a smooth upgrade path from v1.36 to a release with <code>SELinuxMount</code> enabled (anticipated to be v1.37), we suggest you follow these steps:</p>
<ol>
<li>Enable selinux-warning-controller in the kube-controller-manager.</li>
<li>Check the <code>selinux_warning_controller_selinux_volume_conflict</code> metric. It shows all <em>potential</em> conflicts between Pods.
For each conflicting Pod (Deployment, StatefulSet, etc.), either apply the opt-out (set Pod's <code>spec.securityContext.seLinuxChangePolicy: Recursive</code>)
or re-architect the application to remove such a conflict. For example, do your Pods really need to run as privileged?</li>
<li>Check the <code>volume_manager_selinux_volume_context_mismatch_warnings_total</code> metric. This metric is emitted by the kubelet when it actually
starts a Pod that runs when <code>SELinuxMount</code> is disabled, but such a Pod won't start when <code>SELinuxMount</code> is enabled.
This metric lists the number of Pods that will experience a true conflict. Unfortunately, this metric does not expose the exact Pod name as a label.
The full Pod name is available only in the <code>selinux_warning_controller_selinux_volume_conflict</code> metric.</li>
<li>Once both metrics have been accounted for, upgrade to a Kubernetes version that has <code>SELinuxMount</code> enabled.</li>
</ol>
<p>Consider using a <a href="https://kubernetes.io/docs/reference/access-authn-authz/mutating-admission-policy/">MutatingAdmissionPolicy</a>,
a <a href="https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks_">mutating webhook</a>,
or a policy engine like <a href="https://github.com/kyverno/kyverno/">Kyverno</a> or <a href="https://github.com/open-policy-agent/gatekeeper">Gatekeeper</a>
to apply the opt-out to all Pods in a namespace or across the entire cluster.</p>
<p>When <code>SELinuxMount</code> is enabled, the kubelet will emit the metric <code>volume_manager_selinux_volume_context_mismatch_errors_total</code> with the number of
Pods that could not start because their SELinux label conflicts with an existing Pod that uses the same volume.
The exact Pod names should still be available in the <code>selinux_warning_controller_selinux_volume_conflict</code> metric,
if the selinux-warning-controller is enabled.</p>
<h2 id="further-reading">Further reading</h2>
<ul>
<li>KEP: <a href="https://kep.k8s.io/1710">Speed up SELinux volume relabeling using mounts</a></li>
<li><a href="https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#feature-gates">SELinux Volume Relabeling Feature Gates</a></li>
<li><a href="https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/1710-selinux-relabeling#story-3-cluster-upgrade">Story 3: cluster upgrade</a></li>
<li><a href="https://kubernetes.io/docs/tasks/configure-pod-container/security-context/">Configure a security context for a Pod</a> — Efficient SELinux volume relabeling and selinux-warning-controller</li>
</ul>
<h2 id="acknowledgements">Acknowledgements</h2>
<p>If you run into issues, have feedback, or want to contribute, find us
on the Kubernetes Slack in <code>#sig-node</code> and <code>#sig-storage</code> or join a
<a href="https://github.com/kubernetes/community/tree/main/sig-node">SIG Node</a> or <a href="https://github.com/kubernetes/community/tree/main/sig-storage">SIG Storage</a> meetings.</p>
