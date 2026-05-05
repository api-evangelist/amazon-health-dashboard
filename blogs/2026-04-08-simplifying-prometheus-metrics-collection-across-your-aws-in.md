---
title: "Simplifying Prometheus metrics collection across your AWS infrastructure"
url: "https://aws.amazon.com/blogs/mt/simplifying-prometheus-metrics-collection-across-your-aws-infrastructure/"
date: "Wed, 08 Apr 2026 14:39:52 +0000"
author: "Nilo Bustani"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<p>If you’re running services such as <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon EC2</a> instances, <a href="https://aws.amazon.com/ecs/" rel="noopener" target="_blank">Amazon Elastic Container Service</a> (Amazon ECS) containers, and <a href="https://aws.amazon.com/msk/" rel="noopener" target="_blank">Amazon Managed Streaming for Apache Kafka</a> (Amazon MSK) clusters in AWS, maintaining separate Prometheus servers for each environment creates significant operational burden. Managing scraper configurations, high availability, scaling, and security distracts you from building great applications.</p> 
<p><a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector.html" rel="noopener" target="_blank">AWS managed collector</a> or <a href="https://aws.amazon.com/prometheus/" rel="noopener" target="_blank">Amazon Managed Service for Prometheus</a> scraper helps eliminate this overhead. Instead of deploying and maintaining Prometheus servers in each environment, you can now use fully managed scrapers that collect Prometheus metrics from your <a href="https://docs.aws.amazon.com/vpc/" rel="noopener" target="_blank">Amazon VPC</a>-connected resources and store them in your Amazon Managed Service for Prometheus workspace. This means you no longer need to manage collector availability, scaling, or configuration drift.</p> 
<p>This post walks you through implementing AWS managed collectors across three common compute environments: Amazon EC2 instances, Amazon ECS workloads, and Amazon MSK clusters. The post demonstrates how a single managed service can replace multiple self-managed Prometheus deployments.</p> 
<h2>Prerequisites and setup</h2> 
<p>Before you begin, verify you have the following in place:</p> 
<p><strong>AWS Resources:</strong></p> 
<ul> 
 <li>An Amazon Managed Service for Prometheus <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-create-workspace.html" rel="noopener" target="_blank">workspace</a></li> 
 <li>An <a href="https://docs.aws.amazon.com/vpc/">Amazon VPC</a> with private subnets across multiple Availability Zones</li> 
 <li><a href="https://aws.amazon.com/iam/features/manage-permissions">IAM permissions</a> to create scrapers and manage security groups</li> 
 <li><a href="https://aws.amazon.com/grafana/" rel="noopener" target="_blank">Amazon Managed Grafana</a> <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html" rel="noopener" target="_blank">workspace</a> (optional, for visualization)</li> 
</ul> 
<p><strong>Tools:</strong></p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener" target="_blank">AWS CLI v2</a></li> 
</ul> 
<p><strong>Note:</strong> The examples use the US West (Oregon) us-west-2 Region. You can adapt the commands and ARNs to your preferred Region. The AWS CLI commands use shell variables (for example, $SUBNET_ID_1, $SECURITY_GROUP_ID) as placeholders. Before running any command, verify that all variables are populated with valid values for your environment. Testing commands in a non-production account first is recommended.</p> 
<h2>Collecting metrics from Amazon EC2 instances</h2> 
<p>EC2 instances remain a popular choice for workloads requiring specific instance types or configurations. You can start by configuring the managed scraper to collect both application metrics and system-level metrics from EC2 instances. Here’s what the setup can look like:</p> 
<p style="text-align: center;"><img alt="Figure 1: AWS managed-collector with workloads running on Amazon EC2 instances" class="aligncenter size-full" height="592" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig1-ec2-architecture.png" width="1400" /><br /> <em>Figure 1: AWS managed-collector with workloads running on Amazon EC2 instances</em></p> 
<h3>1. Set up node exporter</h3> 
<p>This example uses Prometheus Node Exporter on an Amazon EC2 instance to expose system metrics in Prometheus format. Follow the installation instructions in the <a href="https://prometheus.io/docs/guides/node-exporter/" rel="noopener" target="_blank">Prometheus Node Exporter documentation</a>.</p> 
<h3>2. Create the AWS managed collector configuration</h3> 
<p>The managed scraper will collect metrics from both the Node Exporter (exposing metrics on the default port 9100) and the sample application metrics (exposed on port 8080) using the following configuration:</p> 
<pre><code>cat &gt; /tmp/ec2-scraper.yml &lt;&lt; EOF
global:
  scrape_interval: 30s
  scrape_timeout: 10s

