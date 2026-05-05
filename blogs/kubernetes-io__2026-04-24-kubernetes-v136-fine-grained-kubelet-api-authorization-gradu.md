---
title: "Kubernetes v1.36: Fine-Grained Kubelet API Authorization Graduates to GA"
url: "https://kubernetes.io/blog/2026/04/24/kubernetes-v1-36-fine-grained-kubelet-authorization-ga/"
date: "Fri, 24 Apr 2026 10:35:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p>On behalf of Kubernetes SIG Auth and SIG Node, we are pleased to announce the
graduation of fine-grained <code>kubelet</code> API authorization to General Availability
(GA) in Kubernetes v1.36!</p>
<p>The <code>KubeletFineGrainedAuthz</code> feature gate was introduced as an opt-in alpha
feature in Kubernetes v1.32, then graduated to beta (enabled by default) in
v1.33. Now, the feature is generally available and the feature gate is locked
to enabled. This feature enables more precise, least-privilege access control
over the <code>kubelet</code>'s HTTPS API, replacing the need to grant the overly broad
<code>nodes/proxy</code> permission for common monitoring and observability use cases.</p>
<h2 id="motivation-the-nodes-proxy-problem">Motivation: the <code>nodes/proxy</code> problem</h2>
<p>The <code>kubelet</code> exposes an HTTPS endpoint with several APIs that give access to data
of varying sensitivity, including pod listings, node metrics, container logs,
and, critically, the ability to execute commands inside running containers.</p>
<p>Prior to this feature, <code>kubelet</code> authorization used a coarse-grained model. When
webhook authorization was enabled, almost all <code>kubelet</code> API paths were mapped to a
single <code>nodes/proxy</code> subresource. This meant that any workload needing to read
metrics or health status from the <code>kubelet</code> required <code>nodes/proxy</code> permission,
the same permission that also grants the ability to execute arbitrary commands
in any container running on the node.</p>
<h3 id="what-s-wrong-with-that">What's wrong with that?</h3>
<p>Granting <code>nodes/proxy</code> to monitoring agents, log collectors, or health-checking
tools violates the principle of least privilege. If any of those workloads were
compromised, an attacker would gain the ability to run commands in every
container on the node. The <code>nodes/proxy</code> permission is effectively a node-level
superuser capability, and granting it broadly dramatically increases the blast
radius of a security incident.</p>
<p>This problem has been well understood in the community for years (see
<a href="https://github.com/kubernetes/kubernetes/issues/83465">kubernetes/kubernetes#83465</a>),
and was the driving motivation behind this enhancement <a href="https://kep.k8s.io/2862">KEP-2862</a>.</p>
<h3 id="the-nodes-proxy-get-websocket-rce-risk">The <code>nodes/proxy GET</code> WebSocket RCE risk</h3>
<p>The situation is more severe than it might appear at first glance. Security
researchers <a href="https://grahamhelton.com/blog/nodes-proxy-rce">demonstrated in early 2026</a>
that <code>nodes/proxy GET</code> alone, which is the minimal read-only permission routinely
granted to monitoring tools, can be abused to execute commands in any pod on
reachable nodes.</p>
<p>The root cause is a mismatch between how WebSocket connections work and how the
<code>kubelet</code> maps HTTP methods to RBAC verbs. The
<a href="https://datatracker.ietf.org/doc/html/rfc6455#section-1.2">WebSocket protocol (RFC 6455)</a>
requires an HTTP <code>GET</code> request for the initial connection handshake. The <code>kubelet</code>
maps this <code>GET</code> to the RBAC <code>get</code> verb and authorizes the request without
performing a secondary check to confirm that <code>CREATE</code> permission is also present
for the write operation that follows. Using a WebSocket client like <code>websocat</code>,
an attacker can reach the <code>kubelet</code>'s <code>/exec</code> endpoint directly on port 10250 and
execute arbitrary commands:</p>
<div class="highlight"><pre tabindex="0"><code class="language-bash"><span style="display: flex;"><span>websocat --insecure <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --header <span style="color: #b44;">"Authorization: Bearer </span><span style="color: #b8860b;">$TOKEN</span><span style="color: #b44;">"</span> <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --protocol v4.channel.k8s.io <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> <span style="color: #b44;">"wss://</span><span style="color: #b8860b;">$NODE_IP</span><span style="color: #b44;">:10250/exec/default/nginx/nginx?output=1&amp;error=1&amp;command=id"</span>
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span><span style="color: #b8860b;">uid</span><span style="color: #666;">=</span>0<span style="color: #666;">(</span>root<span style="color: #666;">)</span> <span style="color: #b8860b;">gid</span><span style="color: #666;">=</span>0<span style="color: #666;">(</span>root<span style="color: #666;">)</span> <span style="color: #b8860b;">groups</span><span style="color: #666;">=</span>0<span style="color: #666;">(</span>root<span style="color: #666;">)</span>
</span></span></code></pre></div><h2 id="fine-grained-kubelet-authorization-how-it-works">Fine-grained <code>kubelet</code> authorization: how it works</h2>
<p>With <code>KubeletFineGrainedAuthz</code>, the <code>kubelet</code> now performs an additional, more
specific authorization check before falling back to the <code>nodes/proxy</code>
subresource. Several commonly used <code>kubelet</code> API paths are mapped to their own
dedicated subresources:</p>
<table>
<thead>
<tr>
<th><code>kubelet</code> API</th>
<th>Resource</th>
<th>Subresource</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/stats/*</code></td>
<td>nodes</td>
<td>stats</td>
</tr>
<tr>
<td><code>/metrics/*</code></td>
<td>nodes</td>
<td>metrics</td>
</tr>
<tr>
<td><code>/logs/*</code></td>
<td>nodes</td>
<td>log</td>
</tr>
<tr>
<td><code>/pods</code></td>
<td>nodes</td>
<td>pods, proxy</td>
</tr>
<tr>
<td><code>/runningPods/</code></td>
<td>nodes</td>
<td>pods, proxy</td>
</tr>
<tr>
<td><code>/healthz</code></td>
<td>nodes</td>
<td>healthz, proxy</td>
</tr>
<tr>
<td><code>/configz</code></td>
<td>nodes</td>
<td>configz, proxy</td>
</tr>
<tr>
<td><code>/spec/*</code></td>
<td>nodes</td>
<td>spec</td>
</tr>
<tr>
<td><code>/checkpoint/*</code></td>
<td>nodes</td>
<td>checkpoint</td>
</tr>
<tr>
<td>all others</td>
<td>nodes</td>
<td>proxy</td>
</tr>
</tbody>
</table>
<p>For the endpoints that now have fine-grained subresources (<code>/pods</code>,
<code>/runningPods/</code>, <code>/healthz</code>, <code>/configz</code>), the <code>kubelet</code> first sends a
<code>SubjectAccessReview</code> for the specific subresource. If that check succeeds, the
request is authorized. If it fails, the <code>kubelet</code> retries with the coarse-grained
<code>nodes/proxy</code> subresource for backward compatibility.</p>
<p>This dual-check approach ensures a smooth migration path. Existing workloads
with <code>nodes/proxy</code> permissions continue to work, while new deployments can adopt
least-privilege access from day one.</p>
<h2 id="what-this-means-in-practice">What this means in practice</h2>
<p>Consider a Prometheus node exporter or a monitoring <code>DaemonSet</code> that needs to
scrape <code>/metrics</code> from the <code>kubelet</code>. Previously, you would need an RBAC
<code>ClusterRole</code> like this:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># Old approach: overly broad</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>rbac.authorization.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ClusterRole<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>monitoring-agent<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span>- <span style="color: #008000; font-weight: bold;">apiGroups</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">""</span>]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">"nodes/proxy"</span>]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">verbs</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">"get"</span>]<span style="color: #bbb;">
</span></span></span></code></pre></div><p>This grants the monitoring agent far more access than it needs. With
fine-grained authorization, you can now scope the permissions precisely:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># New approach: least privilege</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>rbac.authorization.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ClusterRole<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>monitoring-agent<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span>- <span style="color: #008000; font-weight: bold;">apiGroups</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">""</span>]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">"nodes/metrics"</span>,<span style="color: #bbb;"> </span><span style="color: #b44;">"nodes/stats"</span>]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">verbs</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">"get"</span>]<span style="color: #bbb;">
</span></span></span></code></pre></div><p>The monitoring agent can now read metrics and stats from the <code>kubelet</code> without
ever being able to execute commands in containers.</p>
<h2 id="updated-system-kubelet-api-admin-clusterrole">Updated <code>system:kubelet-api-admin</code> <code>ClusterRole</code></h2>
<p>When RBAC authorization is enabled, the built-in <code>system:kubelet-api-admin</code>
<code>ClusterRole</code> is automatically updated to include permissions for all the new
fine-grained subresources. This ensures that cluster administrators who already
use this role, including the API server's <code>kubelet</code> client, continue to have
full access without any manual configuration changes.</p>
<p>The role now includes permissions for:</p>
<ul>
<li><code>nodes/proxy</code></li>
<li><code>nodes/stats</code></li>
<li><code>nodes/metrics</code></li>
<li><code>nodes/log</code></li>
<li><code>nodes/spec</code></li>
<li><code>nodes/checkpoint</code></li>
<li><code>nodes/configz</code></li>
<li><code>nodes/healthz</code></li>
<li><code>nodes/pods</code></li>
</ul>
<h2 id="upgrade-considerations">Upgrade considerations</h2>
<p>Because the <code>kubelet</code> performs a dual authorization check (fine-grained first,
then falling back to <code>nodes/proxy</code>), upgrading to v1.36 should be seamless for
most clusters:</p>
<ul>
<li><strong>Existing workloads</strong> with <code>nodes/proxy</code> permissions continue to work without
changes. The fallback to <code>nodes/proxy</code> ensures backward compatibility.</li>
<li><strong>The API server</strong> always has <code>nodes/proxy</code> permissions via
<code>system:kubelet-api-admin</code>, so <code>kube-apiserver</code>-to-<code>kubelet</code> communication is
unaffected regardless of feature gate state.</li>
<li><strong>Mixed-version clusters</strong> are handled gracefully. If a <code>kubelet</code> supports
fine-grained authorization but the API server does not (or vice versa),
<code>nodes/proxy</code> permissions serve as the fallback.</li>
</ul>
<h2 id="verifying-the-feature-is-enabled">Verifying the feature is enabled</h2>
<p>You can confirm that the feature is active on a given node by checking the
<code>kubelet</code> metrics endpoint. Since the metrics endpoint on port 10250 requires
authorization, you'll first need to create appropriate RBAC bindings for the pod
or <code>ServiceAccount</code> making the request.</p>
<p><strong>Step 1: Create a <code>ServiceAccount</code> and <code>ClusterRole</code></strong></p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ServiceAccount<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>kubelet-metrics-checker<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>default<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>rbac.authorization.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ClusterRole<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>kubelet-metrics-reader<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span>- <span style="color: #008000; font-weight: bold;">apiGroups</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">""</span>]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">resources</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">"nodes/metrics"</span>]<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">verbs</span>:<span style="color: #bbb;"> </span>[<span style="color: #b44;">"get"</span>]<span style="color: #bbb;">
</span></span></span></code></pre></div><p><strong>Step 2: Bind the <code>ClusterRole</code> to the <code>ServiceAccount</code></strong></p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>rbac.authorization.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ClusterRoleBinding<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>kubelet-metrics-checker<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">subjects</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span>- <span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ServiceAccount<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>kubelet-metrics-checker<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>default<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">roleRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ClusterRole<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>kubelet-metrics-reader<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">apiGroup</span>:<span style="color: #bbb;"> </span>rbac.authorization.k8s.io<span style="color: #bbb;">
</span></span></span></code></pre></div><p>Apply both manifests:</p>
<div class="highlight"><pre tabindex="0"><code class="language-bash"><span style="display: flex;"><span>kubectl apply -f serviceaccount.yaml
</span></span><span style="display: flex;"><span>kubectl apply -f clusterrole.yaml
</span></span><span style="display: flex;"><span>kubectl apply -f clusterrolebinding.yaml
</span></span></code></pre></div><p><strong>Step 3: Run a pod with the <code>ServiceAccount</code> and check the feature flag</strong></p>
<div class="highlight"><pre tabindex="0"><code class="language-bash"><span style="display: flex;"><span>kubectl run kubelet-check <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --image<span style="color: #666;">=</span>curlimages/curl <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --serviceaccount<span style="color: #666;">=</span>kubelet-metrics-checker <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --restart<span style="color: #666;">=</span>Never <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --rm -it <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> -- sh
</span></span></code></pre></div><p>Then from within the pod, retrieve the node IP and query the metrics endpoint:</p>
<div class="highlight"><pre tabindex="0"><code class="language-bash"><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># Get the token</span>
</span></span><span style="display: flex;"><span><span style="color: #b8860b;">TOKEN</span><span style="color: #666;">=</span><span style="color: #a2f; font-weight: bold;">$(</span>cat /var/run/secrets/kubernetes.io/serviceaccount/token<span style="color: #a2f; font-weight: bold;">)</span>
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span><span style="color: #080; font-style: italic;"># Query the kubelet metrics and filter for the feature gate</span>
</span></span><span style="display: flex;"><span>curl -sk <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> --header <span style="color: #b44;">"Authorization: Bearer </span><span style="color: #b8860b;">$TOKEN</span><span style="color: #b44;">"</span> <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> https://<span style="color: #b8860b;">$NODE_IP</span>:10250/metrics <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> | grep kubernetes_feature_enabled <span style="color: #b62; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #b62; font-weight: bold;"></span> | grep KubeletFineGrainedAuthz
</span></span></code></pre></div><p>If the feature is enabled, you should see output like:</p>
<pre tabindex="0"><code>kubernetes_feature_enabled{name="KubeletFineGrainedAuthz",stage="GA"} 1
</code></pre><blockquote>
<p><strong>Note:</strong> Replace <code>$NODE_IP</code> with the IP address of the node you want to check.
You can retrieve node IPs with <code>kubectl get nodes -o wide</code>.</p>
</blockquote>
<h2 id="the-journey-from-alpha-to-ga">The journey from alpha to GA</h2>
<table>
<thead>
<tr>
<th>Release</th>
<th>Stage</th>
<th>Details</th>
</tr>
</thead>
<tbody>
<tr>
<td>v1.32</td>
<td>Alpha</td>
<td>Feature gate <code>KubeletFineGrainedAuthz</code> introduced, disabled by default</td>
</tr>
<tr>
<td>v1.33</td>
<td>Beta</td>
<td>Enabled by default; fine-grained checks for <code>/pods</code>, <code>/runningPods/</code>, <code>/healthz</code>, <code>/configz</code></td>
</tr>
<tr>
<td>v1.36</td>
<td>GA</td>
<td>Feature gate locked to enabled; fine-grained <code>kubelet</code> authorization is always active</td>
</tr>
</tbody>
</table>
<h2 id="what-s-next">What's next?</h2>
<p>With fine-grained <code>kubelet</code> authorization now GA, the Kubernetes community can
begin recommending and eventually enforcing the use of specific subresources
instead of <code>nodes/proxy</code> for monitoring and observability workloads. The urgency
of this migration is underscored by
<a href="https://grahamhelton.com/blog/nodes-proxy-rce">research showing that <code>nodes/proxy GET</code> can be abused for unlogged remote code execution</a> via the WebSocket protocol. This risk is present in the default RBAC
configurations of dozens of widely deployed Helm charts. Over time, we expect:</p>
<ul>
<li><strong>Ecosystem adoption:</strong> Monitoring tools like Prometheus, Datadog agents, and
other <code>DaemonSets</code> can update their default RBAC configurations to use
<code>nodes/metrics</code>, <code>nodes/stats</code>, and <code>nodes/pods</code> instead of <code>nodes/proxy</code>. This
directly eliminates the WebSocket RCE attack surface for those workloads.</li>
<li><strong>Policy enforcement:</strong> Admission controllers and policy engines can flag or
reject RBAC bindings that grant <code>nodes/proxy</code> when fine-grained alternatives
exist, helping organizations adopt least-privilege access at scale.</li>
<li><strong>Deprecation path:</strong> As adoption grows, <code>nodes/proxy</code> may eventually be
deprecated for monitoring use cases, further reducing the attack surface of
Kubernetes clusters.</li>
</ul>
<h2 id="getting-involved">Getting involved</h2>
<p>This enhancement was driven by SIG Auth and SIG Node. If you are interested in
contributing to the security and authorization features of Kubernetes, please
join us:</p>
<ul>
<li><a href="https://github.com/kubernetes/community/tree/master/sig-auth">SIG Auth</a></li>
<li><a href="https://github.com/kubernetes/community/tree/master/sig-node">SIG Node</a></li>
<li>Slack: <code>#sig-auth</code> and <code>#sig-node</code></li>
<li><a href="https://github.com/kubernetes/enhancements/issues/2862">KEP-2862: Fine-Grained Kubelet API Authorization</a></li>
</ul>
<p>We look forward to hearing your feedback and experiences with this feature!</p>
