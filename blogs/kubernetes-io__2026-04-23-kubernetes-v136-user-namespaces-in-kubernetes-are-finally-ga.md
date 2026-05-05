---
title: "Kubernetes v1.36: User Namespaces in Kubernetes are finally GA"
url: "https://kubernetes.io/blog/2026/04/23/kubernetes-v1-36-userns-ga/"
date: "Thu, 23 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>After several years of development, User Namespaces support in
Kubernetes reached General Availability (GA) with the v1.36 release.
This is a Linux-only feature.</p>
<p>For those of us working on low level container runtimes and rootless
technologies, this has been a long awaited milestone. We finally
reached the point where &quot;rootless&quot; security isolation can be used for
Kubernetes workloads.</p>
<p>This feature also enables a critical pattern: running workloads with
privileges and still being confined in the user namespace. When
<code>hostUsers: false</code> is set, capabilities like <code>CAP_NET_ADMIN</code> become
<strong>namespaced</strong>, meaning they grant administrative power over container
local resources without affecting the host. This effectively enables
new use cases that were not possible before without running a fully
privileged container.</p>
<h2 id="the-problem-with-uid-0">The Problem with UID 0</h2>
<p>A process running as root inside a container is also seen from the
kernel as root on the host. If an attacker manages to break out of
the container, whether through a kernel vulnerability or a
misconfigured mount, they are root on the host.</p>
<p>While there are many security measures in place for running
containers, these measures don't change the underlying identity of the
process, it still has some &quot;parts&quot; of root.</p>
<h2 id="the-engine-id-mapped-mounts">The engine: ID-mapped mounts</h2>
<p>The road to GA wasn't just about the Kubernetes API; it was about
making the kernel work for us. In the early stages, one of the
biggest blockers was volume ownership. If you mapped a container to a
high UID range, the Kubelet had to recursively <code>chown</code> every file in
the attached volume so the container could read/write them. For large
volumes, this was such an expensive operation that destroyed startup
performance.</p>
<p>The key enabler was <em>ID-mapped mounts</em> (introduced in Linux
5.12 and refined in later versions). Instead of rewriting file
ownership on disk, the kernel remaps it at mount time.</p>
<p>When a volume is mounted into a Pod with User Namespaces enabled, the
kernel performs a transparent translation of the UIDs (user ids) and
GIDs (group ids). To the container, the files appear owned by
UID 0. On disk, file ownership is unchanged — no <code>chown</code> is needed.
This is an <code>O(1)</code> operation, instant and efficient.</p>
<h2 id="using-it-in-kubernetes-v1-36">Using it in Kubernetes v1.36</h2>
<p>Using user namespaces is straightforward: all you need to do is set
<code>hostUsers: false</code> in your Pod spec. No changes to your container
images, no complex configuration. The interface remains the same one
introduced during the Alpha phase. In the <code>spec</code> for a Pod (or PodTemplate), you explicitly
opt-out of the host user namespace:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Pod<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>isolated-workload<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostUsers</span>:<span style="color: #bbb;"> </span><span style="color: #a2f; font-weight: bold;">false</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">containers</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">image</span>:<span style="color: #bbb;"> </span>fedora:42<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">securityContext</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">runAsUser</span>:<span style="color: #bbb;"> </span><span style="color: #666;">0</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>For more details on how user namespaces work in practice and demos of
CVEs rated HIGH mitigated, see the previous blog posts:
<a href="https://kubernetes.io/blog/2022/10/03/userns-alpha/">User Namespaces alpha</a>,
<a href="https://kubernetes.io/blog/2023/09/13/userns-alpha/">User Namespaces stateful pods in alpha</a>,
<a href="https://kubernetes.io/blog/2024/04/22/userns-beta/">User Namespaces beta</a>, and
<a href="https://kubernetes.io/blog/2025/04/25/userns-enabled-by-default/">User Namespaces enabled by default</a>.</p>
<h2 id="getting-involved">Getting involved</h2>
<p>If you're interested in user namespaces or want to contribute, here
are some useful links:</p>
<ul>
<li><a href="https://kubernetes.io/docs/concepts/workloads/pods/user-namespaces/">User Namespaces documentation</a></li>
<li><a href="https://kep.k8s.io/127">KEP-127: Support User Namespaces</a></li>
<li><a href="https://github.com/kubernetes/community/tree/master/sig-node">SIG Node</a></li>
</ul>
<h2 id="acknowledgments">Acknowledgments</h2>
<p>This feature has been years in the making: the first KEP was opened
10 years ago by other contributors, and we have been actively working
on it for the last 6 years. We'd like to thank everyone who
contributed across SIG Node, the container runtimes, and the Linux
kernel. Special thanks to the reviewers and early adopters who helped
shape the design through multiple alpha and beta cycles.</p>