scrape_configs:
  - job_name: 'ec2-metrics'
    static_configs:
      - targets:
          - ip-$IP_ADDRESS.us-west-2.compute.internal:8080
    metrics_path: '/metrics'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'ec2-metrics-instance-new'
      - target_label: service
        replacement: 'ec2-metrics'
      - target_label: environment
        replacement: 'dev'
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: '.*'
        action: keep

  - job_name: ec2-node-exporter
    static_configs:
      - targets:
        - ip-$IP_ADDRESS.us-west-2.compute.internal:9100
EOF

CONFIG_BLOB=$(base64 -w 0 /tmp/ec2-scraper.yml)</code></pre> 
<p>The relabel_configs section placeholder can be modified to add consistent labels to your metrics, making it easier to query and filter across different services.</p> 
<h3>3. Deploy the managed scraper</h3> 
<pre><code>aws amp create-scraper \
  --alias "ec2-payment-service-scraper" \
  --source '{
    "vpcConfiguration": {
      "subnetIds": ["$SUBNET_ID_1", "$SUBNET_ID_2"],
      "securityGroupIds": ["$SECURITY_GROUP_ID"]
    }
  }' \
  --destination '{
    "ampConfiguration": {
      "workspaceArn": "arn:aws:aps:us-west-2:ACCOUNT_ID:workspace/ws-WORKSPACE_ID"
    }
  }' \
  --scrape-configuration "{\"configurationBlob\":\"$CONFIG_BLOB\"}"</code></pre> 
<p>When you create a managed scraper, Amazon Managed Service for Prometheus automatically creates a service-linked role (AWSServiceRoleForAmazonPrometheusScraperInternal) that grants the scraper permissions to access your VPC resources and write to your workspace.</p> 
<h3>4. Validate EC2 metrics</h3> 
<p>Within minutes, you’ll see metrics flowing into the destination Amazon Managed Service for Prometheus workspace. You can verify this in the <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-CW-usage-metrics.html" rel="noopener" target="_blank">CloudWatch metrics console</a> or by querying the workspace from Amazon Managed Grafana.</p> 
<p>Query metrics for the ec2-metrics job:</p> 
<pre><code>{job="ec2-metrics"}</code></pre> 
<p style="text-align: center;"><img alt="Figure 2: Querying Amazon Managed Service for Prometheus for EC2 metrics from the AWS-managed collector" class="aligncenter size-full" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig2-ec2-query-2.png" /><br /> <em>Figure 2: Querying Amazon Managed Service for Prometheus for EC2 metrics from the AWS-managed collector</em></p> 
<h2>Monitoring ECS workloads with dynamic service discovery</h2> 
<p>Amazon ECS tasks are ephemeral, IP addresses change as containers are replaced, and services scale dynamically. <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect-shared-namespaces.html" rel="noopener" target="_blank">AWS Cloud Map</a> service discovery addresses these challenges by maintaining DNS records for running tasks, which the managed scraper queries automatically.</p> 
<p style="text-align: center;"><img alt="Figure 3: AWS managed-collector with workloads running on Amazon ECS" class="aligncenter size-full" height="736" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig3-ecs-architecture.png" width="1400" /><br /> <em>Figure 3: AWS managed-collector with workloads running on Amazon ECS</em></p> 
<p>For this example, the payforadoption service from the <a href="https://catalog.workshops.aws/observability/en-US" rel="noopener" target="_blank">AWS One Observability Workshop</a> is used, which demonstrates a realistic microservice exposing Prometheus metrics.</p> 
<h3>1. Configure security groups</h3> 
<p>Confirm that the ECS task security group allows inbound traffic from the scraper:</p> 
<pre><code>aws ec2 authorize-security-group-ingress \
  --group-id $SECURITY_GROUP_ID \
  --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,UserIdGroupPairs='[{GroupId=$SECURITY_GROUP_ID}]'</code></pre> 
