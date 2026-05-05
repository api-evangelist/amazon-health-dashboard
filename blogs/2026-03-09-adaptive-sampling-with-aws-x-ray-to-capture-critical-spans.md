---
title: "Adaptive sampling with AWS X-Ray to capture critical spans"
url: "https://aws.amazon.com/blogs/mt/adaptive-sampling-with-aws-x-ray-to-capture-critical-spans/"
date: "Mon, 09 Mar 2026 16:28:59 +0000"
author: "Kartik Bheemisetty"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<h1>Introduction</h1> 
<p>Enterprise applications using <a href="https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html">AWS X-Ray </a>generate large volumes of distributed tracing data across multiple services. Static sampling strategies keep costs down by capturing a fixed percentage of traffic. However, they frequently miss critical data during intermittent failures or sudden latency spikes. Tracing every request for maximum visibility at scale may increase sampling costs for your organization.</p> 
<p><a href="https://docs.aws.amazon.com/xray/latest/devguide/xray-adaptive-sampling.html">Adaptive sampling</a> helps you control and predict costs during active incidents, latency spikes, or fault conditions. It dynamically adjusts sampling behavior based on runtime conditions, enabling you to capture more relevant traces and spans when anomalies occur.</p> 
<p>This post describes how adaptive sampling works in AWS X-Ray, walks through configuration for specific use cases, and demonstrates how to capture critical diagnostic data efficiently. By the end of this post, you will know how to create sampling rules that prioritize high-value traces by keeping sampling costs under control.</p> 
<h1>Prerequisites</h1> 
<ul> 
 <li>Familiarity with X-Ray concepts including traces, segments, and sampling.</li> 
 <li>Familiarity with <a href="https://aws.amazon.com/cloudwatch/pricing/">Amazon CloudWatch pricing</a> for <a href="https://aws.amazon.com/cloudwatch/features/application-observability-apm/">application observability</a>.</li> 
 <li>Enabled <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Monitoring-Sections.html">CloudWatch Application Signals</a> for the supported services before proceeding.</li> 
 <li>The root services (entry points) in an application must run on supported compute service.</li> 
 <li>Using <a href="https://aws.amazon.com/otel/">AWS Distro for OpenTelemetry</a> (ADOT) Software Development Kit (SDK) for Java version 2.11.5/Python version 0.15.0 or higher.</li> 
 <li>Integration with the ADOT SDK and executed together with either the Amazon CloudWatch Agent or the OpenTelemetry Collector.</li> 
 <li>For viewing and querying traces, using <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Transaction-Search.html">Transaction Search</a> is recommended.</li> 
</ul> 
<h1>Sampling rules vs Adaptive sampling</h1> 
<p>Static sampling and adaptive sampling can be used independently or together. Understanding this behavior is critical when designing sampling rules. The following is a brief overview of both features:</p> 
<p><strong>Static sampling</strong>: Traditional X-Ray sampling relies on static rules that define a fixed sampling rate, a reservoir quota, and matching conditions. Although effective for cost control, static sampling does not react to runtime anomalies and can result in missing traces during short-lived failures.</p> 
<p><strong>Adaptive sampling</strong>: Adaptive sampling builds on existing sampling rules and introduces dynamic behavior through two complementary mechanisms:</p> 
<ol> 
 <li><strong>Sampling Boost</strong>, which automatically increases sampling rates when the ADOT SDK detects anomalies.</li> 
 <li><strong>Anomaly Span Capture</strong>, which is designed to capture critical spans even when full traces are not sampled. The ADOT SDK performs anomaly detection locally per conditions configured in the application environment.</li> 
</ol> 
<h1>Solution Overview</h1> 
<p>X-Ray relies entirely on parent-based sampling, and the root services (first instrumented service) make the sampling decisions. Downstream services cannot override an upstream sampling decision. A sampling rule targeting a non-root service only takes effect if no upstream decision has already been made.</p> 
<p>Let’s consider an example of an application that will be used to demonstrate the use cases in this post. For demonstration purposes, this example application is deployed on <a href="https://aws.amazon.com/pm/ec2/">Amazon Elastic Compute Cloud</a> (EC2) instances with an <a href="https://aws.amazon.com/otel/">OpenTelemetry</a> Collector. The services are configured with Application Signals feature to automatically collect metrics and traces from the applications.</p> 
<div class="wp-caption aligncenter" id="attachment_64231" style="width: 1173px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure1.png" rel="noopener" target="_blank"><img alt="Figure 1: Application connectivity flow with three services and a database." class="wp-image-64231 size-full" height="226" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure1.png" width="1163" /></a>
 <p class="wp-caption-text" id="caption-attachment-64231">Figure 1: Application connectivity flow with three services and a database.</p>
