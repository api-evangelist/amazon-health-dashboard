---
title: "Scaling AWS Governance: How Moeve reduced response times with automated notifications"
url: "https://aws.amazon.com/blogs/mt/how-moeve-reduced-response-times-with-automated-notifcations/"
date: "Thu, 12 Feb 2026 16:01:18 +0000"
author: "Ignacio Rodríguez García"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<p>Moeve, formerly known as Cepsa, is a global integrated energy company with over 90 years of experience and more than 11,000 employees. Moeve is committed to driving Europe’s energy transition and accelerating decarbonization efforts. The company has embraced digital transformation to enhance energy efficiency, safety, and sustainability, focusing on investments in green hydrogen, second-generation biofuels, and ultra-fast electric vehicle charging infrastructure.</p> 
<p>Since 2020, Moeve has put a lot of focus on Governance topics, becoming <a href="https://aws.amazon.com/controltower/">AWS Control Tower</a> heavy users and early adopters of its features. In a continuous effort to customize their environment, they recently started using <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> hooks to have more flexibility while trying to make their organization compliant with their rules.</p> 
<h2><strong>Introduction</strong></h2> 
<p><em>The importance of a notification system</em></p> 
<p>Operating in a multi-account AWS environment brings flexibility, but requires a mature governance strategy as you scale. Gaining real-time visibility into what’s happening across the organization is a constant challenge. Whether it’s a security event, a misconfiguration, a cost anomaly, or an infrastructure change, being able to notify the right team at the right time is critical for operational excellence and in time remediation.</p> 
<p><em>Key foundations for implementing a notification system</em></p> 
<p>To notify to the right team or individual, you first need clear ownership: who is responsible for each resource, system, or workload? That requires an up-to-date, automated inventory of your environment, one that includes resource metadata, owner attribution and contextual tagging.</p> 
<p>Something to consider is signal versus noise. Not every event should trigger a notification. Bombarding teams with irrelevant alerts leads to fatigue and blind spots. Therefore, a well-designed notification system must include filtering logic to determine what must be notified, how, and to whom.</p> 
<h2><strong>Prerequisites</strong></h2> 
<p>In this blog post, you will see how Moeve implemented a notification system by leveraging specific AWS services. The following components were in place for it:</p> 
<p><em>AWS Environment Setup</em></p> 
<ul> 
 <li><a href="https://aws.amazon.com/organizations/"><strong>AWS Organizations</strong></a> with multi-account structure set up</li> 
 <li><strong>AWS Control Tower</strong> deployed at the organization level with: 
  <ul> 
   <li>AWS Config aggregator configured</li> 
   <li>AWS CloudTrail enabled across the organization</li> 
  </ul> </li> 
 <li><strong>Administrative access</strong> to your AWS Management Console and programmatic access with appropriate permissions</li> 
</ul> 
<p><em>&nbsp;AWS Services</em></p> 
<ul> 
 <li><a href="https://aws.amazon.com/config/"><strong>AWS Config</strong></a> with multi-account aggregator enabled</li> 
 <li><a href="https://aws.amazon.com/eventbridge/"><strong>Amazon EventBridge</strong></a> with cross-account event bus sharing permissions</li> 
 <li><a href="https://aws.amazon.com/lambda/"><strong>AWS Lambda</strong></a> for processing events and notifications</li> 
 <li><a href="https://aws.amazon.com/s3/"><strong>Amazon S3</strong></a> buckets for storing inventory data</li> 
 <li><a href="https://aws.amazon.com/glue/"><strong>AWS Glue</strong></a> for cataloging inventory data</li> 
 <li><a href="https://aws.amazon.com/athena/"><strong>Amazon Athena</strong></a> for querying inventory data</li> 
 <li><a href="https://aws.amazon.com/dynamodb"><strong>Amazon DynamoDB</strong></a> for storing ownership and notification metadata</li> 
 <li><a href="https://aws.amazon.com/ses/"><strong>Amazon Simple Email Service</strong></a> (Amazon SES) configured and verified for sending notifications</li> 
 <li><a href="https://aws.amazon.com/security-hub/"><strong>AWS Security Hub</strong></a> enabled across your organization</li> 
 <li><a href="https://aws.amazon.com/premiumsupport/technology/trusted-advisor/"><strong>AWS Trusted Advisor</strong></a> with Business or Enterprise Support subscription</li> 
 <li><a href="https://aws.amazon.com/aws-cost-management/aws-cost-anomaly-detection/"><strong>AWS Cost Anomaly Detection</strong></a> configured for cost monitoring</li> 