<p>This creates a self-referencing rule, allowing resources within the same security group to communicate.</p> 
<h3>2. Create the AWS managed collector configuration</h3> 
<p>Create the DNS based scraper configuration:</p> 
<pre><code>cat &gt; /tmp/ecs-scraper.yml &lt;&lt; EOF
global:
  scrape_interval: 30s
  scrape_timeout: 10s

scrape_configs:
  - job_name: 'ecs-payforadoption'
    dns_sd_configs:
      - names: ['payforadoption-go.Workshop-space']
        type: A
        port: 80
    metrics_path: '/metrics'
    relabel_configs:
      - target_label: service_name
        replacement: 'payforadoption-go'
      - target_label: cloudmap_namespace
        replacement: 'Workshop-space'
      - target_label: environment
        replacement: 'production'
      - target_label: compute_platform
        replacement: 'ecs-fargate'
EOF

CONFIG_BLOB=$(base64 -w 0 /tmp/ecs-scraper.yml)</code></pre> 
<p>The dns_sd_configs section tells the scraper to query the DNS name payforadoption-go.Workshop-space and scrape all returned IP addresses.</p> 
<h3>3. Create the AWS managed collector</h3> 
<p>The deployment pattern remains the same as before:</p> 
<pre><code>aws amp create-scraper \
  --alias "ecs-payforadoption-scraper" \
  --source '{
    "vpcConfiguration": {
      "subnetIds": ["$SUBNET_ID_1", "$SUBNET_ID_2"],
      "securityGroupIds": ["$SECURITY_GROUP_ID"]
    }
  }' \
  --destination '{
    "ampConfiguration": {
      "workspaceArn": "arn:aws:aps:us-west-2:ACCOUNT_ID:workspace/ws-WORKSPACE_ID"
    }
  }' \
  --scrape-configuration "{\"configurationBlob\":\"$CONFIG_BLOB\"}"</code></pre> 
<p>When Amazon ECS replaces a task, the scraper picks up the changes on its next DNS query (typically every 30 seconds), supporting continuous metrics collection without manual intervention.</p> 
<h3>4. Validate Amazon ECS metrics</h3> 
<p>Run a query. For example, show the average requests per second to the ECS task over the last 5 minutes:</p> 
<pre><code>rate(payforadoption_requests_total[5m])</code></pre> 
<p style="text-align: center;"><img alt="Figure 4: Querying Amazon Managed Service for Prometheus for ECS metrics" class="aligncenter size-full" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig4-ecs-query-2.png" /><br /> <em>Figure 4: Querying Amazon Managed Service for Prometheus for ECS metrics</em></p> 
<h2>Collecting Prometheus metrics from Amazon MSK clusters</h2> 
<p>Monitoring Amazon MSK cluster health is critical for event-driven architectures. Amazon MSK clusters expose two types of metrics through Prometheus exporters when you enable OpenMonitoring: JMX Exporter for Kafka-specific metrics (topics, partitions, consumer lag) and Node Exporter for broker system metrics (CPU, memory, disk).</p> 
<p style="text-align: center;"><img alt="Figure 5: AWS managed-collector with workloads running on Amazon MSK" class="aligncenter size-full" height="694" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig5-msk-architecture.png" width="1400" /><br /> <em>Figure 5: AWS managed-collector with workloads running on Amazon MSK</em></p> 
<h3>1. Enable OpenMonitoring on Amazon MSK clusters</h3> 
<p>The following command enables both exporters on all broker nodes. JMX Exporter listens on port 11001, and Node Exporter on port 11002:</p> 
<pre><code>aws kafka update-monitoring \
  --cluster-arn "arn:aws:kafka:REGION:ACCOUNT_ID:cluster/metrics-msk-cluster/CLUSTER_ID" \
  --current-version "CURRENT_VERSION" \
  --open-monitoring '{
    "Prometheus": {
      "JmxExporter": {"EnabledInBroker": true},
      "NodeExporter": {"EnabledInBroker": true}
    }
  }' \
  --enhanced-monitoring PER_TOPIC_PER_PARTITION</code></pre> 