</div> 
<p><strong>Root service</strong>: Service A is the root service. Sampling rules configuration starts at the root service. A supported SDK version is required to enable sampling boost. Else, it cannot trigger a sampling boost.</p> 
<p><strong>Downstream services</strong>: Service B is a downstream service to Service A. Service C is a downstream service to Service B. They can report anomalies and trigger sampling boost at root service but cannot independently make sampling decisions. If downstream services are configured with a supported SDK version, they can trigger a sampling boost at the root service during anomalies.</p> 
<p><strong>Region and account constraints</strong>: In this example, all services are in the same AWS account and Region. If the downstream services span multiple AWS accounts and Regions, you can capture anomaly spans using local SDK configuration, but they cannot trigger sampling boost.</p> 
<p><strong>Local configuration to ADOT SDK for adaptive sampling</strong>: The following environment variable YAML Ain’t Markup Language (YAML) configuration shows the local configuration applied to the ADOT SDK for Service C:</p> 
<pre><code class="lang-yaml">AWS_XRAY_ADAPTIVE_SAMPLING_CONFIG="{version: 1.0, anomalyConditions: [{errorCodeRegex: \"^(500|501)$\", usage: \"both\"}, {highLatencyMs: 100, usage: \"sampling-boost\"}], anomalyCaptureLimit: {anomalyTracesPerSecond: 1}}"</code></pre> 
<p>This configuration triggers sampling boost and anomaly span capture when the service returns HTTP 500 and 501 faults. The <code>usage</code> field controls whether the configuration triggers sampling boost, anomaly span capture, or both. It also triggers sampling boost when latency exceeds 100 ms.</p> 
<p>To capture error spans during server-side issues, <code>anomalyTracesPerSecond</code> is configured 1. This configuration captures 1 trace per second and prevents publishing all similar error traces (cost protection).</p> 
<p><strong>Note: Anomaly spans are partial traces and cannot influence the sampling boost (full end-to-end trace). For more details refer to <a href="https://docs.aws.amazon.com/xray/latest/devguide/xray-adaptive-sampling.html#local-sdk-configuration">Local SDK configuration</a>.&nbsp;</strong></p> 
<p><strong>Sampling rule for the root service: </strong>The following JSON shows the test sampling rule that is configured for Service A. The console view is shown in figure 2 and figure 3 below.</p> 
<pre><code class="lang-json">{
  "RuleName": "test",
  "Priority": 1,
  "ReservoirSize": 0,
  "FixedRate": 0.01,
  "ServiceName": "ServiceA",
  "ServiceType": "*",
  "Host": "*",
  "HTTPMethod": "*",
  "URLPath": "*",
  "SamplingRateBoost": {
    "MaxRate": 0.80,
    "CooldownWindowMinutes": 1
  }
}</code></pre> 
<div class="wp-caption aligncenter" id="attachment_64230" style="width: 1139px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure2.png" rel="noopener" target="_blank"><img alt="Figure 2: Console view of the adaptive sampling rule configuration" class="wp-image-64230 size-full" height="378" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure2.png" width="1129" /></a>
 <p class="wp-caption-text" id="caption-attachment-64230">Figure 2: Console view of the adaptive sampling rule configuration</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_64229" style="width: 1142px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure3.png" rel="noopener" target="_blank"><img alt="Figure 3: Console view of sampling boost configuration" class="wp-image-64229 " height="310" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure3.png" width="1132" /></a>
 <p class="wp-caption-text" id="caption-attachment-64229">Figure 3: Console view of sampling boost configuration</p>