</ul> 
<p><em>Tagging Strategy</em></p> 
<ul> 
 <li>Established tagging strategy with mandatory tags including: 
  <ul> 
   <li>Owner/Team tags</li> 
   <li>Project tags</li> 
   <li>Business Unit tags</li> 
   <li>Environment tags (e.g., dev, test, prod)</li> 
  </ul> </li> 
 <li>AWS Organizations tag policies implemented at the organization and account levels</li> 
 <li><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html">Service Control Policies</a> (SCPs) for enforcing tagging compliance</li> 
</ul> 
<p><em>Technical Knowledge</em></p> 
<ul> 
 <li>Familiarity with AWS Well-Architected Framework pillars</li> 
 <li>Experience with Infrastructure as Code (AWS CloudFormation, Terraform, or <a href="https://aws.amazon.com/cdk/">AWS Cloud Development Kit</a> (AWS CDK))</li> 
 <li>Understanding of <span style="color: #0972d3;"><a href="https://aws.amazon.com/iam/" style="color: #0972d3;">AWS Identity and Access Management</a></span> (IAM)</li> 
 <li>Basic knowledge of Python for Lambda functions</li> 
 <li>SQL knowledge for writing Athena queries</li> 
 <li>Understanding of EventBridge rules and patterns</li> 
</ul> 
<p><em>Resource Inventory System</em></p> 
<ul> 
 <li>Centralized resource inventory mechanism</li> 
 <li>Mapping between resource tags and ownership information</li> 
 <li>Process for keeping inventory data current and accurate</li> 
</ul> 
<p><em>Additional Requirements</em></p> 
<ul> 
 <li>Email distribution lists or notification channels for different teams</li> 
 <li>Defined notification priorities and schedules aligned with Well-Architected pillars</li> 
 <li>Established remediation procedures for common issues</li> 
</ul> 
<h2><strong>Tagging and Inventory as the foundations of a notification system</strong></h2> 
<p><strong>A. Tagging for governance</strong></p> 
<p>In order to create AWS notifications, you need to identity owners to the resource. Without clear ownership, alerts become spam. Moeve embed ownership through consistent tagging using technical enforcement, deployment discipline, and cultural alignment:</p> 
<ul> 
 <li>Tagging Culture: They established that tagging (Owner, Team, Application, Environment) is mandatory infrastructure hygiene, integrated into their development.</li> 
 <li>Infrastructure as Code: Moeve uses AWS CloudFormation, Terraform, and AWS CDK to enforce tagging at deployment, minimizing human error and ensuring consistency.</li> 
 <li>AWS Tag Policies: They implemented organization-wide tag policies requiring specific keys (Project, Business Unit) with defined values, enforced through layered policies at the Organization Unit level. “Lifecycle from day one.”</li> 