<h3>2. Deploy the scraper for Amazon MSK</h3> 
<p>Amazon MSK provides a cluster-level DNS name that resolves to all broker IPs. Using this for service discovery makes your monitoring resilient to broker replacements and cluster scaling.</p> 
<p>Get your cluster DNS name (remove the broker-specific prefix like b-1. or b-2.):</p> 
<pre><code>CLUSTER_DNS="metricsmskcluster.xxxx.xxx.kafka.REGION.amazonaws.com"

cat &gt; /tmp/msk-scraper.yml &lt;&lt; EOF
global:
  scrape_interval: 30s
  external_labels:
    cluster_name: metrics-msk-cluster

scrape_configs:
  - job_name: msk-jmx
    scheme: http
    metrics_path: /metrics
    scrape_timeout: 10s
    dns_sd_configs:
      - names:
          - $CLUSTER_DNS
        type: A
        port: 11001
    relabel_configs:
      - source_labels: [__meta_dns_name]
        target_label: broker_dns
      - source_labels: [__address__]
        target_label: instance
      - target_label: compute_platform
        replacement: 'msk'

  - job_name: msk-node
    scheme: http
    metrics_path: /metrics
    scrape_timeout: 10s
    dns_sd_configs:
      - names:
          - CLUSTER_DNS
        type: A
        port: 11002
    relabel_configs:
      - source_labels: [__meta_dns_name]
        target_label: broker_dns
      - source_labels: [__address__]
        target_label: instance
      - target_label: compute_platform
        replacement: 'msk'
EOF

CONFIG_BLOB=$(base64 -w 0 /tmp/msk-scraper.yml)</code></pre> 
<h3>3. Create the AWS managed collector</h3> 
<p>Amazon MSK brokers need to allow inbound traffic from the scraper on both exporter ports. You’ll then deploy the scraper configuration:</p> 
<pre><code>SCRAPER_SG=$(aws cloudformation describe-stacks \
  --stack-name msk-metrics-stack \
  --query 'Stacks[0].Outputs[?OutputKey==`PrometheusScraperSecurityGroupId`].OutputValue' \
  --output text)

aws amp create-scraper \
  --alias "msk-metrics-scraper" \
  --source "{\"vpcConfiguration\":{\"subnetIds\":[\"subnet-xxx\",\"subnet-yyy\"],\"securityGroupIds\":[\"$SCRAPER_SG\"]}}" \
  --destination "{\"ampConfiguration\":{\"workspaceArn\":\"arn:aws:aps:us-west-2:ACCOUNT_ID:workspace/ws-WORKSPACE_ID\"}}" \
  --scrape-configuration "{\"configurationBlob\":\"$CONFIG_BLOB\"}"</code></pre> 
