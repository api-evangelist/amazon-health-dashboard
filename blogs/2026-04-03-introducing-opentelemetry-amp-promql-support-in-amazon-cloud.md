---
title: "Introducing OpenTelemetry & AMP; PromQL support in Amazon CloudWatch"
url: "https://aws.amazon.com/blogs/mt/introducing-opentelemetry-promql-support-in-amazon-cloudwatch/"
date: "Fri, 03 Apr 2026 17:29:51 +0000"
author: "Rodrigue Koffi"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<p>If you run Kubernetes or microservices workloads on AWS, your metrics likely carry dozens of labels: namespace, pod, container, node, deployment, replica set, and custom business dimensions. To get a complete picture of your environment, you may be splitting your metrics pipeline: Amazon CloudWatch for AWS metrics, and a separate Prometheus-compatible backend for high-cardinality (many unique label combinations) container and application metrics. Some teams go further. They use Prometheus CloudWatch exporters to pull AWS resource metrics into their Prometheus backend through <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html">GetMetricData</a> API calls. This adds operational overhead and cost but lets them query everything in one place.</p> 
<p>Amazon CloudWatch now natively ingests <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-OpenTelemetry-Sections.html">OpenTelemetry metrics</a> and supports querying them with <a href="https://prometheus.io/docs/prometheus/latest/querying/basics/">Prometheus Query Language (PromQL)</a>. This preview capability introduces a high-cardinality metrics store that supports up to 150 labels per metric, so you can send rich, label-dense metrics directly to CloudWatch without conversion or truncation. Combined with automatic AWS vended metric enrichment, CloudWatch becomes a single destination for infrastructure, container, and application metrics, all queryable with PromQL.</p> 
<p>In this post, you will learn how to:</p> 
<ul> 
 <li>Enable OpenTelemetry metrics ingestion and automatic AWS resource enrichment in your account</li> 
 <li>Deploy Amazon CloudWatch Container Insights on an Amazon Elastic Kubernetes Service (Amazon EKS) cluster</li> 
 <li>Query infrastructure and AWS resource metrics using PromQL in Amazon CloudWatch and Amazon Managed Grafana</li> 
 <li>Create custom application metrics in Amazon CloudWatch using the OpenTelemetry SDK and see them automatically enriched with AWS context</li> 
</ul> 
<h1>What OpenTelemetry support means for Amazon CloudWatch</h1> 
<p>The OpenTelemetry Protocol (OTLP) is the standard wire protocol for the OpenTelemetry (OTel) project. It defines how telemetry data, including metrics, traces, and logs, is encoded and transported between components. With this preview, CloudWatch exposes a regional OTLP endpoint that OpenTelemetry-compatible collectors or SDKs can send metrics to.</p> 
<p>CloudWatch receives the metrics and stores them in a new high-cardinality metrics store, retaining OpenTelemetry metric types, including counters, histograms, gauges, and up-down counters, without conversion. With this launch, CloudWatch completes its support for OpenTelemetry across all three pillars of observability. CloudWatch already accepts traces and logs through its <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-OTLPEndpoint.html">OTLP endpoints</a>, adding native OTLP metrics ingestion means you can now send all your telemetry to CloudWatch using open standards, through a single protocol. Three capabilities make this significant:</p> 
<p><strong>Extended label and cardinality support.</strong> OTLP-ingested metrics support up to 150 labels, compared to the 30-dimension limit of CloudWatch custom metrics. This removes a key constraint for Kubernetes, microservice, and OpenTelemetry workloads that rely on high-cardinality labels for filtering and aggregation. Visit the <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_limits.html">quotas page</a> to stay up to date as these limits continue to evolve.</p> 
<p><strong>PromQL query support.</strong> You can query metrics ingested through OTLP using PromQL. If you already use Prometheus, you can use the same query language directly in CloudWatch and Amazon Managed Grafana, no new syntax to learn.</p> 
<p><strong>Automatic AWS resource enrichment.</strong> This capability fundamentally changes how you query and filter metrics across your AWS infrastructure. CloudWatch enriches every ingested metric with AWS resource context: account ID, Region, cluster Amazon Resource Name (ARN), and resource tags from AWS Resource Explorer. This enrichment happens automatically, without additional instrumentation. You can filter and group metrics by AWS account, Region, environment tag, or application name, whether they come from Container Insights, a custom application, or an AWS service. No exporters, no custom labels, no additional API calls.</p> 
<div class="wp-caption aligncenter" id="attachment_64440" style="width: 2870px;">
 <img alt="Architecture diagram showing two metric ingestion paths into Amazon CloudWatch. Applications and collectors inside an AWS account and outside AWS send OpenTelemetry metrics to the CloudWatch OTLP and PromQL store. AWS resources including AWS Lambda, Amazon EC2, and Amazon RDS send vended metrics through PutMetricData and GetMetricData. An enrichment layer connects both stores, adding AWS resource context to OpenTelemetry metrics." class="wp-image-64440 size-full" height="1646" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/02/zeus-hld-1.png" width="2860" />
 <p class="wp-caption-text" id="caption-attachment-64440">Figure 1: OpenTelemetry metrics ingestion and enrichment architecture in Amazon CloudWatch.</p>