</ul> 
<pre><code class="lang-json">{
	  "tags": {
	    "global:bu": {
	      "tag_key": {
	        "@@assign": "bu"
	      },
	      "tag_value": {
	        "@@assign": [
	          "bu_value"
	        ]
	      },
	      "enforced_for": {
	        "@@assign": [
                  "aoss:ALL_SUPPORTED",
	          "amplifyuibuilder:ALL_SUPPORTED",
	          "apigateway:ALL_SUPPORTED",
	          "appmesh:ALL_SUPPORTED",
	          "appconfig:application",
	          "appconfig:application/configurationprofile",		  
			  All availabled resources
			  
			  ... 
			  "workspaces:*"
	        ]
	      }
	    }
	  }
}
</code></pre> 
<p>They also leveraged the <a href="https://aws.amazon.com/about-aws/whats-new/2025/07/aws-organization-tag-policies-wildcard-statement/">new wildcard support</a> in AWS Organizations Tag Policies to simplify and scale their tagging enforcement across all resource types within each service. Then at the account level, they add a tag policy that includes the permitted tags for the projects hosted in that account, as illustrated in the next example.</p> 
<pre><code class="lang-json">{
	  "tags": {
	    "global:bu": {
	      "tag_key": {
	        "@@assign": "bu"
	      },
	      "tag_value": {
	        "@@append": [
	          "project_bu_value"
	        ]
	      }
	    }
	  }
}</code></pre> 
<div class="wp-caption alignnone" id="attachment_64123" style="width: 641px;">
 <img alt="Organization Unit setup and tagging" class="size-full wp-image-64123" height="881" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/figure1.png" width="631" />
 <p class="wp-caption-text" id="caption-attachment-64123">Figure 1. Organization Unit setup and tagging</p>
</div> 
<ul> 
 <li>Preventing untagged resources with <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html">Service Control Policies</a> (SCPs)</li> 
 <li>In some cases, they needed stricter enforcement. They used SCPs to prevent the creation of resources if critical tags are missing or malformed. For instance, they blocked AWS CloudFormation Stacksets from being created that didn’t include a BU.</li> 
</ul> 
<pre><code class="lang-json">{
	      "Sid": "SCPBUTag",  
	      "Effect": "Deny",
	      "Action": [
	        "cloudformation:UpdateStackSet",
	        "cloudformation:CreateStack",
	        "cloudformation:UpdateStack"
	      ],
	      "Resource": [
	        "arn:aws:cloudformation:*:*:stack/*/*",
	        "arn:aws:cloudformation:*:*:stackset/*:*"
	      ],
	      "Condition": {
	        "ForAllValues:StringNotEquals": {
	          "aws:TagKeys": [
	            ":bu"
	         ]
	        }
	      }
}
</code></pre> 
<p>This creates an environment where every resource has traceable ownership, enabling targeted notifications routed directly to those who can act on them.</p> 
<p><strong>From Tagging to Targeted Notifications</strong></p> 
<p>Once they have achieved consistent resource tagging and built an up-to-date project inventory, they moved from generic alerts to precise, actionable notifications.</p> 
<p>Their inventory maps Resource Tags (like Project, Owner, Team, Environment) to metadata such as contact information or ticketing queues. With this mapping in place, the notification logic can automatically answer critical questions such as:</p> 
<ul> 
 <li>What team owns the <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud</a> (EC2) instance that just triggered an <a href="https://aws.amazon.com/cloudwatch/">Amazon CloudWatch</a> alarm?</li> 
 <li>Who should be notified when a public S3 bucket is created in a specific environment?</li> 
</ul> 
<p>This tag-to-owner resolution layer has become the heart of their routing mechanism. It allows their system to dynamically look up the right recipients, even in a fast-changing cloud environment. By doing this, they decouple infrastructure events from hardcoded recipient lists and create a notification system that is automated, scalable, and ready to maintain.</p> 
<p><strong>B. Inventory</strong></p> 
<p>A centralized inventory is key to deliver alerts to the right people. Moeve implemented <a href="https://aws.amazon.com/controltower/">AWS Control Tower</a> at the organization level that allowed them to have a centralized <a href="https://aws.amazon.com/config/"><strong>AWS Config</strong></a><strong> aggregator</strong>. This gives them near real-time visibility into all the resources provisioned in our environment, including their configurations and compliance status. This is the backbone of their inventory strategy.</p> 
<div class="wp-caption alignnone" id="attachment_64124" style="width: 881px;">
 <img alt="Notification system architecture" class="size-full wp-image-64124" height="711" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/figure2.png" width="871" />
 <p class="wp-caption-text" id="caption-attachment-64124">Figure 2. Notification system architecture</p>
</div> 
<p>For notifications to be useful, they combined inventory data with real-time event signals. They integrated different sources of truth into our notification pipeline, as described in Figure 2:</p> 
<ul> 
 <li><strong>AWS Config</strong>: Central aggregator tracks resource state and changes across accounts and regions, helping them identify misconfigurations or policy violations. Here’s a simplified example of how they pull inventory data programmatically from the aggregator and push it to S3:</li> 