</div> 
<p>This rule uses <code>Priority 1</code>, which means X-Ray evaluates it before any lower-priority rules. It is configured with <code>ReservoirSize</code> to 0 and <code>FixedRate</code> to 0.01 (1%). This samples 1% of all requests and provides no guaranteed minimum per second. To capture a minimum number of traces per second, you can increase the <code>ReservoirSize</code> value.</p> 
<p>The <code>ServiceName</code> field matches this rule to Service A only. The <code>Host</code>, <code>HTTPMethod</code> and <code>URLPath</code> fields are set to <code>*</code>, which means the rule applies to all hosts and URL paths for that service. You can narrow these fields to target specific endpoints or hostnames when needed.</p> 
<p>This rule enables <code>SamplingRateBoost</code> with a <code>MaxRate</code> of 0.80 (80%) and <code>CooldownWindowMinutes</code> of 1. When the boost is triggered, up to 80% of traces are sampled.</p> 
<p>The <code>CooldownWindowMinutes</code> parameter controls how frequently boosts can occur. A 1-minute <code>CooldownWindow</code> allows continuous boosting during persistent anomalies. This setting is appropriate for critical services where complete incident visibility justifies higher costs.</p> 
<p>You can configure the <code>MaxRate</code> parameter to define the upper limit for sampling during anomalies. Set this value based on your maximum acceptable cost and the level of visibility you need during incidents. For example, a <code>MaxRate</code> of 0.25 (25%) provides substantial coverage during anomalies without excessive costs.</p> 
<p>This way X-Ray applies sampling rules to determine which requests to get traced. You can modify the default rule or configure additional rules (with <code>Priority</code> order) that determine which requests to get traced based on the properties of the service or request. For more details on sampling rules configuration, refer to <a href="https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html">Configuring sampling rules</a> documentation.</p> 
<h1>Managing adaptive sampling for different use cases</h1> 
<h2>Use case 1: Boost sampling when X-Ray detects anomalies</h2> 
<p>Use sampling boost in environments that use conservative baseline sampling and need improved visibility into recurring anomalies without increasing overall tracing cost. It dynamically increases the sampling rate when anomalies are detected. Sampling boost uses a probabilistic model that evaluates how likely it is to capture at least one anomalous trace (end-to-end trace) when anomalies repeat over a short period of time.</p> 
<h4>How it works</h4> 
<p>X-Ray observes the number of anomalies detected within an evaluation window and estimates how much the sampling rate needs to increase so that one or more anomalous requests are likely to be traced. The more frequently an anomaly occurs, the lower the sampling rate increase required to achieve this goal. Conversely, when anomalies occur infrequently, a higher sampling rate is needed to improve the likelihood of capture.</p> 
<p>X-Ray dynamically adjusts the sampling rate toward the minimum level that provides sufficient confidence of capturing an anomalous trace, while always respecting the maximum sampling rate configured in the sampling rule. This approach balances trace coverage and cost by increasing sampling only when repeated anomalies indicate that additional trace data is likely to be valuable.</p> 
<p>By default, the sampling boost is driven by HTTP 5XX faults observed across services in the call chain. To treat other conditions such as elevated latency as anomalies, you define those conditions through local SDK configuration.</p> 
<p>Consider the example application with three services described earlier. The example application had no anomalies and, with the configured <code>FixedRate</code> of 1%, it may not capture traces every minute, as shown in the following figure 4.</p> 
<div class="wp-caption aligncenter" id="attachment_64228" style="width: 2566px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure4.png" rel="noopener" target="_blank"><img alt="Figure 4: Spans are not captured with FixedRate of 1% and ReservoirSize 0" class="wp-image-64228 size-full" height="822" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure4.png" width="2556" /></a>
 <p class="wp-caption-text" id="caption-attachment-64228">Figure 4: Spans are not captured with FixedRate of 1% and ReservoirSize 0</p>
</div> 
<p>By default, when faults are detected under Service A, X-Ray automatically triggers the sampling boost based on the configured rule. This example generated high latency spikes at Service C to trigger the sampling boost at Service A. Because Service C is a downstream service to Service A and has the supported configuration, it can trigger the sampling boost.</p> 
<h4>Testing by generating latency spikes</h4> 
<p>This test generated 8 high latency requests per minute for the first 40 seconds, sustained over 10 minutes on Service C. The increase in latency, triggered the sampling boost at Service A to capture full traces. The following figure 5 shows X-Ray sampling traces during high latency.</p> 
<ul> 
 <li><code>aws.trace.flag.sampled = 1</code> indicates that X-Ray captured a full trace with the sampling boost.</li> 
 <li><code>aws.trace.flag.sampled = 0</code> does not appear because anomaly span capture is not triggered.</li> 
</ul> 
<div class="wp-caption aligncenter" id="attachment_64226" style="width: 2564px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure5.png" rel="noopener" target="_blank"><img alt="Figure 5: X-Ray sampling boost results during high latency" class="wp-image-64226 size-full" height="909" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure5.png" width="2554" /></a>
 <p class="wp-caption-text" id="caption-attachment-64226">Figure 5: X-Ray sampling boost results during high latency</p>