</div> 
<h1>Enable OTLP ingestion and AWS resource enrichment</h1> 
<p>Before you can ingest and query OTLP metrics, enable two account-level settings. The first enables resource tags propagation from your AWS resources to your telemetry, the same tags you see in AWS Resource Explorer. The second enables OTLP ingestion for CloudWatch.</p> 
<p>You can enable both enrichment settings from the Amazon CloudWatch console or using the AWS CLI.</p> 
<h2>Using the console</h2> 
<p>To enable enrichment in the CloudWatch console, follow these steps:</p> 
<ol> 
 <li>Open the Amazon CloudWatch console.</li> 
 <li>In the navigation pane, choose <strong>Settings</strong>.</li> 
 <li>Enable resource tags on telemetry.</li> 
 <li>Enable OTel enrichment for AWS metrics.</li> 
</ol> 
<p>After you enable both settings, your account is ready to receive OTLP metrics at the regional endpoint.</p> 
<div class="wp-caption aligncenter" id="attachment_64396" style="width: 2496px;">
 <img alt="Architecture diagram showing the OTLP metrics ingestion flow from an Amazon EKS cluster through the CloudWatch agent to the CloudWatch OTLP endpoint, with enrichment layers and PromQL query paths" class="size-full wp-image-64396" height="1434" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/01/otel-enrichment-console.png" width="2486" />
 <p class="wp-caption-text" id="caption-attachment-64396">Figure 2: Enabling OTel enrichment and resource tags in the CloudWatch console settings</p>
</div> 
<h2>Using the AWS CLI</h2> 
<p>Alternatively, use the AWS CLI to enable both enrichment layers. Run the following commands:</p> 
<pre><code class="lang-bash">
# Enable resource tags on telemetry
aws observabilityadmin start-telemetry-enrichment
# Enable OTel enrichment for CloudWatch
aws cloudwatch start-otel-enrichment
</code></pre> 
<p>To verify that both enrichment settings are active, run the following command:</p> 
<pre><code class="lang-bash">
aws cloudwatch get-otel-enrichment-status
</code></pre> 
<p>With enrichment enabled, every metric ingested through the OTLP endpoint is automatically tagged with AWS resource context. The following table shows the attributes that CloudWatch adds:</p> 
<table> 
 <tbody> 
  <tr> 
   <th>Attribute</th> 
   <th>Description</th> 
   <th>Example</th> 
  </tr> 
  <tr> 
   <td><strong>@aws.account</strong></td> 
   <td>AWS account ID</td> 
   <td><code>123456789012</code></td> 
  </tr> 
  <tr> 
   <td><strong>@aws.region</strong></td> 
   <td>AWS Region</td> 
   <td><code>us-west-2</code></td> 
  </tr> 
  <tr> 
   <td><strong>cloud.resource_id</strong></td> 
   <td>Full EKS cluster ARN</td> 
   <td><code>arn:aws:eks:us-west-2:123456789012:cluster/prod</code></td> 
  </tr> 
  <tr> 
   <td><strong>k8s.cluster.name</strong></td> 
   <td>EKS cluster name</td> 
   <td><code>production-cluster</code></td> 
  </tr> 
  <tr> 
   <td><strong>k8s.namespace.name</strong></td> 
   <td>Kubernetes namespace</td> 
   <td><code>karpenter</code></td> 
  </tr> 
  <tr> 
   <td><strong>k8s.container.name</strong></td> 
   <td>Container name</td> 
   <td><code>controller</code></td> 
  </tr> 
  <tr> 
   <td><strong>@instrumentation.name</strong></td> 
   <td>Instrumentation source</td> 
   <td><code>cloudwatch-otel-ci</code></td> 
  </tr> 
  <tr> 
   <td><strong>Resource tags</strong></td> 
   <td>Tags from AWS Resource Explorer (@aws.tag.Application, @aws.tag.CostCenter, @resource.ec2.tag.ManagedBy, …)</td> 
   <td><code>env=production</code></td> 
  </tr> 
 </tbody> 
