---
title: "Simplify AWS Control Tower governance with enhanced AWS CloudFormation Hooks"
url: "https://aws.amazon.com/blogs/mt/simplify-aws-control-tower-governance-with-enhanced-aws-cloudformation-hooks/"
date: "Fri, 06 Feb 2026 20:47:54 +0000"
author: "Surya Vijayalakshmi"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<div class="WordSection1"> 
 <h2><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Introduction</span></h2> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Organizations using </span><span lang="EN-GB"><a href="https://aws.amazon.com/controltower/"><span style="font-family: 'Amazon Ember',sans-serif;">AWS Control Tower</span></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> to govern their multi-account environments face a persistent challenge: when AWS CloudFormation deployments fail due to proactive control violations, teams receive minimal information about why the failure occurred or how to fix it. This lack of visibility leads to:</span></p> 
 <ul type="disc"> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Delayed deployments as developers struggle to understand cryptic error messages</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Increased troubleshooting time spent investigating compliance failures</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Reduced confidence in proactive control enforcement</span></li> 
 </ul> 
 <p class="MsoNormal" style="margin-bottom: 0cm;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">AWS Control Tower customers can now gain unprecedented visibility into proactive control enforcement with the new <b>AWS</b> <b>CloudFormation Hook Invocation Summary </b>console page. This enhancement provides detailed execution logs and troubleshooting guidance that help organizations reduce deployment failures while accelerating compliance issue resolution.</span></p> 
 <h2><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">What are AWS Control Tower proactive controls?</span></h2> 
 <p class="MsoNormal"><span lang="EN-GB"><a href="https://docs.aws.amazon.com/controltower/latest/controlreference/proactive-controls.html"><span style="font-family: 'Amazon Ember',sans-serif;">Proactive controls</span></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> are <b>optional controls</b> implemented with </span><span lang="EN-GB"><a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/what-is-cloudformation-hooks.html"><span style="font-family: 'Amazon Ember',sans-serif;">AWS CloudFormation hooks</span></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> and managed by AWS Control Tower. Unlike detective controls that identify issues after deployment, proactive controls work as preventive guardrails that:</span></p> 
 <ul type="disc"> 
  <li class="MsoNormal" style="line-height: normal;"><b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Validate resource configurations before creation</span></b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> – Evaluate CloudFormation templates during the deployment process</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Enforce organizational policies at deployment time</span></b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> – Block non-compliant resources from being provisioned</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Provide immediate feedback</span></b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> – Give developers and administrators instant visibility into policy violations</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Prevent compliance drift</span></b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> – Stop security and governance issues before they occur</span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">&nbsp;</span></li> 
 </ul> 
 <p class="MsoNormal" style="margin-bottom: 0cm;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">When a CloudFormation deployment violates a proactive control, the hook prevents the resource from being created and provides feedback about the violation. This shift-left approach to governance helps organizations maintain compliance while reducing the cost of remediation.</span></p> 
 <h2><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">The AWS CloudFormation Hook invocation summary</span></h2> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">The CloudFormation Hook Invocation Summary console page transforms how teams interact with proactive controls by providing comprehensive visibility into hook execution. </span></p> 
 <h3><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Accessing the console</span></h3> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">To view hook execution details:</span></p> 
 <ol start="1" type="1"> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Open the </span><span lang="EN-GB"><a href="https://console.aws.amazon.com/cloudformation"><span style="font-family: 'Amazon Ember',sans-serif;">AWS CloudFormation console</span></a></span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Choose <b>Invocation Summary</b> from the left navigation panel</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Review detailed information about hook executions, including status, duration, and error messages</span></li> 
 </ol> 
 <h2><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Real-world scenario: CT.S3.PR.1 – Amazon S3 bucket public access control</span></h2> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">&nbsp;This walkthrough demonstrates how proactive controls work and how the AWS CloudFormation Hook Invocation Summary console helps identify and resolve violations for </span><span lang="EN-GB"><a href="https://docs.aws.amazon.com/controltower/latest/controlreference/s3-rules.html#ct-s3-pr-1-description"><span style="font-family: 'Amazon Ember',sans-serif;">CT.S3.PR.1 – S3 Bucket Public Access Control</span></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">. This control checks whether your Amazon Simple Storage Service (Amazon S3) bucket has a bucket-level Block Public Access (BPA) configuration.</span></p> 
 <h3><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Prerequisites</span></h3> 
 <ul type="disc"> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">An existing <a href="https://docs.aws.amazon.com/controltower/latest/userguide/quick-start.html">AWS Control Tower</a> environment set up</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Access to an AWS account within your AWS Control Tower organization</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Permissions to deploy AWS CloudFormation templates</span></li> 
 </ul> 
 <h3><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Enabling the proactive control</span></h3> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">First, let’s enable the CT.S3.PR.1 – S3 public access control:</span></p> 
 <ol start="1" type="1"> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Navigate to the <b>AWS Control Tower </b></span><span lang="EN-GB"><a href="https://us-east-1.console.aws.amazon.com/controltower/home?region=us-east-1"><span style="font-family: 'Amazon Ember',sans-serif;">console</span></a></span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Click on </span><span lang="EN-GB"><a href="https://us-east-1.console.aws.amazon.com/controltower/home/controls?region=us-east-1"><b><span style="font-family: 'Amazon Ember',sans-serif;">Control <span class="SpellE">Catalog</span></span></b></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> under the Controls tab</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Search for control: <b>CT.S3.PR.1</b> – “Require an Amazon S3 bucket to have block public access settings configured”</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Choose the <b>Control actions</b> and select <b>Enable</b></span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Choose the organizational units where you want to apply this control</span></li> 
 </ol> 
 <div class="wp-caption alignnone" style="width: 914px;">
  <img alt="Figure 1- Control Catalog page with filtered result for CT.S3.PR.1 control" border="0" height="236" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/01/08/image001.png" style="margin: 10px 0px 10px 0px;" width="904" />
  <p class="wp-caption-text">Figure 1- Control Catalog page with filtered result for CT.S3.PR.1 control</p>
 </div> 
 <div class="wp-caption alignnone" style="width: 914px;">
  <img alt="Screenshot of Control Catalog page with filtered result for CT.S3.PR.1 control" border="0" height="94" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/01/08/image002.png" style="margin: 10px 0px 10px 0px;" width="904" />
  <p class="wp-caption-text">Figure 1.2- Control Catalog page with filtered result for CT.S3.PR.1 control</p>
 </div> 
 <p class="MsoNormal" style="margin-bottom: 0cm;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Once enabled, this control will evaluate all S3 buckets created via AWS CloudFormation in the target account(s) of the OU where the control is enabled to ensure they have proper public access block settings configured.</span></p> 
 <h3><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Testing the “Fail” scenario</span></h3> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Now let’s create a stack that violates the control. Deploy this CloudFormation template:</span></p> 
 <div class="WordSection1"> 
  <pre><code class="lang-yaml">Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false</code></pre> 
  <p><b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Outcome:</span></b></p> 
 </div> 
 <p>The stack creation will fail with the error: <b>“The following hook(s) failed: [<span class="GramE">AWS::<span class="SpellE">ControlTower</span>::</span>Hook]”</b></p> 
 <div class="wp-caption alignnone" style="width: 914px;">
  <img alt="Screenshot of AWS CloudFormation stack events showing the failed hook execution" border="0" height="420" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/01/08/image003-1.png" style="margin: 10px 0px 10px 0px;" width="904" />
  <p class="wp-caption-text">Figure 2- AWS CloudFormation stack events showing the failed hook execution</p>
 </div> 
 <h3><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Viewing the Hook execution details</span></h3> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">To understand why the deployment failed:</span></p> 
 <ol start="1" type="1"> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Open the AWS CloudFormation </span><span lang="EN-GB"><a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1"><span style="font-family: 'Amazon Ember',sans-serif;">console</span></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> and navigate to your failed stack</span></li> 
  <li class="MsoNormal" style="line-height: normal;"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Choose the failed invocation from </span><span lang="EN-GB"><a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/hooks/invocations"><b><span style="font-family: 'Amazon Ember',sans-serif;">Invocation Summary</span></b></a></span><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"> on the left to see detailed root cause</span></li> 
 </ol> 
 <div class="wp-caption alignnone" style="width: 1653px;">
  <img alt="Screenshot of invocation ID page showing failed invocation details" border="0" height="477" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/01/08/image004.png" style="margin: 10px 0px 10px 0px;" width="1643" />
  <p class="wp-caption-text">Figure 3- Invocation ID page showing failed invocation details<b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">&nbsp;</span></b></p>
 </div> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">This detailed error message indicates: </span></p> 
 <ul> 
  <li class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Which control failed – <code>CT.S3.PR.1</code></span></li> 
  <li class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">What is the requirement – <code>block public access settings</code></span></li> 
  <li class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">How to fix it – <code>set all four parameters to true</code></span></li> 
 </ul> 
 <h3><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Testing the “Pass” scenario</span></h3> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Now, let’s fix the template and deploy a compliant version:</span></p> 
 <pre><code class="lang-yaml">Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true</code></pre> 
 <p class="MsoNormal"><b><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Outcome:</span></b></p> 
 <p>The stack creation succeeds! In the CloudFormation events, you’ll see: <b>“Hook invocations complete. Resource creation initiated”</b></p> 
 <div class="wp-caption alignnone" style="width: 914px;">
  <img alt="Screenshot of AWS CloudFormation stack events showing the hook execution status" border="0" height="236" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/01/08/image005.png" style="margin: 10px 0px 10px 0px;" width="904" />
  <p class="wp-caption-text">Figure 4- AWS CloudFormation stack events showing the hook execution status<span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">&nbsp;</span></p>
 </div> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">When you check the Invocation Summary, you’ll see: – Hook status: <b>PASS</b><br /> </span></p> 
 <div class="wp-caption alignnone" style="width: 914px;">
  <img alt="Screenshot of invocation summary page showing invocation result as “Pass”" border="0" height="82" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/01/08/image006.png" style="margin: 10px 0px 10px 0px;" width="904" />
  <p class="wp-caption-text">Figure 5-Invocation summary page showing invocation result as “Pass”</p>
 </div> 
 <p><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;"><b>Impact:</b> This proactive control prevents accidental exposure of Amazon S3 buckets to the public internet, protecting sensitive data from unauthorized access before the resource is even created.</span></p> 
 <h2><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">Conclusion</span></h2> 
 <p class="MsoNormal"><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">In this post, we walked through a practical example of enabling and testing proactive controls and explored the AWS CloudFormation Invocation Summary page. This improved visibility into hook executions marks an important step forward in cloud governance, making it easier to uphold security and compliance standards while empowering developers to stay productive.</span></p> 
 <p class="MsoNormal"><i><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">For more information about AWS Control Tower and proactive controls, visit the </span></i><span lang="EN-GB"><a href="https://docs.aws.amazon.com/controltower/"><i><span style="font-family: 'Amazon Ember',sans-serif;">AWS Control Tower documentation</span></i></a></span><i><span lang="EN-GB" style="font-family: 'Amazon Ember',sans-serif;">.</span></i></p> 
</div>