</ul> 
<pre><code class="lang-python">	def get_resource_types(config, aggregator_name):
	    resource_types = set()
	    paginator = config.get_paginator('describe_aggregate_discovered_resource_counts')
	    for page in paginator.paginate(ConfigurationAggregatorName=aggregator_name):
	        for group in page.get('GroupedResourceCounts', []):
	            resource_types.add(group['GroupName'])
	    return list(resource_types)
	def get_resources_by_type(config, aggregator_name, resource_type, accounts_filter=None, regions_filter=None):
	    resources = []
	    paginator = config.get_paginator('list_aggregate_discovered_resources')
	    for page in paginator.paginate(ConfigurationAggregatorName=aggregator_name, ResourceType=resource_type):
	        for res in page.get('ResourceIdentifiers', []):
	            if accounts_filter and res['SourceAccountId'] not in accounts_filter:
	                continue
	            if regions_filter and res['Region'] not in regions_filter:
	                continue
	            resources.append(res)
	    return resources
</code></pre> 
<p><strong>Scalability Note</strong>: If you’re querying many resource types or large datasets, Lambda may not be enough due to timeout limits. In their case, for heavy operations like full inventory exports, they use <strong>containerized workloads in </strong><a href="https://aws.amazon.com/pm/eks"><strong>Amazon EKS</strong></a> to run asynchronously and at scale. Once the data lands in an S3 bucket, they use <a href="https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html"><strong>AWS Glue Crawlers</strong></a> to catalog it into a table and <strong>query it with </strong><a href="https://aws.amazon.com/athena/"><strong>Amazon Athena.</strong></a></p> 
<ul> 
 <li><strong>Amazon EventBridge</strong>: It defines a set of rules to capture key operational and security-related events in real time, things like EC2 instance changes, public S3 bucket creations, and IAM modifications. Each rule is designed to <strong>match specific events</strong> and forward them to a <strong>central Amazon EventBridge bus</strong>, where you can fan out processing and notify the right owners.</li> 
</ul> 
<p>Here’s a simplified example of the pattern:</p> 
<p><strong>EventBridge Rule (Account A)</strong></p> 
<pre><code class="lang-json">{
  "detail-type": ["AWS API Call via CloudTrail"],
  "source": ["aws.elasticloadbalancing"],
  "detail": {
    "eventSource": ["elasticloadbalancing.amazonaws.com"],
    "requestParameters": {
      "scheme": ["internet-facing"],
      "type": ["application"]
    },
    "eventName": ["CreateLoadBalancer"]
  }
}</code></pre> 
<p>This rule targets a central bus in another account using <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-bus.html"><strong>event bus</strong></a><strong> sharing</strong>:</p> 
<p><strong>Target: Event bus in notification-core account</strong></p> 
<pre><code class="lang-json">{
  "Arn": "arn:aws:events:us-east-1:123456789012:event-bus/central-notifications",
  "RoleArn": "arn:aws:iam::123456789012:role/EventBridgeForwarderRole"
}</code></pre> 
<p>In the receiving account, Lambda processed the events. These processors enrich the events with metadata (tags, owners, compliance status) before sending notifications.</p> 
<ul> 
 <li><strong>AWS Security Hub</strong>: This consolidates security findings from different AWS services. It’s important for surfacing critical vulnerabilities or misconfigurations to the teams that can respond.</li> 
</ul> 
<ul> 
 <li><strong>AWS Trusted Advisor</strong>: It provides high-value insights for cost optimization, fault tolerance, performance, and to generate scheduled recommendations and notify the relevant owners.</li> 
</ul> 
<p>A Lambda function would then retrieve all findings or recommendations and store them in S3. Ultimately, the key is to have all the information in S3 and make it queryable via Amazon Athena.</p> 
<ul> 
 <li><a href="https://aws.amazon.com/aws-cost-management/aws-cost-anomaly-detection/"><strong>AWS Cost Anomaly Detection</strong></a>: At creation time all accounts go through an onboarding phase where a cost anomaly monitor is set up. This is used by the notification tool to alert the account owners about any anomalies.</li> 
