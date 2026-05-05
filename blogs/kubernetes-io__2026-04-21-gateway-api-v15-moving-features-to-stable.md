---
title: "Gateway API v1.5: Moving features to Stable"
url: "https://kubernetes.io/blog/2026/04/21/gateway-api-v1-5/"
date: "Tue, 21 Apr 2026 08:30:00 -0800"
author: ""
feed_url: "https://kubernetes.io/feed.xml"
---
<p><img alt="Gateway API logo" src="https://kubernetes.io/blog/2026/04/21/gateway-api-v1-5/gateway-api-logo.svg" /></p>
<p>The Kubernetes SIG Network community presents the release of Gateway API (v1.5)!
Released on February 27, 2026, version 1.5 is our biggest release yet, and concentrates on moving existing Experimental features to Standard (Stable).</p>
<p>The Gateway API <a href="https://github.com/kubernetes-sigs/gateway-api/releases/tag/v1.5.1">v1.5.1</a> patch release is already available.</p>
<p>The Gateway API v1.5 brings six widely-requested feature promotions to the Standard channel (Gateway API's GA release channel):</p>
<ul>
<li>ListenerSet</li>
<li>TLSRoute</li>
<li>HTTPRoute CORS Filter</li>
<li>Client Certificate Validation</li>
<li>Certificate Selection for Gateway TLS Origination</li>
<li>ReferenceGrant</li>
</ul>
<p>Special thanks for <a href="https://github.com/kubernetes-sigs/gateway-api/blob/a811d174a406553006bbb9a3594b49380cb9069e/CHANGELOG/1.5-TEAM.md">Gateway API Contributors</a> for their efforts on this release.</p>
<h2 id="new-release-process">New release process</h2>
<p>As of Gateway API v1.5, the project has moved to a release train model, where on a feature freeze date, any features that are ready are shipped in the release.</p>
<p>This applies to both Experimental and Standard, and also applies to documentation -- if the documentation isn't ready to ship, the feature isn't ready to ship.</p>
<p>We are aiming for this to produce a more reliable release cadence (since we are basing our work off the excellent work done by SIG Release on Kubernetes itself).
As part of this change, we've also introduced Release Manager and Release Shadow roles to our release team. Many thanks to Flynn (Buoyant) and Beka Modebadze (Google) for all the great work coordinating and filing the rough edges of our release process. They are both going to continue in this role for the next release as well.</p>
<h2 id="new-standard-features">New standard features</h2>
<h3 id="listenerset">ListenerSet</h3>
<p>Leads: <a href="https://github.com/dprotaso">Dave Protasowski</a>, <a href="https://github.com/davidjumani">David Jumani</a></p>
<p><a href="https://gateway-api.sigs.k8s.io/geps/gep-1713/">GEP-1713</a></p>
<h4 id="why-listenerset">Why ListenerSet?</h4>
<p>Prior to ListenerSet, all listeners had to be specified directly on the Gateway object.
While this worked well for simple use cases, it created challenges for more complex
or multi-tenant environments:</p>
<ul>
<li>Platform teams and application teams often needed to coordinate changes to the same Gateway</li>
<li>Safely delegating ownership of individual listeners was difficult</li>
<li>Extending existing Gateways required direct modification of the original resource</li>
</ul>
<p><a href="https://gateway-api.sigs.k8s.io/guides/listener-set/">ListenerSet</a> addresses these limitations by allowing listeners to be defined independently and then merged onto a target Gateway.</p>
<p>ListenerSets also enable attaching more than 64 listeners to a single, shared Gateway. This is critical for large scale deployments and scenarios with multiple hostnames per listener.</p>
<blockquote>
<p>Even though the ListenerSet feature significantly enhances scalability, the <strong>listener</strong> field in Gateway <strong>remains a mandatory requirement</strong> and the Gateway must have at least one valid listener.</p>
</blockquote>
<h4 id="how-it-works">How it works</h4>
<p>A ListenerSet attaches to a Gateway and contributes one or more listeners.
The Gateway controller is responsible for merging listeners from the Gateway resource itself and any attached ListenerSet resources.</p>
<p>In this example, a central infrastructure team defines a Gateway with a default HTTP listener,
while two different application teams define their own ListenerSet resources in separate namespaces.
Both ListenerSets attach to the same Gateway and contribute additional HTTPS listeners.</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>infra<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">gatewayClassName</span>:<span style="color: #bbb;"> </span>example-gateway-class<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowedListeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespaces</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">from</span>:<span style="color: #bbb;"> </span>All<span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># A selector lets you fine tune this</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>http<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>HTTP<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">80</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ListenerSet<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>team-a-listeners<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>team-a<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>infra<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>https-a<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>HTTPS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostname</span>:<span style="color: #bbb;"> </span>a.example.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">certificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>a-cert<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ListenerSet<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>team-b-listeners<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>team-b<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>infra<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>https-b<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>HTTPS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostname</span>:<span style="color: #bbb;"> </span>b.example.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">certificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>b-cert<span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="tlsroute">TLSRoute</h3>
<p>Leads: <a href="https://github.com/rostislavbobo">Rostislav Bobrovsky</a>, <a href="https://github.com/rikatz">Ricardo Pchevuzinske Katz</a></p>
<p><a href="https://gateway-api.sigs.k8s.io/geps/gep-2643/">GEP-2643</a></p>
<p>The <a href="https://gateway-api.sigs.k8s.io/api-types/tlsroute/">TLSRoute resource</a> allows you to route requests by matching the Server Name Indication (SNI) presented by the client during the TLS handshake and directing the stream to the appropriate Kubernetes backends.</p>
<p>When working with TLSRoute, a Gateway's TLS listener can be configured in one of two modes: <code>Passthrough</code> or <code>Terminate</code>.</p>
<p><strong>If you install Gateway API v1.5 Standard over v1.4 or earlier Experimental, your existing
Experimental TLSRoutes will not be usable</strong>. This is because they will be stored in the <code>v1alpha2</code> or <code>v1alpha3</code> version, which is <strong><em>not</em></strong> included in the v1.5 Standard YAMLs.
If this applies to you, either continue using Experimental for v1.5.1 and onward, or you'll need to download and migrate your TLSRoutes to <code>v1</code>, which <em>is</em> present in the Standard YAMLs.</p>
<h4 id="passthrough-mode">Passthrough mode</h4>
<p>The Passthrough mode is designed for strict security requirements. It is ideal for scenarios where traffic must remain encrypted end-to-end until it reaches the destination backend, when the external client and backend need to authenticate directly with each other, or when you can’t store certificates on the Gateway. This configuration is also applicable when an encrypted TCP stream is required instead of standard HTTP traffic.</p>
<p>In this mode, the encrypted byte stream is proxied directly to the destination backend. The Gateway has zero access to private keys or unencrypted data.</p>
<p>The following TLSRoute is attached to a listener that is configured in <code>Passthrough</code> mode. It will match only TLS handshakes with the <code>foo.example.com</code> SNI hostname and apply its routing rules to pass the encrypted TCP stream to the configured backend:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">gatewayClassName</span>:<span style="color: #bbb;"> </span>example-gateway-class<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>tls-passthrough<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>TLS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">mode</span>:<span style="color: #bbb;"> </span>Passthrough<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>TLSRoute<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-route<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">sectionName</span>:<span style="color: #bbb;"> </span>tls-passthrough<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostnames</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"foo.example.com"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">backendRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-svc<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8443</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h4 id="terminate-mode">Terminate mode</h4>
<p>The Terminate mode provides the convenience of centralized TLS certificate management directly at the Gateway.</p>
<p>In this mode, the TLS session is fully terminated at the Gateway, which then
routes the decrypted payload to the destination backend as a plain text TCP stream.</p>
<p>The following TLSRoute is attached to a listener that is configured in Terminate mode.
It will match only TLS handshakes with the <code>bar.example.com</code> SNI hostname and apply its routing rules to pass the decrypted TCP stream to the configured backend:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">gatewayClassName</span>:<span style="color: #bbb;"> </span>example-gateway-class<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>tls-terminate<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>TLS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">mode</span>:<span style="color: #bbb;"> </span>Terminate<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">certificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>tls-terminate-certificate<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #00f; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>TLSRoute<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>bar-route<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>example-gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">sectionName</span>:<span style="color: #bbb;"> </span>tls-terminate<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostnames</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"bar.example.com"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">backendRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>bar-svc<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8080</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="httproute-cors-filter">HTTPRoute CORS filter</h3>
<p>Leads: <a href="https://github.com/DamianSawicki">Damian Sawicki</a>, <a href="https://github.com/rikatz">Ricardo Pchevuzinske Katz</a>, <a href="https://github.com/snorwin">Norwin Schnyder</a>, <a href="https://github.com/zhaohuabing">Huabing (Robin) Zhao</a>, <a href="https://github.com/LiangLliu">LiangLliu</a>,</p>
<p><a href="https://gateway-api.sigs.k8s.io/geps/gep-1767/">GEP-1767</a></p>
<p>Cross-origin resource sharing (CORS) is an HTTP-header based security mechanism that allows (or denies) a web page to access resources from a server on an origin different from the domain that served the web page. See our <a href="https://gateway-api.sigs.k8s.io/guides/http-cors/">documentation page</a> for more information.
The <a href="https://gateway-api.sigs.k8s.io/api-types/httproute/">HTTPRoute resource</a> can be used to configure Cross-Origin Resource Sharing (CORS). The following HTTPRoute allows requests from <a href="https://app.example">https://app.example</a>:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HTTPRoute<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>cors<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>same-namespace<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">matches</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">path</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>PathPrefix<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">value</span>:<span style="color: #bbb;"> </span>/cors-behavior-creds-false<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">backendRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>infra-backend-v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8080</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">filters</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">cors</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowOrigins</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- https://app.example<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>CORS<span style="color: #bbb;">
</span></span></span></code></pre></div><p>Instead of specifying a list of specific origins, you can also specify a single wildcard (&quot;*&quot;), which will allow any origin. It is also allowed to use semi-specified origins in the list, where the wildcard appears after the scheme and at the beginning of the hostname, e.g. https://*.bar.com:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HTTPRoute<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>cors<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>same-namespace<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">matches</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">path</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>PathPrefix<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">value</span>:<span style="color: #bbb;"> </span>/cors-behavior-creds-false<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">backendRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>infra-backend-v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8080</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">filters</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">cors</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowOrigins</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- https://www.baz.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- https://*.bar.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- https://*.foo.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>CORS<span style="color: #bbb;">
</span></span></span></code></pre></div><p>HTTPRoute filters allow for the configuration of CORS settings. See a list of supported options below:</p>
<dl>
<dt><code>allowCredentials</code></dt>
<dd>Specifies whether the browser is allowed to include credentials (such as cookies and HTTP authentication) in the CORS request.</dd>
<dt><code>allowMethods</code></dt>
<dd>The HTTP methods that are allowed for CORS requests.</dd>
<dt><code>allowHeaders</code></dt>
<dd>The HTTP headers that are allowed for CORS requests.</dd>
<dt><code>exposeHeaders</code></dt>
<dd>The HTTP headers that are exposed to the client.</dd>
<dt><code>maxAge</code></dt>
<dd>The maximum time in seconds that the browser should cache the preflight response.</dd>
</dl>
<p>A comprehensive example:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HTTPRoute<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>cors-allow-credentials<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">parentRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>same-namespace<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">rules</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">matches</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">path</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>PathPrefix<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">value</span>:<span style="color: #bbb;"> </span>/cors-behavior-creds-true<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">backendRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>infra-backend-v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8080</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">filters</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">cors</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowOrigins</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"https://www.foo.example.com"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"https://*.bar.example.com"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowMethods</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- GET<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- OPTIONS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowHeaders</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"*"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">exposeHeaders</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"x-header-3"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #b44;">"x-header-4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">allowCredentials</span>:<span style="color: #bbb;"> </span><span style="color: #a2f; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">maxAge</span>:<span style="color: #bbb;"> </span><span style="color: #666;">3600</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>CORS<span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="gateway-client-certificate-validation">Gateway client certificate validation</h3>
<p>Leads: <a href="https://github.com/arkodg">Arko Dasgupta</a>, <a href="https://github.com/kl52752">Katarzyna Łach</a>, <a href="https://github.com/snorwin">Norwin Schnyder</a></p>
<p><a href="https://gateway-api.sigs.k8s.io/geps/gep-91/">GEP-91</a></p>
<p>Client certificate validation, also known as mutual TLS (mTLS), is a security mechanism where the client provides a certificate to the server to prove its identity. This is in contrast to standard TLS, where only the server presents a certificate to the client.
In the context of the Gateway API, frontend mTLS means that the Gateway validates the client's certificate before allowing the connection to proceed to a backend service. This validation is done by checking the client certificate against a set of trusted Certificate Authorities (CAs) configured on the Gateway. The API was shaped this way to address a critical security vulnerability related to connection reuse and still provide some level of flexibility.</p>
<h4 id="configuration-overview">Configuration overview</h4>
<p>Client validation is defined using the frontendValidation struct, which specifies how the Gateway should verify the client's identity.</p>
<ul>
<li><strong>caCertificateRefs</strong>: A list of references to Kubernetes objects (typically ConfigMap's) containing PEM-encoded CA certificate bundles used as trust anchors to validate the client's certificate.</li>
<li><strong>mode</strong>: Defines the validation behavior.
<ul>
<li><strong>AllowValidOnly</strong> (Default): The Gateway accepts connections only if the client presents a valid certificate that passes validation against the specified CA bundle.</li>
<li><strong>AllowInsecureFallback</strong>: The Gateway accepts connections even if the client certificate is missing or fails verification. This mode typically delegates authorization to the backend and should be used with caution.</li>
</ul>
</li>
</ul>
<p>Validation can be applied globally to the Gateway or overridden for specific ports:</p>
<ol>
<li><strong>Default Configuration</strong>: This configuration applies to all HTTPS listeners on the Gateway, unless a per-port override is defined.</li>
<li><strong>Per-Port Configuration</strong>: This allows for fine-grained control, overriding the default configuration for all listeners handling traffic on a specific port.</li>
</ol>
<p>Example:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>client-validation-basic<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">gatewayClassName</span>:<span style="color: #bbb;"> </span>acme-lb<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">frontend</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">default</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">validation</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">caCertificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ConfigMap<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">group</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">""</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-example-com-ca-cert<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">perPort</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">validation</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">caCertificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ConfigMap<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">group</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">""</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-example-com-ca-cert<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">mode</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">"AllowInsecureFallback"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-https<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>HTTPS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostname</span>:<span style="color: #bbb;"> </span>foo.example.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">certificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Secret<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">group</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">""</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-example-com-cert<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>bar-https<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>HTTPS<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">8443</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostname</span>:<span style="color: #bbb;"> </span>bar.example.com<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">certificateRefs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Secret<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">group</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">""</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>bar-example-com-cert<span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="certificate-selection-for-gateway-tls-origination">Certificate selection for Gateway TLS origination</h3>
<p>Leads: <a href="https://github.com/mkosieradzki">Marcin Kosieradzki</a>, <a href="https://github.com/robscott">Rob Scott</a>, <a href="https://github.com/snorwin">Norwin Schnyder</a>, <a href="https://github.com/LiorLieberman">Lior Lieberman</a>, <a href="https://github.com/kl52752">Katarzyna Lach</a></p>
<p><a href="https://gateway-api.sigs.k8s.io/geps/gep-3155/">GEP-3155</a></p>
<p>Mutual TLS (mTLS) for upstream connections requires the Gateway to present a client certificate to the backend, in addition to verifying the backend's certificate. This ensures that the backend only accepts connections from authorized Gateways.</p>
<h4 id="gateway-s-client-certificate-configuration">Gateway’s client certificate configuration</h4>
<p>To configure the client certificate that the Gateway uses when connecting to backends,
use the <strong>tls.backend.clientCertificateRef</strong> field in the Gateway resource.
This configuration applies to the Gateway as a client for <strong>all</strong> upstream connections managed by that Gateway.</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #008000; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>gateway.networking.k8s.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Gateway<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>backend-tls<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #008000; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">gatewayClassName</span>:<span style="color: #bbb;"> </span>acme-lb<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">tls</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">backend</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">clientCertificateRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Secret<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">group</span>:<span style="color: #bbb;"> </span><span style="color: #b44;">""</span><span style="color: #bbb;"> </span><span style="color: #080; font-style: italic;"># empty string means core API group</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-example-cert<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">listeners</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #008000; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>foo-http<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">protocol</span>:<span style="color: #bbb;"> </span>HTTP<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">port</span>:<span style="color: #bbb;"> </span><span style="color: #666;">80</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #008000; font-weight: bold;">hostname</span>:<span style="color: #bbb;"> </span>foo.example.com<span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="referencegrant-promoted-to-v1">ReferenceGrant promoted to v1</h3>
<p>The ReferenceGrant resource has not changed in more than a year, and we do not expect it to change further, so its version has been bumped to v1, and it is now officially in the Standard channel, and abides by the GA API contract (that is, no breaking changes).</p>
<h2 id="try-it-out">Try it out</h2>
<p>Unlike other Kubernetes APIs, you don't need to upgrade to the latest version of
Kubernetes to get the latest version of Gateway API. As long as you're running
Kubernetes 1.30 or later, you'll be able to get up and running with this version
of Gateway API.</p>
<p>To try out the API, follow the <a href="https://gateway-api.sigs.k8s.io/guides/">Getting Started Guide</a>.</p>
<p>As of this writing, seven implementations are already fully conformant with Gateway API v1.5. In alphabetical order:</p>
<ul>
<li><a href="https://github.com/agentgateway/agentgateway/releases/tag/v1.0.0">Agentgateway</a></li>
<li><a href="https://github.com/airlock/microgateway/releases/tag/5.0.0">Airlock Microgateway</a></li>
<li><a href="https://docs.cloud.google.com/kubernetes-engine/docs/concepts/gateway-api">GKE Gateway</a></li>
<li><a href="https://github.com/jcmoraisjr/haproxy-ingress/releases/tag/v0.17.0-alpha.1">HAProxy Ingress</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/releases/tag/v2.3.0-beta.3">kgateway</a></li>
<li><a href="https://github.com/nginx/nginx-gateway-fabric/releases/tag/v2.5.0">NGINX Gateway Fabric</a></li>
<li><a href="https://github.com/traefik/traefik/releases/tag/v3.7.0-rc.1">Traefik Proxy</a></li>
</ul>
<h2 id="get-involved">Get involved</h2>
<p>Wondering when a feature will be added? There are lots of opportunities to get
involved and help define the future of Kubernetes routing APIs for both ingress
and service mesh.</p>
<ul>
<li>Check out the <a href="https://gateway-api.sigs.k8s.io/guides">user guides</a> to see what use-cases can be addressed.</li>
<li>Try out one of the <a href="https://gateway-api.sigs.k8s.io/implementations/">existing Gateway controllers</a>.</li>
<li>Or <a href="https://gateway-api.sigs.k8s.io/contributing/">join us in the community</a> and help us build the future of Gateway API together!</li>
</ul>
<p>The maintainers would like to thank <strong>everyone</strong> who's contributed to Gateway
API, whether in the form of commits to the repo, discussion, ideas, or general
support. We could never have made this kind of progress without the support of
this dedicated and active community.</p>
<p><em>This article was edited in April 2026 to correct the release date for Gateway API 1.5.0.</em></p>