</table> 
<p>These attributes are added by CloudWatch with no manual instrumentation required. This is what makes it possible to query across AWS accounts, Regions, and resource tags without building custom pipelines or running exporters.</p> 
<h1>Amazon CloudWatch Container Insights with OpenTelemetry metrics</h1> 
<p>To see OpenTelemetry with CloudWatch in action, let’s start with Container Insights. Amazon CloudWatch Container Insights for Amazon EKS <a href="https://aws.amazon.com/about-aws/whats-new/2026/04/cloudwatch-otel-container-insights-eks/">now supports</a> Prometheus and OpenTelemetry metrics. This standardizes container metrics with OpenTelemetry attributes and makes them queryable with PromQL. You can <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/container-insights-otel-metrics.html">enable Container Insights</a> using the Amazon EKS add-on through the console or the AWS CLI.</p> 
<h2>Container Insights dashboards</h2> 
<p>After Container Insights is deployed, CloudWatch automatically creates dashboards showing cluster-level metrics including CPU utilization, memory usage, and pod counts. To view these dashboards, open the CloudWatch console, choose <strong>Container Insights</strong> from the navigation pane, and select your cluster from the dropdown. You can switch between cluster, namespace, and pod-level views to drill into specific workloads.</p> 
<div class="wp-caption aligncenter" id="attachment_64431" style="width: 3354px;">
 <img alt="Amazon CloudWatch Container Insights dashboard with the EKS OTEL service selected, showing cluster state summary for eks-test-cluster with 4% CPU and 4% memory utilization, 150 desired and 140 ready pods across 20 available nodes, and control plane metrics including API server requests and latency." class="size-full wp-image-64431" height="1566" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/02/eks-cwci.png" width="3344" />
 <p class="wp-caption-text" id="caption-attachment-64431">Figure 3: Amazon CloudWatch Container Insights dashboard</p>
</div> 
<h1>Query metrics using PromQL with CloudWatch Query Studio</h1> 
<p>You can query OTLP-ingested metrics using PromQL in the CloudWatch console, Amazon Managed Grafana, or query interfaces that supports PromQL and AWS Signature Version 4 (SigV4). <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-PromQL-QueryStudio.html">CloudWatch Query Studio</a> provides a built-in PromQL editor for exploring and visualizing these metrics directly in the console. Select the PromQL query mode to get started.</p> 
<div class="wp-caption aligncenter" id="attachment_64409" style="width: 3082px;">
 <img alt="CloudWatch Query Studio interface with PromQL query mode selected" class="size-full wp-image-64409" height="1634" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/01/Screenshot-2026-04-01-at-14.38.09.png" width="3072" />
 <p class="wp-caption-text" id="caption-attachment-64409">Figure 4: Amazon CloudWatch Query Studio interface with PromQL query mode</p>
</div> 
<h2>Queries using enriched AWS resource context</h2> 
<p>Because enrichment is enabled, you can query across AWS resource boundaries using the tags that CloudWatch adds automatically. No exporters, no custom labels:</p> 
<pre><code class="lang-bash">
# AWS Lambda function duration for functions tagged with application "order-pipeline"
Duration{"@aws.tag.appname"="order-pipeline"}

# Amazon EC2 CPU utilization for production delivery workloads
CPUUtilization{"@aws.tag.Environment"="production", "@aws.tag.Application"="delivery"}

# Running pods grouped by AWS account and namespace
sum by (aws_account_id, k8s_namespace_name) (kube_pod_status_phase{phase="Running"})
</code></pre> 
<p>The last query returns the count of running pods grouped by AWS account and Kubernetes namespace without any custom instrumentation. The <code>aws_account_id</code> label is added automatically by the enrichment layer.</p> 
<div class="wp-caption aligncenter" id="attachment_64404" style="width: 3436px;">
 <img alt="CloudWatch Query Studio showing a PromQL query filtering Lambda duration metrics by resource tag." class="size-full wp-image-64404" height="1530" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/01/Screenshot-2026-04-01-at-15.12.07.png" width="3426" />
 <p class="wp-caption-text" id="caption-attachment-64404">Figure 5: CloudWatch Query Studio querying Lambda duration metrics</p>