</ul> 
<p>Their project inventory enriches events with context to automatically route notifications to the right people, creating targeted alerts instead of organizational spam. With this foundation, the next challenge is deciding what’s worth notifying to avoid alert fatigue and keep teams focused.</p> 
<h2><strong>The notification system solution</strong></h2> 
<p><strong>Choosing What and When to Notify</strong></p> 
<p>At this stage, the data layer is in place, resource ownership resolved, inventory sources integrated, and events flowing in. Next step is to decide what is worth notifying about, and what isn’t.</p> 
<p>One of Moeve’s learnings on this topic is that without careful filtering and prioritization, there is a risk of overwhelming teams with spam, leading to alert fatigue and, ultimately, ignored messages. They defined a clear strategy for both what to notify and how to notify, based on the nature and criticality of each event.</p> 
<p>Here’s the approach:</p> 
<ol> 
 <li>a) Classify Events by Criticality</li> 
</ol> 
<p>They started by categorizing their event sources into critical, important, and informational, depending on their potential impact:</p> 
<ul> 
 <li>Critical: Events that require immediate action: security findings, high-risk configuration drifts, failures affecting production workloads or big cost anomalies. These are notified in real time at detection.</li> 
</ul> 
<ul> 
 <li>Important: Events that need attention but don’t require immediate reaction, AWS Trusted Advisor checks, medium-severity security issues, resource changes in non-prod environments or cost recommendations. These go into weekly summaries.</li> 
</ul> 
<ul> 
 <li>Informational: All Health events are notified in real time.</li> 
</ul> 
<ol> 
 <li>b) Contextualize Every Message</li> 
</ol> 
<p>They adopted as a rule not to send raw events. Every notification includes enriched metadata, like project name, owner, environment, severity, and suggested action, so recipients can immediately understand the relevance and respond (or ignore) accordingly.</p> 
<p>By doing this, they created a system where alerts are timely, relevant, and targeted. Teams don’t get flooded, and when something does show up in their inbox, they know it’s worth paying attention to.</p> 
<p>Next, you’ll dive into how Moeve structured the notification engine itself, how they route, enrich, and deliver the messages using AWS services and serverless components.</p> 
<h2><strong>Notification approaches</strong></h2> 
<p>One of the foundations for notifications is a reliable inventory of resources. In their implementation, they operate with two different notification approaches. Let’s walk through the first one.</p> 
<p><strong>1.- Scheduled Notifications Based on Inventory Queries</strong></p> 
<p>For predictable, non-urgent insights, they run scheduled queries against our resource inventory. These queries are executed via Amazon Athena, usually once per day, with a cadence aligned to those <a href="https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html">AWS Well-Architected Framework pillars</a> most relevant for Moeve. That is:</p> 
<ul> 
 <li>Monday → Operational Excellence</li> 
 <li>Tuesday → Security</li> 
 <li>Wednesday → Reliability</li> 
 <li>Thursday → Performance Efficiency</li> 
 <li>Friday → Cost Optimization</li> 
</ul> 
<p>This way, they cycled through key areas of cloud health every week, distributing notifications in a focused and manageable way.</p> 
<div class="wp-caption alignnone" id="attachment_64138" style="width: 1531px;">
 <img alt="Notification system architecture" class="size-full wp-image-64138" height="971" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/figure3.png" width="1521" />
 <p class="wp-caption-text" id="caption-attachment-64138">Figure 3. Notification system architecture</p>
</div> 
<p>Here’s how it works:</p> 
<p><strong>Athena Query</strong> (Number 1 in Figure 3): Every day, they launched a query against our Glue-cataloged inventory to find resources that match the criteria for that day’s pillar. For example, publicly exposed Snapshots for Security Day.</p> 
<pre><code class="lang-sql">SELECT 
region,
	account_id,
	accountname,
	checkname,
	category,
	timestamp,
	issuppressed,
	resourceid,
	status,
	metadata