</div> 
<h4><strong>Key characteristics of Sampling Boost</strong></h4> 
<ul> 
 <li>Configured as part of an X-Ray sampling rule.</li> 
 <li>Boost magnitude is calculated based on the number of anomalies observed.</li> 
 <li>Each boost is temporary and followed by a <code>CooldownWindow</code> period.</li> 
 <li>Sampling rate is increased only as much as needed to capture at least one anomalous trace, up to a configured maximum.</li> 
 <li>Define sampling rules per each root service to achieve more precise and predictable boost behavior.</li> 
 <li>Because this mechanism is probability-based, sampling boost is most effective for recurring anomalies or anomalies that last longer.</li> 
</ul> 
<h2>Use case 2: Capture spans on demand when an anomaly condition is met</h2> 
<p>Anomaly Span Capture is a complementary capability that operates independently of X-Ray sampling rules. Unlike sampling boost, it doesn’t rely on sampling roots or parent-based sampling decisions. This mechanism is designed to record anomalies whenever they occur, based on conditions defined in local configuration.</p> 
<h4>How it works</h4> 
<p>A service can define an anomaly condition locally, such as a latency threshold being exceeded for a specific operation. When a request violates that condition, X-Ray captures the span chain for that service from the service’s root span, through the span where the high latency is observed. Although the captured trace covers a single service, it provides detailed visibility into the execution path surrounding the anomaly and may be sufficient to identify the root cause.</p> 
<p><strong>Using Sampling Boost and Anomaly Span Capture together</strong></p> 
<p>By combining sampling boost and anomaly span capture, you can lower baseline sampling rates to reduce cost, capture anomalies deterministically, and collect additional traces opportunistically when issues persist. This combination provides strong diagnostic coverage without requiring high steady-state trace volume.</p> 
<p><strong>Testing by generating faults</strong></p> 
<p>This test generated faults to trigger sampling-boost and anomaly-span-capture (<code>usage: \”both“\</code>). It generated 8 faults per minute for the first 40 seconds, sustained over 10 minutes at Service C. This triggered sampling boost at Service A (full traces) and anomaly span capture at Service C (partial traces). When the ADOT SDK detects an anomaly span, it emits as many spans as possible, up to the limit set by <code>anomalyTracesPerSecond</code> in the local configuration. The following figure 6 shows X-Ray sampling traces during faults:</p> 
<ul> 
 <li><code>aws.trace.flag.sampled = 0</code> indicates anomaly span capture.</li> 
 <li><code>aws.trace.flag.sampled = 1</code> indicates that X-Ray captured a full trace with sampling boost.</li> 
</ul> 
<div class="wp-caption aligncenter" id="attachment_64225" style="width: 2561px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure6.png" rel="noopener" target="_blank"><img alt="Figure 6: X-Ray sampling results showing both sampling boost and anomaly span capture" class="wp-image-64225 size-full" height="896" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure6.png" width="2551" /></a>
 <p class="wp-caption-text" id="caption-attachment-64225">Figure 6: X-Ray sampling results showing both Sampling Boost and Anomaly Span Capture</p>
</div> 
<p><strong>Key characteristics of Anomaly Span Capture</strong></p> 
<ul> 
 <li>Configured locally at the SDK level using a YAML configuration file.</li> 
 <li>Independent of sampling rules and sampling roots.</li> 
 <li>Captures spans on demand rather than through probabilistic sampling.</li> 
 <li>Includes all anomalous spans during an anomaly.</li> 
 <li>Produces a partial trace scoped to one service, which provides highly actionable debugging information.</li> 
</ul> 
<h1>Monitoring Sampling boost</h1> 
<p>Sampling Boost respects configured maximum rates and cooldown windows, which helps maintain predictable costs. When you define a sampling rule with sampling boost enabled, X-Ray automatically emits metrics with <strong>SamplingRate</strong> as the metric name. For every rule with <code>SamplingRateBoost</code> enabled, this metric is emitted. During the test described earlier, sampling boost triggered twice: once during a latency spike and once during 5XX faults at Service C. The following figure shows X-Ray boosting the sampling rate to 28% during faults and up to 80% (the configured maximum) during the latency spike, designed to capture at least one trace during application issues.</p> 
<div class="wp-caption aligncenter" id="attachment_64224" style="width: 3066px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure7.png" rel="noopener" target="_blank"><img alt="Figure 7: X-Ray SamplingRate metric showing boost during faults and latency spike for the test rule" class="wp-image-64224 size-full" height="930" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure7.png" width="3056" /></a>
 <p class="wp-caption-text" id="caption-attachment-64224">Figure 7: X-Ray SamplingRate metric showing boost during faults and latency spike for the test rule</p>