</div> 
<h1>Query metrics using PromQL in Grafana</h1> 
<p>To visualize OTLP-ingested metrics in Amazon Managed Grafana, add a Prometheus data source that points to the CloudWatch PromQL endpoint. This section walks through configuring the data source with AWS Signature Version 4 (SigV4) authentication.</p> 
<ol> 
 <li>Open your Amazon Managed Grafana workspace.</li> 
 <li>Choose <strong>Data Sources</strong>.</li> 
 <li>Choose <strong>Add data source</strong>.</li> 
 <li>Select <strong>Prometheus</strong> as the data source type.</li> 
 <li>For the URL, enter the CloudWatch PromQL endpoint for your Region: <code>https://monitoring.&lt;AWS Region&gt;.amazonaws.com/v1/metrics</code></li> 
 <li>Under <strong>Authentication</strong>, select <strong>SigV4</strong>.</li> 
 <li>Configure the appropriate IAM role for SigV4 authentication.</li> 
 <li>Choose <strong>Save &amp; Test</strong> to verify the connection.</li> 
</ol> 
<blockquote>
 <p>If Save &amp; Test succeeds, you will see a “Data source is working” confirmation. If it fails, verify that the IAM role has <code>cloudwatch:GetMetricData</code> and <code>cloudwatch:ListMetrics</code> permissions, and that SigV4 signing is properly configured.</p>
</blockquote> 
<p>After the data source is configured, you can use the same PromQL queries in Grafana dashboards.</p> 
<div class="wp-caption aligncenter" id="attachment_64412" style="width: 2326px;">
 <img alt="Amazon Managed Grafana Explore view showing a PromQL query for container CPU usage rate with enriched AWS labels available as filters" class="size-full wp-image-64412" height="1034" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/01/grafana-query.png" width="2316" />
 <p class="wp-caption-text" id="caption-attachment-64412">Figure 6: Grafana Explore with CloudWatch PromQL</p>
</div> 
<h1>Custom application metrics</h1> 
<p>CloudWatch OTLP ingestion also supports custom application metrics. Applications instrumented with an OpenTelemetry SDK can send metrics through the CloudWatch agent running in your cluster with no changes to your instrumentation code required.</p> 
<p>To see this in action, deploy a sample Python application from the <a href="https://github.com/aws-observability/aws-otel-community">aws-otel-community</a> repository. The application uses the OpenTelemetry Python SDK to emit custom metrics covering all OTel metric types: counters, histograms, gauges, and up-down counters. For example, the app defines a <code>latency_time</code> histogram that measures API response times:</p> 
<pre><code class="lang-python">
from opentelemetry import metrics
meter = metrics.get_meter(__name__)