FROM inventory
where checkname = 'RDS snapshot should be private'
and status != 'ok'
</code></pre> 
<p><strong>Resource Tag Lookup</strong>: For every resource returned, they invoke a Lambda in the owning account to get its current tags. This ensures the most up-to-date information (not what was in the last inventory snapshot).</p> 
<p><strong>Ownership Resolution</strong>: With the tag data, they consult with their internal project database, which maps tags like Project, Team, or Environment to real owners, email groups, or ticket queues.</p> 
<p><strong>Email Notification</strong>: Then they built a custom notification—often in email format—with all the relevant context: resource ID, issue found, owning team, and even remediation tips if available.</p> 
<p><strong>2.- Near Real Time Notifications Based on Events </strong></p> 
<p>The second approach is with Real Time Notifications. In this case, they focus on events that were identified as critical or high-priority, things that require immediate attention and should be notified as soon as they are detected. These include events like public S3 bucket creation, IAM policy changes, VPC changes or non-compliant resource deployments.</p> 
<div class="wp-caption alignnone" id="attachment_64139" style="width: 1361px;">
 <img alt="Notification System Architecture – real time notifications" class="size-full wp-image-64139" height="775" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/figure4-2.png" width="1351" />
 <p class="wp-caption-text" id="caption-attachment-64139">Figure 4. Notification System Architecture – real time notifications</p>
</div> 
<p>To detect those events, Amazon EventBridge rules were created in individual accounts – Number 1 in Figure 4 – . When one of these events occurs, the rule targets a central Amazon EventBridge bus in their core notifications account. This allowed them to centralize detection and decouple it from processing.</p> 
<p>Once the event reaches the central bus – Number 2 in Figure 4 -, it would trigger a processing pipeline that follows a similar enrichment process than our inventory-based approach:</p> 
<p><strong>Tag Resolution</strong> – Number 3 Figure 4 -: A Lambda would be invoked in the source account to get the current tags of the affected resource. This step is important for accuracy, as they always wanted to notify based on the resource’s latest state, not on cached metadata.</p> 
<p><strong>Ownership Lookup</strong> – Number 4 Figure 4 -: Using the tag values to query their internal project ownership database to determine who is responsible for that resource.</p> 
<p><strong>Notification Generation</strong> – Number 5 in Figure 4 -: With that context, they constructed and sent a targeted email notification, including relevant event details, resource metadata, and ownership information.</p> 
<p><strong>Bonus: Real-Time Auto Remediation for Critical Events</strong></p> 
<p>In some high-severity cases, detection and notification are not enough—you also want to automatically remediate the issue as soon as it’s detected.</p> 
<p>For a subset of critical events (such as publicly exposed S3 buckets, insecure IAM policies, or untagged resources in sensitive environments), they had an automatic remediation enabled.</p> 
<div class="wp-caption alignnone" id="attachment_64126" style="width: 1323px;">
 <img alt="Automatic remediations solution architecture" class="size-full wp-image-64126" height="651" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/figure5.png" width="1313" />
 <p class="wp-caption-text" id="caption-attachment-64126">Figure 5. Automatic remediations solution architecture</p>
</div> 
<p><strong>Event Detection</strong>: A critical event is captured by Amazon EventBridge rule and forwarded to the central bus.</p> 
<p><strong>Remediation Lambda</strong>: Before sending any notification, a Lambda is invoked to remediate the resource and, if needed, takes immediate action. This could mean, for example, blocking or removing public access from a snapshot.</p> 
<p><strong>Notification with Remediation Info:</strong> If a remediation is applied, include that detail in the notification.</p> 
<p>This approach gives a layer of proactive protection, ensuring that certain violations are detected and immediately corrected, while keeping the owners informed.</p> 
<p>This helps reduce response time, enforce policy automatically, and maintain a high standard of security and compliance without human bottlenecks.</p> 
<p>The following animation shows an end-to-end demo for a notification and remediation following a deployment of an <a href="https://aws.amazon.com/kms/">AWS Key Management Service</a> (KMS) key without automatic rotation.</p> 
<div class="wp-caption alignnone" id="attachment_64122" style="width: 810px;">
 <img alt="Real-time notification and auto-remediation workflow" class="size-full wp-image-64122" height="396" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/animation1.gif" width="800" />
 <p class="wp-caption-text" id="caption-attachment-64122">Animation 1. Real-time notification and auto-remediation workflow</p>