<h3>4. Verify Amazon MSK metrics</h3> 
<p>Within minutes, you start seeing consumer lag, partition counts, broker resource utilization, and other MSK metrics without managing any Prometheus infrastructure.</p> 
<p>Run a query. For example, show the overall broker throughput across all topics using:</p> 
<pre><code>kafka_server_BrokerTopicMetrics_MeanRate</code></pre> 
<p style="text-align: center;"><img alt="Figure 6: Querying Amazon Managed Service for Prometheus for Amazon MSK metrics" class="aligncenter size-full" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig6-msk-query-2.png" /><br /> <em>Figure 6: Querying Amazon Managed Service for Prometheus for Amazon MSK metrics</em></p> 
<h2>Bringing it all together</h2> 
<p>Now that you have metrics from EC2, ECS, and MSK, you can unify querying and alerting across your entire infrastructure. You can <a href="https://docs.aws.amazon.com/grafana/latest/userguide/prometheus-data-source.html" rel="noopener" target="_blank">connect an Amazon Managed Grafana workspace</a> to the Amazon Managed Service for Prometheus workspace as a data source and write PromQL queries that span all three compute platforms:</p> 
<p>Query total request rate across all services:</p> 
<pre><code>sum(rate({__name__=~"http_requests_total|payforadoption_requests_total"}[5m])) by (environment, service)</code></pre> 
<p>This aggregates HTTP request rates from the EC2 instance and ECS service, grouped by service and platform.</p> 
<p style="text-align: center;"><img alt="Figure 7: Querying Amazon Managed Service for Prometheus for total request rate across all services" class="aligncenter size-full" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig7-total-request-rate-1.png" /><br /> <em>Figure 7: Querying Amazon Managed Service for Prometheus for total request rate across all services</em></p> 
<p>Monitor CPU usage across compute types:</p> 
<p>This query works identically for EC2 instances and MSK brokers since both expose Node Exporter metrics with the same naming convention.</p> 
<pre><code>100 - (avg by (instance, job) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)</code></pre> 
<p style="text-align: center;"><img alt="Figure 8: Querying Amazon Managed Service for Prometheus for CPU usage across all compute types" class="aligncenter size-full" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/07/amp-scraper-fig8-cpu-usage-1.png" /><br /> <em>Figure 8: Querying Amazon Managed Service for Prometheus for CPU usage across all compute types</em></p> 
<p>Track Kafka consumer lag for ECS consumers:</p> 
<p>This helps you identify if your Amazon MSK consumers are keeping up with message production.</p> 
<pre><code>sum(kafka_consumer_group_ConsumerLagMetrics_Value) by (groupId, topic)</code></pre> 
<h3>Cross-service alerting</h3> 
<p>You can define alerts that span the infrastructure. For example, alert when consumer lag exceeds a threshold AND the consuming service’s error rate increases:</p> 
<pre><code>(
  sum(kafka_consumer_group_ConsumerLagMetrics_Value{topic="payment-processor"}) &gt; 10000
  and
  rate(http_requests_total{service_name="payment-api",status=~"5.."}[5m]) &gt; 0.01
)</code></pre> 
<p>Correlating Kafka metrics with application metrics can help you identify root causes faster and reduce mean time to resolution.</p> 
<h2>Security considerations</h2> 
<ul> 
 <li>Under the <a href="https://aws.amazon.com/compliance/shared-responsibility-model/" rel="noopener" target="_blank">AWS Shared Responsibility Model</a>, AWS manages the security of the managed scraper infrastructure while you are responsible for configuring secure access to your resources.</li> 
 <li>Implement least privilege IAM policies. Each scraper assumes an IAM role. Follow the principle of least privilege by granting only the permissions needed:</li> 
</ul> 
<pre><code>{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "aps:RemoteWrite"
      ],
      "Resource": "arn:aws:aps:us-west-2:ACCOUNT_ID:workspace/ws-WORKSPACE_ID"
    }
  ]
}</code></pre> 
<ul> 
 <li>Restrict security group ingress to only the scraper’s security group on the specific exporter ports</li> 
 <li>Deploy scrapers in private subnets and use VPC endpoints for Amazon Managed Service for Prometheus to keep traffic within your VPC</li> 
 <li>Data sent from the scraper to the Amazon Managed Service for Prometheus workspace is encrypted in transit using TLS</li> 
 <li>The scraper communicates with targets over your VPC’s private network. You can configure TLS on your exporters using scheme: https for additional protection</li> 
 <li>Metrics stored in Amazon Managed Service for Prometheus are encrypted at rest by default. You can optionally use <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/security-encryption-at-rest.html" rel="noopener" target="_blank">customer managed keys</a> for additional control</li> 
 <li>Enable scraper logging to Amazon CloudWatch Logs for auditing and troubleshooting</li> 