# Histogram --- measures API latency distribution
latency_time = meter.create_histogram(
  name="latency_time",
  description="Measures latency time",
  unit="ms",
)
</code></pre> 
<h2>Deploy the sample application</h2> 
<p>Find the <a href="https://github.com/aws-observability/aws-otel-community.git">sample application</a> and all deployment manifests in the aws-otel-community repository on GitHub. The Container Insights add-on you deployed earlier includes a CloudWatch agent that acts as an OpenTelemetry collector. Point the sample app to it by setting the <code>OTEL_EXPORTER_OTLP_ENDPOINT</code> environment variable: <code>http://cloudwatch-agent.amazon-cloudwatch.svc.cluster.local:4317</code>.</p> 
<p>This walkthrough uses the CloudWatch agent, but you can use any OpenTelemetry-compatible collector or SDK that supports OTLP/HTTP to send metrics directly to the CloudWatch OTLP endpoint.</p> 
<h2>Query application metrics with PromQL</h2> 
<p>After deploying the application, open CloudWatch Query Studio or in your Amazon Managed Grafana workspace, navigate to Explore, and select the CloudWatch PromQL data source.</p> 
<p>The following query shows the p99 latency for the demo app in Amazon Managed Grafana, grouped by the automatically enriched <code>@aws.region</code> label:</p> 
<pre><code>
histogram_quantile(0.99, sum by (le, aws_region) (rate(latency_time_bucket{resource_service_name="python-demo-app"}[5m])))`
</code></pre> 
<div class="wp-caption aligncenter" id="attachment_64426" style="width: 3032px;">
 <img alt="Amazon Managed Grafana Explore view showing a PromQL histogramquantile query for p99 latencytime of the python-demo-app, grouped by the enriched @aws.region label, with results displayed as a bar chart over time." class="wp-image-64426 size-full" height="1498" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/02/Screenshot-2026-04-02-at-10.28.18.png" width="3022" />
 <p class="wp-caption-text" id="caption-attachment-64426">Figure 7: P99 latency for the sample application in Amazon Managed Grafana</p>
</div> 
<p>Because enrichment is enabled, every application metric is automatically enriched with AWS resource context. For example, querying <code>cpu_usage</code> returns these labels without any additional instrumentation:</p> 
<div class="wp-caption aligncenter" id="attachment_64427" style="width: 3034px;">
 <img alt="Amazon Managed Grafana Explore view showing custom application metrics from the Python demo app with automatically enriched AWS resource labels." class="wp-image-64427 size-full" height="1350" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/02/Screenshot-2026-04-02-at-10.17.57.png" width="3024" />
 <p class="wp-caption-text" id="caption-attachment-64427">Figure 8: Showing all enriched labels from custom OTel instrumentation</p>
</div> 
<h1>Pricing</h1> 
<p>The OTLP ingestion capability and PromQL queries are available at no additional cost during the preview period. For current pricing details, see the Amazon CloudWatch <a href="https://aws.amazon.com/cloudwatch/pricing/">Pricing page</a>.</p> 
<p>Amazon EKS and Amazon Managed Grafana resources used in this walkthrough are charged at standard rates. To avoid ongoing charges, follow the clean up in the following section after you finish the walkthrough.</p> 
<h1>Clean up</h1> 
<ul> 
 <li>Delete the sample application:</li> 
</ul> 
<pre><code class="lang-bash">
kubectl delete -f demo-app.yaml
</code></pre> 
<ul> 
 <li>Remove the Amazon CloudWatch Observability add-on from your EKS cluster:</li> 
</ul> 
<pre><code class="lang-bash">
aws eks delete-addon \
 --cluster-name \
 --addon-name amazon-cloudwatch-observability
</code></pre> 
<ul> 
 <li>Remove the Prometheus data source from your Grafana workspace (log in to the Grafana console, navigate to Data Sources, and delete the CloudWatch PromQL data source you configured).</li> 
</ul> 
<ul> 
 <li>Delete the Amazon Managed Grafana workspace (only if created for this walkthrough):</li> 
</ul> 
<pre><code class="lang-bash">
aws grafana delete-workspace --workspace-id 
</code></pre> 
<ul> 
 <li>Delete the Amazon EKS cluster (only if created for this walkthrough):</li> 
</ul> 
<pre><code class="lang-bash">
aws eks delete-cluster --name 
</code></pre> 
<ul> 
 <li>Disable OTel enrichment (if no longer needed for your account):</li> 
</ul> 
<pre><code class="lang-bash">
# Disable OTel enrichment
aws cloudwatch stop-otel-enrichment

# Disable telemetry enrichment
aws observabilityadmin stop-telemetry-enrichment
</code></pre> 
<ul> 
 <li>Detach the IAM policy if it was attached specifically for this walkthrough:</li> 
</ul> 
<pre><code class="lang-bash">
aws iam detach-role-policy \
 --role-name \
 --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
</code></pre> 
<h1>Conclusion</h1> 
<p>This post walked through native OpenTelemetry metrics ingestion in Amazon CloudWatch: enabling the enrichment layers, deploying Container Insights on Amazon EKS, sending custom application metrics with the OpenTelemetry SDK, querying everything with PromQL.</p> 
<p>With this preview capability, you can consolidate your metrics pipeline into CloudWatch. High-cardinality metrics with extended label limits, PromQL queries, and automatic AWS resource enrichment work together so that infrastructure metrics, container metrics, and application metrics all flow through the same pipeline and carry the same AWS resource context. No separate backends, no exporters, no additional API calls to bring your AWS metrics into a unified view.</p> 
<p>For more hands-on examples of application-level instrumentation with OpenTelemetry, explore the following resources:</p> 
<ul> 
 <li><a href="https://aws-observability.github.io/observability-best-practices/">AWS Observability Best Practices Guide:</a>&nbsp;patterns for instrumenting applications with OpenTelemetry SDKs</li> 
 <li><a href="https://catalog.workshops.aws/observability/en-US">One Observability Workshop</a>: hands-on labs for metrics, traces, and logs on AWS</li> 
 <li><a href="https://aws-observability.github.io/aws-observability-accelerator/">AWS Observability Accelerator:</a> CDK patterns and Terraform modules to automate telemetry collection and query</li> 
</ul> 
<p>The preview is available at no additional cost in US East (N. Virginia), US West (Oregon), Europe (Ireland), Asia Pacific (Singapore), and Asia Pacific (Sydney). To get started, enable the enrichment layers in your account and deploy the CloudWatch Observability add-on on an Amazon EKS cluster.</p>