</div> 
<h2><strong>Business impact </strong></h2> 
<p>Having a notification system in place is a step forward—but its real value lies in how it influences behavior across the organization for performance of the system. Moeve created a set of <a href="https://aws.amazon.com/quick/quicksight/"><strong>Amazon Quick Sight</strong></a> dashboards, which gave them visibility into key metrics and trends.</p> 
<p><strong>Key Performance Indicators (KPIs)</strong></p> 
<p>To validate these benefits and keep improving, they tracked a set of KPIs:</p> 
<ul> 
 <li>Notification volume by day, source, and severity</li> 
 <li>Auto-remediation success rate</li> 
 <li>Coverage: how many events are traceable to an owner</li> 
 <li>Recurrence of issues (e.g., how often the same team gets notified for the same problem)</li> 
</ul> 
<p><strong>Benefits</strong></p> 
<p>After running the notification system at scale, here are some of the benefits that were achieved over the past year:</p> 
<ul> 
 <li>45% reduction in staff needs thanks to automations.</li> 
 <li>Over 13,000 notifications sent across all teams and accounts in the last 12 months.</li> 
 <li>50% of all cost and security notifications include automated remediation, ensuring that issues are addressed before they escalate.</li> 
 <li>Over 85% adoption rate across all notifications, showing high engagement and accountability from teams.</li> 
 <li>5% savings on committed cloud spend, directly attributed to insights surfaced through cost-related notifications and follow-up actions.</li> 
</ul> 
<p>These numbers validate that notifications are tools to drive action, prevent incidents, and optimize their cloud footprint.</p> 
<div class="wp-caption alignnone" id="attachment_64127" style="width: 1607px;">
 <img alt="Dashboards of notifications" class="size-full wp-image-64127" height="885" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/02/05/figure6.png" width="1597" />
 <p class="wp-caption-text" id="caption-attachment-64127">Figure 6. Dashboards of notifications</p>
</div> 
<h2><strong>Conclusion</strong></h2> 
<p>A centralized, automated notification system in AWS provides scalable governance by delivering context-aware alerts through consistent tagging, directly engaging resource owners to drive accountability and faster response. Moeve has cut response times, reduced risk, and removed manual effort by embedding remediation into the event pipeline. By combining inventory data with real-time signals, enriched with ownership metadata and tracked for outcomes, such a system evolves from a simple tool into a cornerstone of secure, efficient, and scalable cloud operations.</p> 
<p>You can now take your cloud governance to the next level and implement your notification system.</p> 
<ul> 
 <li><strong>AWS Control Tower Setup</strong> 
  <ul> 
   <li>Start here: <a href="https://docs.aws.amazon.com/controltower/latest/userguide/getting-started-with-control-tower.html">AWS Control Tower Getting Started Guide</a></li> 
   <li>Multi-account setup: <a href="https://community.aws/content/2xSGZ4iNiTWPCc6p2hzEpSd9g12/setting-up-aws-control-tower-with-security-and-compliance-in-mind">Setting up AWS Control Tower with Security and Compliance</a></li> 
  </ul> </li> 
 <li><strong>Ready-to-Use GitHub Repositories (Sample Code &amp; Templates):</strong> 
  <ul> 
   <li><a href="https://github.com/aws-samples/aws-management-and-governance-samples">AWS Management &amp; Governance Samples</a></li> 
   <li><a href="https://github.com/aws-cloudformation/aws-cloudformation-templates">CloudFormation Templates</a></li> 
  </ul> </li> 
 <li><strong>Watch the Complete Demo</strong> 
  <ul> 
   <li>See Moeve’s notification system in action at <a href="https://www.youtube.com/watch?v=kQsPZSJ8f88">AWS re:Inforce 2025 Session: Empowering Critical Infrastructure Through Cloud Governance (GRC303)</a></li> 
  </ul> </li> 
</ul>