</ul> 
<h2>Best practices for production deployments</h2> 
<p>Based on real-world implementations, here are key best practices to ensure reliable, secure, and scalable metrics collection:</p> 
<ul> 
 <li>For EC2 workloads, consider migrating from static targets to DNS-based service discovery using the dns_sd_configs directive in your scraper configuration and <a href="https://docs.aws.amazon.com/cloud-map/latest/dg/working-with-instances.html" rel="noopener" target="_blank">registering your EC2 instances in AWS Cloud Map</a>.</li> 
 <li>Deploy multiple scrapers when you have targets with different lifecycles, access patterns or security exposures for better isolation, security and operational flexibility</li> 
 <li>Set appropriate scrape intervals to balance metrics granularity with cost and performance: 
  <ul> 
   <li>30 seconds: Good default for most application metrics</li> 
   <li>60 seconds: Sufficient for infrastructure metrics (CPU, memory)</li> 
   <li>90 seconds+: Non-production environments or low throughput applications</li> 
   <li>Remember that halving your scrape interval doubles your ingestion cost</li> 
  </ul> </li> 
 <li>Drop noisy metrics with relabel configurations</li> 
</ul> 
<pre><code>metric_relabel_configs:
  - source_labels: [__name__]
    regex: '.*_debug_.*'
    action: drop</code></pre> 
<h2>Cleanup</h2> 
<p>To avoid incurring ongoing charges, delete the resources created during this walkthrough when they are no longer needed.</p> 
<p>Remove the managed scrapers using the AWS CLI:</p> 
<pre><code>aws amp list-scrapers
aws amp delete-scraper --scraper-id $SCRAPER_ID</code></pre> 
<p>Additionally, delete the Amazon Managed Service for Prometheus workspace if it was created for testing purposes:</p> 
<pre><code>aws amp delete-workspace --workspace-id $WORKSPACE_ID</code></pre> 
<p>If you deployed supporting resources such as EC2 instances, ECS services, MSK clusters, security groups, or Amazon Managed Grafana workspaces specifically for this walkthrough, remove those as well to stop all associated costs.</p> 
<h2>Conclusion</h2> 
<p>This post walked through implementing Amazon Managed Service for Prometheus managed collector across three distinct compute environments — Amazon EC2, Amazon ECS, and Amazon MSK clusters — demonstrating how a single managed service can replace multiple self-managed Prometheus deployments. By using DNS-based service discovery, consistent labeling strategies, and unified querying in Grafana, you can build a comprehensive observability solution without the operational burden of managing scraper infrastructure.</p> 
<p>The key benefits of this approach:</p> 
<ul> 
 <li><strong>Reduced operational overhead:</strong> No Prometheus servers to patch, scale, or monitor</li> 
 <li><strong>Automatic resilience:</strong> DNS-based discovery adapts to infrastructure changes automatically</li> 
 <li><strong>Unified observability:</strong> Query metrics across all compute platforms from a single interface</li> 
 <li><strong>Cost optimization:</strong> Consolidated billing and easy metric filtering reduce costs</li> 
 <li><strong>Security:</strong> Managed service handles infrastructure security while you control access policies</li> 
</ul> 
<p>Whether you’re running a hybrid architecture across multiple compute types or planning to migrate between platforms, Amazon Managed Service for Prometheus managed collector provides the flexibility and reliability you need for production observability.</p> 
<h2>Next steps</h2> 
<p>Ready to implement this in your environment? Here are some resources to get started:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/prometheus/" rel="noopener" target="_blank">Amazon Managed Service for Prometheus documentation</a></li> 
 <li><a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector.html" rel="noopener" target="_blank">AWS managed collectors API reference</a></li> 
 <li><a href="https://catalog.workshops.aws/observability/" rel="noopener" target="_blank">AWS One Observability Workshop</a> – Hands-on labs</li> 
 <li><a href="https://prometheus.io/docs/practices/" rel="noopener" target="_blank">Prometheus configuration best practices</a></li> 
 <li><a href="https://docs.aws.amazon.com/grafana/" rel="noopener" target="_blank">Amazon Managed Grafana documentation</a></li> 
</ul> 
<p>Have questions or want to share your implementation? Leave a comment below or reach out to the AWS observability team through your account team.</p>
