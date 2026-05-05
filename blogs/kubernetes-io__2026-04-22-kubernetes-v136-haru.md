---
title: "Kubernetes v1.36: ハル (Haru)"
url: "https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/"
date: "Wed, 22 Apr 2026 00:00:00 +0000"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p><strong>Editors:</strong> Chad M. Crowell, Kirti Goyal, Sophia Ugochukwu, Swathi Rao, Utkarsh Umre</p>
<p>Similar to previous releases, the release of Kubernetes v1.36 introduces new stable, beta, and alpha features. The consistent delivery of high-quality releases underscores the strength of our development cycle and the vibrant support from our community.</p>
<p>This release consists of 70 enhancements. Of those enhancements, 18 have graduated to Stable, 25 are entering Beta, and 25 have graduated to Alpha.</p>
<p>There are also some <a href="#deprecations-removals-and-community-updates">deprecations and removals</a> in this release; make sure to read about those.</p>
<h2 id="release-theme-and-logo">Release theme and logo</h2>
<figure class="release-logo ">
<img alt="Kubernetes v1.36 Haru logo: a hex badge with the title Haru in flowing script beneath v1.36; Mount Fuji rises on the right, its peak lit red with streaks of pale snow, the Japanese calligraphy 晴れに翔け brushed down its slope; a white Kubernetes helm floats in the blue sky to the left among stylised clouds in the ukiyo-e manner; in the foreground stand two cats as paired guardians, a grey-and-white cat on the left and a ginger tabby on the right, each wearing a collar with a small blue Kubernetes helm charm" src="https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/k8s-v1.36.svg" />
</figure>
<p>We open 2026 with Kubernetes v1.36, a release that arrives as the season turns and the light shifts on the mountain. ハル (<em>Haru</em>) is a sound in Japanese that carries many meanings; among those we hold closest are 春 (spring), 晴れ (<em>hare</em>, clear skies), and 遥か (<em>haruka</em>, far-off, distant). A season, a sky, and a horizon. You will find all three in what follows.</p>
<p>The logo, created by <a href="https://x.com/avocadoneko">avocadoneko / Natsuho Ide</a>, draws inspiration from <a href="https://en.wikipedia.org/wiki/Hokusai">Katsushika Hokusai</a>'s <a href="https://en.wikipedia.org/wiki/Thirty-six_Views_of_Mount_Fuji"><em>Thirty-six Views of Mount Fuji</em></a> (富嶽三十六景, <em>Fugaku Sanjūrokkei</em>), the same series that gave the world <a href="https://en.wikipedia.org/wiki/The_Great_Wave_off_Kanagawa"><em>The Great Wave off Kanagawa</em></a>. Our v1.36 logo reimagines one of the series' most celebrated prints, <a href="https://en.wikipedia.org/wiki/Fine_Wind,_Clear_Morning"><em>Fine Wind, Clear Morning</em></a> (凱風快晴, <em>Gaifū Kaisei</em>), also known as Red Fuji (赤富士, <em>Aka Fuji</em>): the mountain lit red in a summer dawn, bare of snow after the long thaw. Thirty-six views felt like a fitting number to sit with at v1.36, and a reminder that even Hokusai didn't stop there.<sup>1</sup> Keeping watch over the scene is the Kubernetes helm, set into the sky alongside the mountain.</p>
<p>At the foot of Fuji sit Stella (left) and Nacho (right), two cats with the Kubernetes helm on their collars, standing in for the role of <a href="https://en.wikipedia.org/wiki/Komainu"><em>komainu</em></a>, the paired lion-dog guardians that watch over Japanese shrines. Paired, because nothing is guarded alone. Stella and Nacho stand in for a very much larger set of paws: the SIGs and working groups, the maintainers and reviewers, the people behind docs, blogs, and translations, the release team, first-time contributors taking their first steps, and lifelong contributors returning season after season. Kubernetes v1.36 is, as always, held up by many hands.</p>
<p>Brushed across Red Fuji in the logo is the calligraphy 晴れに翔け (<em>hare ni kake</em>), &quot;soar into clear skies&quot;. It is the first half of a couplet that was too long to fit on the mountain:</p>
<blockquote>
<p><strong>晴れに翔け、未来よ明け</strong><br />
<em>hare ni kake, asu yo ake</em><br />
&quot;Soar into clear skies; toward tomorrow's sunrise.&quot;<sup>2</sup></p>
</blockquote>
<p>That is the wish we carry for this release: to soar into clear skies, for the release itself, for the project, and for everyone who ships it together. The dawn breaking over Red Fuji is not an ending but a passage: this release carries us to the next, and that one to the one after, on toward horizons far beyond what any single view can hold.</p>
<p><sub>1. The series was so popular that Hokusai added ten more prints, bringing the total to forty-six.</sub><br />
<sub>2. 未来 means &quot;the future&quot; in its widest sense, not just tomorrow but everything still to come. It is usually read <em>mirai</em>; here it takes the informal reading <em>asu</em>.</sub></p>
<h2 id="spotlight-on-key-updates">Spotlight on key updates</h2>
<p>Kubernetes v1.36 is packed with new features and improvements. Here are a
few select updates the Release Team would like to highlight!</p>
<h3 id="stable-fine-grained-api-authorization">Stable: Fine-grained API authorization</h3>
<p>On behalf of Kubernetes SIG Auth and SIG Node, we are pleased to announce the
graduation of fine-grained <code>kubelet</code> API authorization to General Availability
(GA) in Kubernetes v1.36!</p>
<p>The <code>KubeletFineGrainedAuthz</code> feature gate was introduced as an opt-in alpha feature
in Kubernetes v1.32, then graduated to beta (enabled by default) in v1.33.
Now, the feature is generally available.
This feature enables more precise, least-privilege access control over the kubelet's
HTTPS API replacing the need to grant the overly broad nodes/proxy permission for
common monitoring and observability use cases.</p>
<p>​​This work was done as a part of <a href="https://kep.k8s.io/2862">KEP #2862</a> led by SIG Auth and SIG Node.</p>
<h3 id="beta-resource-health-status">Beta: Resource health status</h3>
<p>Before the v1.34 release, Kubernetes lacked a native way to report the health of allocated devices,
making it difficult to diagnose Pod crashes caused by hardware failures.
Building on the initial alpha release in v1.31 which focused on Device Plugins,
Kubernetes v1.36 expands this feature by promoting the <code>allocatedResourcesStatus</code>
field within the <code>.status</code> for each Pod (to beta). This field provides a unified health
reporting mechanism for all specialized hardware.</p>
<p>Users can now run <code>kubectl describe pod</code> to determine if a container's crash loop is
due to an <code>Unhealthy</code> or <code>Unknown</code> device status, regardless of whether the hardware was
provisioned via traditional plugins or the newer DRA framework.
This enhanced visibility allows administrators and automated controllers to
quickly identify faulty hardware and streamline the recovery of high-performance workloads.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/4680">KEP #4680</a> led by SIG Node.</p>
<h3 id="alpha-workload-aware-scheduling-was-features">Alpha: Workload Aware Scheduling (WAS) features</h3>
<p>Previously, the Kubernetes scheduler and job controllers managed pods as independent units,
often leading to fragmented scheduling or resource waste for complex, distributed workloads.
Kubernetes v1.36 introduces a comprehensive suite of Workload Aware Scheduling (WAS) features in Alpha,
natively integrating the Job controller with a revised <a href="https://kubernetes.io/docs/concepts/workloads/workload-api/">Workload</a>
API and a new decoupled PodGroup API,
to treat related pods as a single logical entity.</p>
<p>Kubernetes v1.35 already supported <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/gang-scheduling/">gang scheduling</a> by requiring
a minimum number of pods to be schedulable before any were bound to nodes.
v1.36 goes further with a new PodGroup scheduling cycle that evaluates the entire group atomically,
either all pods in the group are bound together, or none are.</p>
<p>This work was done across several KEPs (including <a href="https://kep.k8s.io/4671">#4671</a>, <a href="https://kep.k8s.io/5547">#5547</a>, <a href="https://kep.k8s.io/5832">#5832</a>, <a href="https://kep.k8s.io/5732">#5732</a>, and <a href="https://kep.k8s.io/5710">#5710</a>) led by SIG Scheduling and SIG Apps.</p>
<h2 id="features-graduating-to-stable">Features graduating to Stable</h2>
<p><em>This is a selection of some of the improvements that are now stable following the v1.36 release.</em></p>
<h3 id="volume-group-snapshots">Volume group snapshots</h3>
<p>After several cycles in beta, VolumeGroupSnapshot support reaches General Availability (GA) in Kubernetes v1.36.
This feature allows you to take crash-consistent snapshots across multiple PersistentVolumeClaims simultaneously.
The support for volume group snapshots relies on a set of <a href="https://kubernetes-csi.github.io/docs/group-snapshot-restore-feature.html#volume-group-snapshot-apis">extension APIs for group snapshots</a>.
These APIs allow users to take crash consistent snapshots for a set of volumes.
A key aim is to allow you to restore that set of snapshots to new volumes and recover your workload based on
a crash consistent recovery point.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/3476">KEP #3476</a> led by SIG Storage.</p>
<h3 id="mutable-volume-attach-limits">Mutable volume attach limits</h3>
<p>In Kubernetes v1.36, the <em>mutable <code>CSINode</code> allocatable</em> feature graduates to stable.
This enhancement allows <a href="https://kubernetes-csi.github.io/docs/introduction.html">Container Storage Interface (CSI)</a> drivers to
dynamically update the reported maximum number of volumes that a node can handle.</p>
<p>With this update, the <code>kubelet</code> can dynamically update a node's volume limits and capacity information.
The <code>kubelet</code> adjusts these limits based on periodic checks or in response to
resource exhaustion errors from the CSI driver, without requiring a component restart.
This ensures the Kubernetes scheduler maintains an accurate view of storage availability,
preventing pod scheduling failures caused by outdated volume limits.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/4876">KEP #4876</a> led by SIG Storage.</p>
<h3 id="api-for-external-signing-of-service-account-tokens">API for external signing of ServiceAccount tokens</h3>
<p>In Kubernetes v1.36, the <em>external ServiceAccount token signer</em> feature for service accounts graduates to stable,
making it possible to offload token signing to an external system while still integrating cleanly with the Kubernetes API.
Clusters can now rely on an external JWT signer for issuing projected service account tokens that
follow the standard service account token format, including support for extended expiration when needed.
This is especially useful for clusters that already rely on external identity or key management systems,
allowing Kubernetes to integrate without duplicating key management inside the control plane.</p>
<p>The kube-apiserver is wired to discover public keys from the external signer,
cache them, and validate tokens it did not sign itself,
so existing authentication and authorization flows continue to work as expected.
Over the alpha and beta phases, the API and configuration for the external signer plugin,
path validation, and OIDC discovery were hardened to handle real-world deployments and rotation patterns safely.</p>
<p>With GA in v1.36, external ServiceAccount token signing is now a fully supported option for platforms that
centralize identity and signing, simplifying integration with external IAM systems and
reducing the need to manage signing keys directly inside the control plane.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/740">KEP #740</a> led by SIG Auth.</p>
<h3 id="dra-features-graduating-to-stable">DRA features graduating to Stable</h3>
<p>Part of the Dynamic Resource Allocation (DRA) ecosystem reaches full production maturity in
Kubernetes v1.36 as key governance and selection features graduate to Stable.
The transition of <em>DRA admin access</em> to GA provides a permanent, secure framework for cluster administrators
to access and manage hardware resources globally, while the stabilization of <em>prioritized lists</em> ensures that
resource selection logic remains consistent and predictable across all cluster environments.</p>
<p>Now, organizations can confidently deploy mission-critical hardware automation with the guarantee
of long-term API stability and backward compatibility. These features empower users to implement
sophisticated resource-sharing policies and administrative overrides that are essential for
large-scale GPU clusters and multi-tenant AI platforms, marking the completion of the
core architectural foundation for next-generation resource management.</p>
<p>This work was done as part of KEPs <a href="https://kep.k8s.io/5018">#5018</a> and <a href="https://kep.k8s.io/4816">#4816</a> led by SIG Auth and SIG Scheduling.</p>
<h3 id="mutating-admission-policies">Mutating admission policies</h3>
<p>Declarative cluster management reaches a new level of sophistication in Kubernetes v1.36 with the
graduation of <a href="https://kubernetes.io/docs/reference/access-authn-authz/mutating-admission-policy/">MutatingAdmissionPolicies</a> to Stable. This milestone provides a native,
high-performance alternative to traditional webhooks by allowing administrators to
define resource mutations directly in the API server using the Common Expression Language (CEL),
fully replacing the need for external infrastructure for many common use cases.</p>
<p>Now, cluster operators can modify incoming requests without the latency and operational
complexity associated with managing custom admission webhooks.
By moving mutation logic into a declarative, versioned policy, organizations can achieve
more predictable cluster behavior, reduced network overhead,
and a hardened security model with the full guarantee of long-term API stability.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/3962">KEP #3962</a> led by SIG API Machinery.</p>
<h3 id="declarative-validation-of-kubernetes-native-types-with-validation-gen">Declarative validation of Kubernetes native types with <code>validation-gen</code></h3>
<p>The development of custom resources reaches a new level of efficiency in Kubernetes v1.36
as <em>declarative validation</em> (with <code>validation-gen</code>) graduates to Stable.
This milestone replaces the manual and often error-prone task of writing complex
OpenAPI schemas by allowing developers to define sophisticated validation logic directly
within Go struct tags using the Common Expression Language (CEL).</p>
<p>Instead of writing custom validation functions, Kubernetes contributors can now define validation
rules using IDL marker comments (such as <code>+k8s:minimum</code> or <code>+k8s:enum</code>) directly
within the API type definitions (<code>types.go</code>). The <code>validation-gen</code> tool parses these
comments to automatically generate robust API validation code at compile-time.
This reduces maintenance overhead and ensures that API validation
remains consistent and synchronized with the source code.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/5073">KEP #5073</a> led by SIG API Machinery.</p>
<h3 id="remove-gogo-protobuf-dependency-for-kubernetes-api-types">Removal of gogo protobuf dependency for Kubernetes API types</h3>
<p>Security and long-term maintainability for the Kubernetes codebase take a major step forward
in Kubernetes v1.36 with the completion of the <code>gogoprotobuf</code> removal.
This initiative has eliminated a significant dependency on the unmaintained <code>gogoprotobuf</code> library,
which had become a source of potential security vulnerabilities and
a blocker for adopting modern Go language features.</p>
<p>Instead of migrating to standard Protobuf generation, which presented compatibility risks
for Kubernetes API types, the project opted to fork and internalize the required
generation logic within <code>k8s.io/code-generator</code>. This approach successfully eliminates
the unmaintained runtime dependencies from the Kubernetes dependency graph
while preserving existing API behavior and serialization compatibility.
For consumers of Kubernetes API Go types, this change reduces technical debt and
prevents accidental misuse with standard protobuf libraries.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/5589">KEP #5589</a> led by SIG API Machinery.</p>
<h3 id="node-log-query">Node log query</h3>
<p>Previously, Kubernetes required cluster administrators to log into nodes via SSH or implement a
client-side reader for debugging issues pertaining to control-plane or worker nodes.
While certain issues still require direct node access, issues with the kube-proxy or kubelet
can be diagnosed by inspecting their logs. Node logs offer cluster administrators
a method to view these logs using the kubelet API and kubectl plugin
to simplify troubleshooting without logging into nodes, similar to debugging issues
related to a pod or container. This method is operating system agnostic and
requires the services or nodes to log to <code>/var/log</code>.</p>
<p>As this feature reaches GA in Kubernetes 1.36 after thorough performance validation on production workloads,
it is enabled by default on the kubelet through the <code>NodeLogQuery</code> feature gate.
In addition, the <code>enableSystemLogQuery</code> kubelet configuration option must also be enabled.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/2258">KEP #2258</a> led by SIG Windows.</p>
<h3 id="support-user-namespaces-in-pods">Support User Namespaces in pods</h3>
<p>Container isolation and node security reach a major maturity milestone in Kubernetes v1.36 as
support for User Namespaces graduates to Stable.
This long-awaited feature provides a critical layer of defense-in-depth by allowing the
mapping of a container's root user to a non-privileged user on the host,
ensuring that even if a process escapes the container,
it possesses no administrative power over the underlying node.</p>
<p>Now, cluster operators can confidently enable this hardened isolation for production
workloads to mitigate the impact of container breakout vulnerabilities.
By decoupling the container's internal identity from the host's identity,
Kubernetes provides a robust, standardized mechanism to protect multi-tenant
environments and sensitive infrastructure from unauthorized access,
all with the full guarantee of long-term API stability.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/127">KEP #127</a> led by SIG Node.</p>
<h3 id="support-psi-based-on-cgroupv2">Support PSI based on cgroupv2</h3>
<p>Node resource management and observability become more precise in Kubernetes v1.36
as the export of Pressure Stall Information (PSI) metrics graduates to Stable.
This feature provides the kubelet with the ability to report &quot;pressure&quot; metrics for CPU,
memory, and I/O, offering a more granular view of resource contention than
traditional utilization metrics.</p>
<p>Cluster operators and autoscalers can use these metrics to distinguish between a system that is
simply busy and one that is actively stalling due to resource exhaustion.
By leveraging these signals, users can more accurately tune pod resource requests,
improve the reliability of vertical autoscaling, and detect noisy neighbor
effects before they lead to application performance degradation or node instability.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/4205">KEP #4205</a> led by SIG Node.</p>
<h3 id="volumesource-oci-artifact-and-or-image">Volume source: OCI artifact and/or image</h3>
<p>The distribution of container data becomes more flexible in Kubernetes v1.36 as <em>OCI volume source</em> support graduates to Stable.
This feature moves beyond the traditional requirement of mounting volumes from external storage providers
or config maps by allowing the kubelet to pull and mount content directly from any OCI-compliant registry,
such as a container image or an artifact repository.</p>
<p>Now, developers and platform engineers can package application data, models, or static assets as OCI artifacts
and deliver them to pods using the same registries and versioning workflows they already use for container images.
This convergence of image and volume management simplifies CI/CD pipelines,
reduces dependency on specialized storage backends for read-only content,
and ensures that data remains portable and securely accessible across any environment.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/4639">KEP #4639</a> led by SIG Node.</p>
<h2 id="new-features-in-beta">New features in Beta</h2>
<p><em>This is a selection of some of the improvements that are now beta following the v1.36 release.</em></p>
<h3 id="staleness-mitigation-for-controllers">Staleness mitigation for controllers</h3>
<p>Staleness in Kubernetes controllers is a problem that affects many controllers and can subtly affect controller behavior.
It is usually not until it is too late, when a controller in production has already taken incorrect action,
that staleness is found to be an issue due to some underlying assumption made by the controller author.
This could lead to conflicting updates or data corruption upon controller reconciliation during times of cache staleness.</p>
<p>We are excited to announce that Kubernetes v1.36 includes new features that help mitigate controller staleness and
provide better observability of controller behavior.
This prevents reconciliation based on an outdated view of cluster state that can often lead to harmful behavior.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/5647">KEP #5647</a> led by SIG API Machinery.</p>
<h3 id="ip-cidr-validation-improvements">IP/CIDR validation improvements</h3>
<p>In Kubernetes v1.36, the <code>StrictIPCIDRValidation</code> feature for API IP and CIDR fields graduates to beta,
tightening validation to catch malformed addresses and prefixes that previously slipped through.
This helps prevent subtle configuration bugs where Services, Pods, NetworkPolicies,
or other resources reference invalid IPs, which could otherwise lead to
confusing runtime behavior or security surprises.</p>
<p>Controllers are updated to canonicalize IPs they write back into objects and to warn when they
encounter bad values that were already stored, so clusters can gradually converge on clean,
consistent data. With beta, <code>StrictIPCIDRValidation</code> is ready for wider use,
giving operators more reliable guardrails around IP-related configuration
as they evolve networks and policies over time.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/4858">KEP #4858</a> led by SIG Network.</p>
<h3 id="separate-kubectl-user-preferences-from-cluster-configs">Separate kubectl user preferences from cluster configs</h3>
<p>The <code>.kuberc</code> feature for customizing <code>kubectl</code> user preferences continues to be beta
and is enabled by default. The <code>~/.kube/kuberc</code> file allows users to store aliases, default flags,
and other personal settings separately from <code>kubeconfig</code> files, which hold cluster endpoints and credentials.
This separation prevents personal preferences from interfering with CI pipelines or shared <code>kubeconfig</code> files,
while maintaining a consistent <code>kubectl</code> experience across different clusters and contexts.</p>
<p>In Kubernetes v1.36, <code>.kuberc</code> was expanded with the ability to define policies for credential plugins
(allowlists or denylists) to enforce safer authentication practicies.
Users can disable this functionality if needed by setting the <code>KUBECTL_KUBERC=false</code> or <code>KUBERC=off</code> environment variables.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/3104">KEP #3104</a> led by SIG CLI, with the help from SIG Auth.</p>
<h3 id="mutable-container-resources-when-job-is-suspended">Mutable container resources when Job is suspended</h3>
<p>In Kubernetes v1.36, the <code>MutablePodResourcesForSuspendedJobs</code> feature graduates to beta and is enabled by default.
This update relaxes Job validation to allow updates to container CPU, memory,
GPU, and extended resource requests and limits while a Job is suspended.</p>
<p>This capability allows queue controllers and operators to adjust batch workload requirements based on
real‑time cluster conditions. For example, a queueing system can suspend incoming Jobs,
adjust their resource requirements to match available capacity or quota, and then unsuspend them.
The feature strictly limits mutability to suspended Jobs (or Jobs whose pods have been terminated upon suspension)
to prevent disruptive changes to actively running pods.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/5440">KEP #5440</a> led by SIG Apps.</p>
<h3 id="constrained-impersonation">Constrained impersonation</h3>
<p>In Kubernetes v1.36, the <code>ConstrainedImpersonation</code> feature for user impersonation graduates to beta,
tightening a historically all‑or‑nothing mechanism into something that can actually follow least‑privilege principles.
When this feature is enabled, an impersonator must have two distinct sets of permissions:
one to impersonate a given identity, and another to perform specific actions on that identity’s behalf.
This prevents support tools, controllers, or node agents from using impersonation to gain broader access
than they themselves are allowed, even if their impersonation RBAC is misconfigured.
Existing impersonate rules keep working, but the API server prefers the new constrained checks first,
making the transition incremental instead of a flag day. With beta in v1.36, <code>ConstrainedImpersonation</code> is tested,
documented, and ready for wider adoption by platform teams that rely on impersonation for debugging, proxying,
or node‑level controllers.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/5284">KEP #5284</a> led by SIG Auth.</p>
<h3 id="dra-features-in-beta">DRA features in beta</h3>
<p>The <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/">Dynamic Resource Allocation (DRA)</a> framework reaches another maturity milestone in Kubernetes v1.36 as several core features graduate to beta and are enabled by default.
This transition moves DRA beyond basic allocation by graduating <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/#partitionable-devices">partitionable devices</a> and <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/#consumable-capacity">consumable capacity</a>, allowing for more granular sharing of hardware like GPUs,
while <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/#device-taints-and-tolerations">device taints and tolerations</a> ensure that specialized resources are only utilized by the appropriate workloads.</p>
<p>Now, users benefit from a much more reliable and observable resource lifecycle through <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/#resourceclaim-device-status">ResourceClaim device status</a>
and the ability to ensure device attachment before Pod scheduling.
By integrating these features with <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/#extended-resource">extended resource</a> support,
Kubernetes provides a robust production-ready alternative to the legacy device plugin system,
enabling complex AI and HPC workloads to manage hardware with unprecedented precision and operational safety.</p>
<p>This work was done across several KEPs (including <a href="https://kep.k8s.io/5004">#5004</a>, <a href="https://kep.k8s.io/4817">#4817</a>, <a href="https://kep.k8s.io/5055">#5055</a>, <a href="https://kep.k8s.io/5075">#5075</a>, <a href="https://kep.k8s.io/4815">#4815</a>, and <a href="https://kep.k8s.io/issues/5007">#5007</a>) led by SIG Scheduling and SIG Node.</p>
<h3 id="statusz-for-kubernetes-components">Statusz for Kubernetes components</h3>
<p>In Kubernetes v1.36, the <code>ComponentStatusz</code> feature gate for core Kubernetes components graduates to beta,
providing a <code>/statusz</code> endpoint (enabled by default) that surfaces real‑time build and version details for each component.
This low‑overhead <a href="https://kubernetes.io/docs/reference/instrumentation/zpages/">z-page</a> exposes information like start time, uptime, Go version, binary version,
emulation version, and minimum compatibility version, so operators and developers can quickly see exactly
what is running without digging through logs or configs.</p>
<p>The endpoint offers a human‑readable text view by default, plus a versioned structured API (<code>config.k8s.io/v1beta1</code>)
for programmatic access in JSON, YAML, or CBOR via explicit content negotiation.
Access is granted to the <code>system:monitoring</code> group, keeping it aligned with existing protections on
health and metrics endpoints and avoiding exposure of sensitive data.</p>
<p>With beta, <code>ComponentStatusz</code> is enabled by default across all core control‑plane components and node agents,
backed by unit, integration, and end‑to‑end tests so it can be safely used in production for
observability and debugging workflows.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/4827">KEP #4827</a> led by SIG Instrumentation.</p>
<h3 id="flagz-for-kubernetes-components">Flagz for Kubernetes components</h3>
<p>In Kubernetes v1.36, the <code>ComponentFlagz</code> feature gate for core Kubernetes components graduates to beta,
standardizing a <code>/flagz</code> endpoint that exposes the effective command‑line flags each component was started with.
This gives cluster operators and developers real‑time, in‑cluster visibility into component configuration,
making it much easier to debug unexpected behavior or verify that a flag rollout actually took effect after a restart.</p>
<p>The endpoint supports both a human‑readable text view and a versioned structured API (initially <code>config.k8s.io/v1beta1</code>),
so you can either <code>curl</code> it during an incident or wire it into automated tooling once you are ready.
Access is granted to the <code>system:monitoring</code> group and sensitive values can be redacted,
keeping configuration insight aligned with existing security practices around health and status endpoints.</p>
<p>With beta, <code>ComponentFlagz</code> is now enabled by default and implemented across all core control‑plane components
and node agents, backed by unit, integration, and end‑to‑end tests to ensure the endpoint is reliable in production clusters.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/4828">KEP #4828</a> led by SIG Instrumentation.</p>
<h3 id="mixed-version-proxy">Mixed version proxy (aka <em>unknown version interoperability proxy</em>)</h3>
<p>In Kubernetes v1.36, the <em>mixed version proxy</em> feature graduates to beta, building on its alpha introduction in v1.28
to provide safer control-plane upgrades for mixed-version clusters. Each API request can now be routed to the apiserver
instance that serves the requested group, version, and resource, reducing 404s and failures due to version skew.</p>
<p>The feature relies on peer-aggregated discovery, so apiservers share information about which resources and versions they expose,
then use that data to transparently reroute requests when needed. New metrics on rerouted traffic and proxy behavior
help operators understand how often requests are forwarded and to which peers.
Together, these changes make it easier to run highly available, mixed-version API control planes in production
while performing multi-step or partial control-plane upgrades.</p>
<p>This work was done as a part of <a href="https://kep.k8s.io/4020">KEP #4020</a> led by SIG API Machinery</p>
<h3 id="memory-qos-with-cgroups-v2">Memory QoS with cgroups v2</h3>
<p>Kubernetes now enhances memory QoS on Linux cgroup v2 nodes with smarter, tiered memory protection that better aligns kernel
controls with pod requests and limits, reducing interference and thrashing for workloads sharing the same node.
This iteration also refines how kubelet programs memory.high and memory.min, adds metrics and safeguards to avoid livelocks,
and introduces configuration options so cluster operators can tune memory protection behavior for their environments.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/2570">KEP #2570</a> led by SIG Node.</p>
<h2 id="new-features-in-alpha">New features in Alpha</h2>
<p>This is a selection of some of the improvements that are now alpha following the v1.36 release.</p>
<h3 id="hpa-scale-to-zero-for-custom-metrics">HPA scale to zero for custom metrics</h3>
<p>Until now, the HorizontalPodAutoscaler (HPA) required a minimum of at least one replica to remain active,
as it could only calculate scaling needs based on metrics (like CPU or Memory) from running pods.
Kubernetes v1.36 continues the development of the <em>HPA scale to zero</em> feature (disabled by default) in Alpha,
allowing workloads to scale down to zero replicas specifically when using Object or External metrics.</p>
<p>Now, users can experiment with significantly reducing infrastructure costs by completely idling heavy workloads when
no work is pending. While the feature remains behind the <code>HPAScaleToZero</code> feature gate,
it enables the HPA to stay active even with zero running pods,
automatically scaling the deployment back up as soon as the external metric (e.g., queue length)
indicates that new tasks have arrived.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/2021">KEP #2021</a> led by SIG Autoscaling.</p>
<h3 id="dra-features-in-alpha">DRA features in Alpha</h3>
<p>Historically, the Dynamic Resource Allocation (DRA) framework lacked seamless integration with high-level controllers and
provided limited visibility into device-specific metadata or availability.
Kubernetes v1.36 introduces a wave of DRA enhancements in Alpha, including native ResourceClaim support for workloads,
and DRA native resources to provide the flexibility of DRA to cpu management.</p>
<p>Now, users can leverage the <a href="https://kubernetes.io/docs/concepts/workloads/pods/downward-api/">downward API</a> to expose complex resource attributes directly to containers and
benefit from improved resource availability visibility for more predictable scheduling. these updates,
combined with support for list types in device attributes, transform DRA from a low-level primitive into a robust system
capable of handling the sophisticated networking and compute requirements of modern AI and
high-performance computing (HPC) stacks.</p>
<p>This work was done across several KEPs (including <a href="https://kep.k8s.io/5729">#5729</a>, <a href="https://kep.k8s.io/5304">#5304</a>, <a href="https://kep.k8s.io/5517">#5517</a>, <a href="https://kep.k8s.io/5677">#5677</a>, and <a href="https://kep.k8s.io/5491">#5491</a>) led by SIG Scheduling and SIG Node.</p>
<h3 id="native-histogram-support-for-kubernetes-metrics">Native histogram support for Kubernetes metrics</h3>
<p>High-resolution monitoring reaches a new milestone in Kubernetes v1.36 with the introduction of native histogram support
in Alpha. While classical Prometheus histograms relied on static, pre-defined buckets that often forced a compromise
between data accuracy and memory usage, this update allows the control plane to export sparse histograms that
dynamically adjust their resolution based on real-time data.</p>
<p>Now, cluster operators can capture precise latency distributions for the kube-apiserver and other core components without
the overhead of manual bucket management. This architectural shift ensures more reliable SLIs and SLOs,
providing high-fidelity heatmaps that remain accurate even during the most unpredictable workload spikes.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/5808">KEP #5808</a> led by SIG Instrumentation.</p>
<h3 id="manifest-based-admission-control-config">Manifest based admission control config</h3>
<p>Managing admission controllers moves toward a more declarative and consistent model in Kubernetes v1.36 with the
introduction of <em>manifest-based admission control</em> configuration in Alpha.
This change addresses the long-standing challenge of configuring admission plugins through disparate command-line
flags or separate, complex config files by allowing administrators to define the desired state of admission control
directly through a structured manifest.</p>
<p>Now, cluster operators can manage admission plugin settings with the same versioned, declarative workflows used for
other Kubernetes objects, significantly reducing the risk of configuration drift and manual errors during cluster upgrades.
By centralizing these configurations into a unified manifest, the kube-apiserver becomes easier to audit and automate,
paving the way for more secure and reproducible cluster deployments.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/5793">KEP #5793</a> led by SIG API Machinery.</p>
<h3 id="cri-list-streaming">CRI list streaming</h3>
<p>With the introduction of <em>CRI list streaming</em> in Alpha, Kubernetes v1.36 uses new internal streaming operations.
This enhancement addresses the memory pressure and latency spikes often seen on large-scale nodes by replacing traditional,
monolithic <code>List</code> requests between the kubelet and the container runtime with a more efficient server-side streaming RPC.</p>
<p>Now, instead of waiting for a single, massive response containing all container or image data, the kubelet can process results
incrementally as they are streamed. This shift significantly reduces the peak memory footprint of the kubelet and improves
responsiveness on high-density nodes, ensuring that cluster management remains fluid even as the number of
containers per node continues to grow.</p>
<p>This work was done as part of <a href="https://kep.k8s.io/5825">KEP #5825</a> led by SIG Node.</p>
<h2 id="other-notable-changes">Other notable changes</h2>
<h3 id="ingress-nginx-retirement">Ingress NGINX retirement</h3>
<p>To prioritize the safety and security of the ecosystem, Kubernetes SIG Network and the Security Response Committee have
retired Ingress NGINX on March 24, 2026.
Since that date, there have been no further releases, no bugfixes, and no updates to resolve any security vulnerabilities discovered. Existing deployments of
Ingress NGINX will continue to function, and installation artifacts like Helm charts and container images will remain available.</p>
<p>For full details, see the <a href="https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/">official retirement announcement</a>.</p>
<h3 id="volume-selinux-labelling">Faster SELinux labelling for volumes (GA)</h3>
<p>Kubernetes v1.36 makes the SELinux volume mounting improvement generally available. This change replaced recursive file
relabeling with <code>mount -o context=XYZ</code> option, applying the correct SELinux label to the entire volume at mount time.
It brings more consistent performance and reduces Pod startup delays on SELinux-enforcing systems.</p>
<p>This feature was introduced as beta in v1.28 for <code>ReadWriteOncePod</code> volumes. In v1.32, it gained metrics and an opt-out
option (<code>securityContext.seLinuxChangePolicy: Recursive</code>) to help catch conflicts. Now in v1.36,
it reaches Stable and defaults to all volumes, with Pods or CSIDrivers opting in via <code>spec.seLinuxMount</code>.</p>
<p>However, we expect this feature to create the risk of breaking changes in the future Kubernetes releases, potentially due to sharing one volume between privileged and unprivileged Pods on the same node.</p>
<p>Developers have the responsibility of setting the <code>seLinuxChangePolicy</code> field and SELinux volume labels on Pods. Regardless of whether they are writing a Deployment, StatefulSet, DaemonSet or even a custom resource that includes a Pod template, being careless with these settings can lead to a range of problems such as Pods not starting up correctly when Pods share a volume.</p>
<p>Kubernetes v1.36 is the ideal release to audit your clusters. To learn more, check out <a href="https://kubernetes.io/blog/2026/04/22/breaking-changes-in-selinux-volume-labeling/">SELinux Volume Label Changes goes GA (and likely implications in v1.37)</a> blog.</p>
<p>For more details on this enhancement, refer to <a href="https://kep.k8s.io/1710">KEP-1710: Speed up recursive SELinux label change</a>.</p>
<h2 id="graduations-deprecations-and-removals-in-v1-36">Graduations, deprecations, and removals in v1.36</h2>
<h3 id="graduations-to-stable">Graduations to stable</h3>
<p>This lists all the features that graduated to stable (also known as general availability).
For a full list of updates including new features and graduations from alpha to beta, see the release notes.</p>
<p>This release includes a total of 18 enhancements promoted to stable:</p>
<ul>
<li><a href="https://kep.k8s.io/127">Support User Namespaces in pods</a></li>
<li><a href="https://kep.k8s.io/740">API for external signing of Service Account tokens</a></li>
<li><a href="https://kep.k8s.io/1710">Speed up recursive SELinux label change</a></li>
<li><a href="https://kep.k8s.io/2589">Portworx file in-tree to CSI driver migration</a></li>
<li><a href="https://kep.k8s.io/2862">Fine grained Kubelet API authorization</a></li>
<li><a href="https://kep.k8s.io/3962">Mutating Admission Policies</a></li>
<li><a href="https://kep.k8s.io/2258">Node log query</a></li>
<li><a href="https://kep.k8s.io/3476">VolumeGroupSnapshot</a></li>
<li><a href="https://kep.k8s.io/4876">Mutable CSINode Allocatable Property</a></li>
<li><a href="https://kep.k8s.io/4816">DRA: Prioritized Alternatives in Device Requests</a></li>
<li><a href="https://kep.k8s.io/4205">Support PSI based on cgroupv2</a></li>
<li><a href="https://kep.k8s.io/4265">add ProcMount option</a></li>
<li><a href="https://kep.k8s.io/3695">DRA: Extend PodResources to include resources from Dynamic Resource Allocation</a></li>
<li><a href="https://kep.k8s.io/4639">VolumeSource: OCI Artifact and/or Image</a></li>
<li><a href="https://kep.k8s.io/5109">Split L3 Cache Topology Awareness in CPU Manager</a></li>
<li><a href="https://kep.k8s.io/5018">DRA: AdminAccess for ResourceClaims and ResourceClaimTemplates</a></li>
<li><a href="https://kep.k8s.io/5589">Remove gogo protobuf dependency for Kubernetes API types</a></li>
<li><a href="https://kep.k8s.io/5538">CSI driver opt-in for service account tokens via secrets field</a></li>
</ul>
<h2 id="deprecations-removals-and-community-updates">Deprecations removals, and community updates</h2>
<p>As Kubernetes develops and matures, features may be deprecated, removed, or replaced with better ones to improve the
project's overall health. See the Kubernetes deprecation and removal policy for more details on this process.
Kubernetes v1.36 includes a couple of deprecations.</p>
<h3 id="deprecate-service-spec-externalips">Deprecation of Service .spec.externalIPs</h3>
<p>With this release, the <code>externalIPs</code> field in Service <code>spec</code> is deprecated. This means the functionality exists, but will no longer function in a <strong>future</strong> version of Kubernetes. You should plan to migrate if you currently rely on that field.
This field has been a known security headache for years,
enabling man-in-the-middle attacks on your cluster traffic, as documented in <a href="https://github.com/kubernetes/kubernetes/issues/97076">CVE-2020-8554</a>.
From Kubernetes v1.36 and onwards, you will see deprecation warnings when using it, with full removal planned for v1.43.</p>
<p>If your Services still lean on <code>externalIPs</code>, consider using LoadBalancer services for cloud-managed ingress,
NodePort for simple port exposure, or <a href="https://gateway-api.sigs.k8s.io/">Gateway API</a> for a more flexible and secure way to handle external traffic.</p>
<p>For more details on this field and its deprecation, refer to <a href="https://kubernetes.io/docs/concepts/services-networking/service/#external-ips">External IPs</a> or read
<a href="https://kep.k8s.io/5707">KEP-5707: Deprecate service.spec.externalIPs</a>.</p>
<h3 id="remove-gitrepo-volume-driver">Removal of the <code>gitRepo</code> volume driver</h3>
<p>The <code>gitRepo</code> volume type has been deprecated since v1.11. For Kubernetes v1.36, the <code>gitRepo</code> volume plugin is
permanently disabled and cannot be turned back on. This change protects clusters from a critical security issue where
using <code>gitRepo</code> could let an attacker run code as root on the node.</p>
<p>Although <code>gitRepo</code> has been deprecated for years and better alternatives have been recommended,
it was still technically possible to use it in previous releases.
From v1.36 onward, that path is closed for good, so any existing workloads depending on <code>gitRepo</code> will need to migrate to
supported approaches such as init containers or external <code>git-sync</code> style tools.</p>
<p>For more details on this removal, refer to <a href="https://kep.k8s.io/5040">KEP-5040: Remove gitRepo volume driver</a></p>
<h2 id="release-notes">Release notes</h2>
<p>Check out the full details of the Kubernetes v1.36 release in our <a href="https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.36.md">release notes</a>.</p>
<h2 id="availability">Availability</h2>
<p>Kubernetes v1.36 is available for download on <a href="https://github.com/kubernetes/kubernetes/releases/tag/v1.36.0">GitHub</a> or on the <a href="https://kubernetes.io/releases/download/">Kubernetes download page</a>.</p>
<p>To get started with Kubernetes, check out <a href="https://kubernetes.io/docs/tutorials/">these tutorials</a> or run local Kubernetes clusters using <a href="https://minikube.sigs.k8s.io/">minikube</a>.
You can also easily <a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/">install v1.36 using kubeadm</a>.</p>
<h2 id="release-team">Release Team</h2>
<p>Kubernetes is only possible with the support, commitment, and hard work of its community. Each release team is
made up of dedicated community volunteers who work together to build the many pieces that make up the
Kubernetes releases you rely on. This requires the specialized skills of people from all corners of our community,
from the code itself to its documentation and project management.</p>
<p>We would like to thank the entire <a href="https://github.com/kubernetes/sig-release/blob/master/releases/release-1.36/release-team.md">Release Team</a> for the hours spent hard at work to deliver the Kubernetes v1.36 release to our community.
The Release Team's membership ranges from first-time shadows to returning team leads with experience forged over
several release cycles. A very special thanks goes out to our release lead, Ryota Sawada,
for guiding us through a successful release cycle, for his hands-on approach to solving challenges,
and for bringing the energy and care that drives our community forward.</p>
<h2 id="project-velocity">Project Velocity</h2>
<p>The CNCF K8s <a href="https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&amp;var-period=m&amp;var-repogroup_name=All">DevStats</a> project aggregates a number of interesting data points related to the velocity of
Kubernetes and various sub-projects. This includes everything from individual contributions to the number of
companies that are contributing, and is an illustration of the depth and breadth of effort that goes into evolving this ecosystem.</p>
<p>During the v1.36 release cycle, which spanned 15 weeks from 12th January 2026 to 22nd April 2026,
Kubernetes received contributions from as many as 106 different companies and 491 individuals.
In the wider cloud native ecosystem, the figure goes up to 370 companies, counting 2235 total contributors.</p>
<p>Note that “contribution” counts when someone makes a commit, code review, comment, creates an issue or PR,
reviews a PR (including blogs and documentation) or comments on issues and PRs.
If you are interested in contributing, visit <a href="https://www.kubernetes.dev/docs/guide/#getting-started">Getting Started</a> on our contributor website.</p>
<p>Source for this data:</p>
<ul>
<li><a href="https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&amp;from=1747609200000&amp;to=1756335599000&amp;var-period=d28&amp;var-repogroup_name=Kubernetes&amp;var-repo_name=kubernetes%2Fkubernetes">Companies contributing to Kubernetes</a></li>
<li><a href="https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&amp;from=1747609200000&amp;to=1756335599000&amp;var-period=d28&amp;var-repogroup_name=All&amp;var-repo_name=kubernetes%2Fkubernetes">Overall ecosystem contributions</a></li>
</ul>
<h2 id="events-update">Events Update</h2>
<p>Explore upcoming Kubernetes and cloud native events, including KubeCon + CloudNativeCon, KCD,
and other notable conferences worldwide. Stay informed and get involved with the Kubernetes community!</p>
<p><strong>April 2026</strong></p>
<ul>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-guadalajara-presents-kcd-guadalajara-2026/cohost-kcd-guadalajara/">Kubernetes Community Days: Guadalajara</a>: April 18, 2026 | Guadalajara, Mexico</li>
</ul>
<p><strong>May 2026</strong></p>
<ul>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-toronto-presents-kcd-toronto-2026/">Kubernetes Community Days: Toronto</a>: May 13, 2026 | Toronto, Canada</li>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-texas-presents-kcd-texas-2026/cohost-kcd-texas/">Kubernetes Community Days: Texas</a>: May 15, 2026 | Austin, USA</li>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-istanbul-presents-kcd-istanbul-2026/">Kubernetes Community Days: Istanbul</a>: May 15, 2026 | Istanbul, Turkey</li>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-helsinki-presents-kubernetes-community-days-helsinki-2026/">Kubernetes Community Days: Helsinki</a>: May 20, 2026 | Helsinki, Finland</li>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-czech-slovak-presents-kcd-czech-amp-slovak-prague-2026/">Kubernetes Community Days: Czech &amp; Slovak</a>: May 21, 2026 | Prague, Czechia</li>
</ul>
<p><strong>June 2026</strong></p>
<ul>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-new-york-presents-kcd-new-york-2026/">Kubernetes Community Days: New York</a>: June 10, 2026 | New York, USA</li>
<li><a href="https://events.linuxfoundation.org/kubecon-cloudnativecon-india/">KubeCon + CloudNativeCon India 2026: June 18-19, 2026</a> | Mumbai, India</li>
</ul>
<p><strong>July 2026</strong></p>
<ul>
<li><a href="https://events.linuxfoundation.org/kubecon-cloudnativecon-japan/">KubeCon + CloudNativeCon Japan 2026: July 29-30, 2026</a> | Yokohama, Japan</li>
</ul>
<p><strong>September 2026</strong></p>
<ul>
<li><a href="https://www.lfopensource.cn/kubecon-cloudnativecon-openinfra-summit-pytorch-conference-china/">KubeCon + CloudNativeCon China 2026: September 8-9, 2026</a> | Shanghai, China</li>
</ul>
<p><strong>October 2026</strong></p>
<ul>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-uk-presents-kubernetes-community-days-uk-edinburgh-2026/">Kubernetes Community Days: UK: Oct 19, 2026</a> | Edinburgh, UK</li>
</ul>
<p><strong>November 2026</strong></p>
<ul>
<li>KCD - <a href="https://community.cncf.io/events/details/cncf-kcd-porto-presents-kcd-porto-2026-collab-with-devops-days-portugal/">Kubernetes Community Days: Porto</a>: Nov 19, 2026 | Porto, Portugal</li>
<li><a href="https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/">KubeCon + CloudNativeCon North America 2026</a>: Nov 9-12, 2026 | Salt Lake, USA</li>
</ul>
<p>You can find the latest event details <a href="https://community.cncf.io/events/#/list">here</a>.</p>
<h2 id="upcoming-release-webinar">Upcoming Release Webinar</h2>
<p>Join members of the Kubernetes v1.36 Release Team on <strong>Wednesday, May 20th 2026 at 4:00 PM (UTC)</strong> to learn about the release highlights
of this release. For more information and registration, visit the <a href="https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cloud-native-live-kubernetes-v136-release/">event page</a> on the CNCF Online Programs site.</p>
<h2 id="get-involved">Get Involved</h2>
<p>The simplest way to get involved with Kubernetes is by joining one of the many <a href="https://github.com/kubernetes/community/blob/master/sig-list.md">Special Interest Groups</a> (SIGs) that align with your interests.
Have something you’d like to broadcast to the Kubernetes community? Share your voice at our weekly <a href="https://github.com/kubernetes/community/tree/master/communication">community meeting</a>,
and through the channels below. Thank you for your continued feedback and support.</p>
<ul>
<li>Follow us on Bluesky <a href="https://bsky.app/profile/kubernetes.io">@kubernetes.io</a> for the latest updates</li>
<li>Join the community discussion on <a href="https://discuss.kubernetes.io/">Discuss</a></li>
<li>Join the community on <a href="https://slack.k8s.io/">Slack</a></li>
<li>Post questions (or answer questions) on <a href="https://stackoverflow.com/questions/tagged/kubernetes">Stack Overflow</a></li>
<li>Share your Kubernetes <a href="https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform">story</a></li>
<li>Read more about what’s happening with Kubernetes on the <a href="https://kubernetes.io/blog/">blog</a></li>
<li>Learn more about the <a href="https://github.com/kubernetes/sig-release/tree/master/release-team">Kubernetes Release Team</a></li>
</ul>