</div> 
<h1>Best Practices</h1> 
<p><strong>Sampling rules at root services</strong>: A wildcard root service aggregates anomaly counts across all matched request paths. The calculated sampling boost satisfies the minimum requirement at an aggregate level but may not capture anomalies for each individual path. For more predictable behavior, define sampling rules per root service or per critical entry point.</p> 
<div class="wp-caption aligncenter" id="attachment_64223" style="width: 2240px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure8.png" rel="noopener" target="_blank"><img alt="Figure 8: Aggregated sampling behavior across multiple paths" class="wp-image-64223 size-full" height="1378" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure8.png" width="2230" /></a>
 <p class="wp-caption-text" id="caption-attachment-64223">Figure 8: Aggregated sampling behavior across multiple paths</p>
</div> 
<p><strong>Baseline sampling rate selection: </strong>Adaptive sampling provides the largest benefit when baseline sampling rates are low. Sampling boost improves the probability of capturing repeated anomalies within a limited evaluation window, not one-off events. A low baseline rate (for example, <code>FixedRate</code> of 1%) benefits most from a temporary boost, while a high baseline rate (for example, <code>FixedRate</code> of 30%) offers limited additional benefit.</p> 
<div class="wp-caption aligncenter" id="attachment_64222" style="width: 1950px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure9.png" rel="noopener" target="_blank"><img alt="Figure 9: Sampling boost benefit for low and high baselines" class="wp-image-64222 size-full" height="1012" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure9.png" width="1940" /></a>
 <p class="wp-caption-text" id="caption-attachment-64222">Figure 9: Sampling boost benefit for low and high baselines</p>
</div> 
<p><strong>Cooldown period</strong>: Sampling boost is intentionally designed to be temporary. The boost window is fixed at 1-minute by default. After the boost completes, X-Ray enters a cooldown phase using fixed, time-aligned windows rather than a rolling timer. A 10-minute cooldown prevents continuous elevated sampling during prolonged incidents, helping you maintain predictable costs.</p> 
<p>For example, with a 10-minute <code>CooldownWindow</code>, if a boost ends at 1:14, no new boost can be triggered before 1:20. Configuring the <code>CooldownWindow</code> to one minute allows sampling boost to occur continuously during persistent anomalies.</p> 
<div class="wp-caption aligncenter" id="attachment_64221" style="width: 2910px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure10.png" rel="noopener" target="_blank"><img alt="Figure 10: Time series view of cooldown period" class="wp-image-64221 size-full" height="736" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure10.png" width="2900" /></a>
 <p class="wp-caption-text" id="caption-attachment-64221">Figure 10: Time series view of cooldown period</p>
</div> 
<p><strong>Multi account and multi region configurations: </strong>X-Ray propagates sampling decisions to downstream services regardless of account boundaries. Anomaly signals only trigger sampling boost if they occur within the same AWS account and AWS Region as the root service sampling rule. However, if sampling boost is already active, the increased sampling rate applies to all downstream services in the same trace call chain.</p> 
<div class="wp-caption aligncenter" id="attachment_64220" style="width: 2516px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure11.png" rel="noopener" target="_blank"><img alt="Figure 11: Multi account sampling boost and anomaly span capture behavior" class="wp-image-64220 size-full" height="1328" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/06/cloudops-2172-Figure11.png" width="2506" /></a>
 <p class="wp-caption-text" id="caption-attachment-64220">Figure 11: Multi account Sampling Boost and Anomaly Span Capture behavior</p>
</div> 
<h1>Conclusion</h1> 
<p>This post demonstrates how to configure adaptive sampling with X-Ray to capture critical spans while balancing observability depth and cost. X-Ray adaptive sampling helps you respond dynamically to runtime anomalies by combining Sampling Boost with Anomaly Span Capture. This helps collect critical diagnostic data without increasing steady-state trace volume and cost.</p> 
<p>To learn more about X-Ray, visit the <a href="https://docs.aws.amazon.com/xray/">AWS X-Ray</a> documentation. Explore Amazon CloudWatch <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Monitoring-Intro.html">Application Performance Monitoring</a> to view and query traces captured. Review <a href="https://aws-observability.github.io/observability-best-practices/">observability best practices</a> for additional information. Have you implemented adaptive sampling in your environment? Share your experience in the comments below.</p>
