---
title: "Automate AWS Systems Manager activation for hybrid-managed node registration"
url: "https://aws.amazon.com/blogs/mt/automate-aws-systems-manager-activation-for-hybrid-managed-node-registration/"
date: "Mon, 09 Mar 2026 16:20:31 +0000"
author: "Justin Thomas"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<p>AWS Systems Manager (formerly known as <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html#service-naming-history">SSM</a>) is an AWS service that you can use to view and control your servers on AWS cloud and on-premises infrastructure. Systems Manager makes it easy to manage a hybrid environment.</p> 
<p>To set up servers and virtual machines (VMs) in your hybrid environment as Systems Manager managed instances, you <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-managed-instance-activation.html">create a managed-instance activation</a>. Creating and managing Systems Manager Hybrid Activations credentials for your on-premises servers and VMs can be a manual and tedious task. The Hybrid Activations credentials can reach its activation expiration date or registration limit value after which the credentials can no longer be used to register the servers. The credentials will need to be recreated manually and the new servers have to be configured to use this new credentials. The core ask is to automate the creation and management of the System Manager Hybrid Activations credentials, reducing the operational support needed in this task.</p> 
<p>In this post, I will walk through the solution on automating the System Manager Hybrid Activations creation.</p> 
<h2>Solution overview</h2> 
<p>The solution is enabled using <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> stack. The Cloudformation stack creates the AWS resources on your account needed for the solution. These resources are as follows:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/api-gateway/">Amazon API Gateway</a>: REST API of Private Type, Integrated with AWS Lambda function. When the web client from the on-premises server performs a GET request to the API gateway, it returns the Hybrid Actions Code/ID combination.</li> 
 <li><a href="https://aws.amazon.com/lambda/">AWS Lambda</a>: The Lambda function provides the Hybrid Activations Code/ID combination to the on-premises server via the API gateway. It will create a new activation code if it finds the existing activation code is expired or has reached registration limit.</li> 
 <li><a href="https://aws.amazon.com/dynamodb/">Amazon DynamoDB</a>: To store the state, the Lambda updates the table to the ‘Locked’ state if it’s serving a request from a client. It updates the table to ‘Unlocked’ after completing serving the request.</li> 
 <li><a href="https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html">Amazon VPC Endpoint</a>: VPC Endpoint for API gateway for privately accessing the API gateway URL from the on-premises network.</li> 
 <li><a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html">AWS Systems Manager Parameter Store</a>: To store the Hybrid Activations ID/Code.</li> 
</ul> 
<div class="wp-caption aligncenter" id="attachment_29794" style="width: 710px;">
 <img alt="Solution Overview displays list of services used to automate activation for hybrid-managed node registration." class="wp-image-29794" height="254" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/07/01/cloudops_1942_1.jpg" width="700" />
 <p class="wp-caption-text" id="caption-attachment-29794">Solution Overview</p>
</div> 
<p>The following is a brief flow of the executions:</p> 
<ol> 
 <li>&nbsp;The web client calls the private API Gateway endpoint (for example, <a href="https://abc123xyz0.execute-api.eu-west-1.amazonaws.com/demostage">GET</a>).</li> 
 <li>&nbsp;When connecting from on-premises servers, the on-premises DNS server should be configured to forward requests to VPC DNS to get the private IP address of the VPC Endpoint. The DNS server resolves and sends back the IP address to the web client.</li> 
 <li>The request is sent to the private IP address of the VPC Endpoint of the API Gateway.</li> 
 <li>The resource Policy of the API gateway is checked to see if the request is coming from the VPC endpoint of the API gateway. If not, then it’s forbidden.</li> 
 <li>API Gateway passes the request to Lambda through an integration request.</li> 
 <li>Lambda updates the state key in DynamoDB to ‘Locked’, indicating it’s serving the request.</li> 
 <li>Lambda retrieves the credentials from the Parameter Store and sends it back to the client.</li> 
</ol> 
<h2>Walkthrough</h2> 
<h3>Prerequisites</h3> 
<p>For this walkthrough, you should have the following:</p> 
<ul> 
 <li>An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup">AWS account</a></li> 
 <li>An <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management (IAM)</a> user/role who can: 
  <ul> 
   <li>Create a private API, create a method, and deploy it in API Gateway.</li> 
   <li>Create Lambda function, DynamoDB, Parameter Store Parameter, and <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html">AWS CloudWatch Log</a> Group.</li> 
   <li>Create a new IAM role with a trust policy. Read more about <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege">Granting least privilege</a> when creating IAM policies.</li> 
  </ul> </li> 
 <li>The VPC to which you’re deploying must have both <a href="https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html#vpc-dns-updating">enableDnsSupport and enableDnsHostnames VPC</a> attributes set to <strong>true</strong>.</li> 
 <li>Basic familiarity with AWS CloudFormation, Systems Manager, and Amazon API Gateway.</li> 
</ul> 
<h3>Step1: Create VPC endpoint for API Gateway</h3> 
<p>In the first step, you create VPC endpoints for the API Gateway in your VPC. You also create a security group attached to the endpoint to allow a TCP port 443. Use the following steps to automate this using CloudFormation.</p> 
<p><strong>Note that if a VPC endpoint for API gateway already exists for the VPC, skip this step and note the existing VPC endpoint ID.</strong></p> 
<ol> 
 <li>Download the <a href="https://github.com/aws-samples/aws-systemsmanager-automate-hybrid-activations/blob/main/Cloudformation%20Templates/createVpcEndpoint.yml">CloudFormation Template</a>.</li> 
 <li>Visit the <a href="https://console.aws.amazon.com/cloudformation/home">AWS CloudFormation console</a> in your preferred region.</li> 
 <li>Choose Create stack, and then choose <strong>With new resources (standard)</strong>.</li> 
 <li>On the <strong>Create stack</strong> page, select <strong>Upload a template</strong>. Choose the template that you downloaded in the preceding step. Then, select Next.</li> 
 <li>Provide a <strong>Stack name</strong>. For example, apigateway-vpcendpoint-setup.</li> 
 <li>The CloudFormation stack requires a few parameters, as shown in the following screenshot:</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_29734" style="width: 710px;">
 <img alt="The Cloudformation stack parameters section displays Allowed IP range for VPC endpoint, Subnet IDs for VPC endpoint and VPC ID for Api Gateway." class="wp-image-29734" height="300" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/06/30/devops-1942_2_X.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-29734">Example for the Cloudformation Stack Parameters.</p>
</div> 
<ol start="7"> 
 <li>Choose <strong>Next</strong> on the <strong>Configure stack options</strong> page.</li> 
 <li>Review the configuration options and choose Create stack.</li> 
 <li>Verify that the stack has a status of <strong>CREATE_COMPLETE</strong>.</li> 
 <li>Once the stack has been created, refer the Outputs section of your stack and copy the VPC endpoint ID.</li> 
</ol> 
<h3>Step2: Create a KMS Key</h3> 
<p>In this step, you’ll create an <a href="https://aws.amazon.com/kms/">AWS Key Management Service (AWS KMS)</a> key to encrypt Parameter Store. Here, Parameter store is used to store the Activation Code and Activation ID. To create a KMS key:</p> 
<ol> 
 <li>Open the AWS KMS console <a href="https://console.aws.amazon.com/kms">here</a>.</li> 
 <li>In the navigation pane, choose <strong>Customer managed keys</strong> and select Create Key.</li> 
 <li>Choose the symmetric AWS KMS key, and select Next</li> 
 <li>Review the other configuration options, and create the Key.</li> 
 <li>Once created, note the key ID.</li> 
</ol> 
<h3>Step3: Create API Gateway and Lambda</h3> 
<p>In the final step, you’ll create and deploy a Private API and Lambda function. Use the following steps to automate this using CloudFormation.</p> 
<ol> 
 <li>Download the <a href="https://github.com/aws-samples/aws-systemsmanager-automate-hybrid-activations/blob/main/Cloudformation%20Templates/createApiGatewayLambda.yml">CloudFormation Template</a>.</li> 
 <li>Visit the <a href="https://console.aws.amazon.com/cloudformation/home">AWS CloudFormation</a> console in your preferred region.</li> 
 <li>Choose <strong>Create stack</strong>, and then choose<strong> With new resources (standard)</strong>.</li> 
 <li>On the<strong> Create stack</strong> page, select <strong>Upload a template</strong>. Choose the template that you downloaded in the preceding step. Then, select Next.</li> 
 <li>Provide a <strong>Stack name</strong>. For example, apigateway-lambda-setup.</li> 
 <li>The CloudFormation stack requires a few parameters, as shown in the following screenshot:</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_29735" style="width: 710px;">
 <img alt="The Cloudformation stack parameters displays Api Gateway Stage Name, Cloudwatch Log group Role ARN, Key Management service ID and VPC endpoint ID for Api Gateway." class="wp-image-29735" height="341" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/06/30/devops-1942_3_X-2.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-29735">Example for the Cloudformation Stack Parameters.</p>
</div> 
<ol start="7"> 
 <li>Review the details of your parameters, and check the box “I acknowledge that AWS CloudFormation might create IAM resources”. Then select Create stack to start building the resources.</li> 
 <li>Once the stack has been created, refer to the Outputs section of your stack and copy the API Gateway Invoke URL.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_29736" style="width: 710px;">
 <img alt="The cloud formation output displays the API Gateway Invoke URL." class="wp-image-29736" height="170" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/06/30/devops-1942_4-1.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-29736">Cloudformation Stack Output</p>
</div> 
<ol start="9"> 
 <li>From the on-premises server, which needs to be registered, access the copied URL using curl/wget or any other web client. The Activation ID/Code combination is returned in the JSON format. In the following example, on my Linux terminal, I am using curl and an optional jq package command to give a structured and formatted view of the output.</li> 
</ol> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-json">[root@ec2amaz-r1rvyg ~]# curl -s https://o2h4ocy7q6.execute-api.us-east-1.amazonaws.com/lambdastage | jq '.'
{
  "ActivationId": "e50a8437-23dd-4326-9e79-5e3b7573493e",
  "ActivationCode": "vVcH9zJX4ROy2XTsh5cb"
}
</code></pre> 
</div> 
<p><strong>Note that you should replace the URL in the example with the URL from your CloudFormation Stack Output.</strong></p> 
<p>You can improve the security of the <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html">private API</a> created above by configuring the VPC endpoint to use VPC endpoint policy. A VPC endpoint policy is an IAM resource policy that you can attach to an interface VPC endpoint to control access to the endpoint. VPC endpoint policies can be used together with API Gateway resource policies. The resource policy is used to specify which principals can access the API. The endpoint policy specifies which private APIs can be called via the VPC endpoint.</p> 
<p>Follow the documentation reference to<a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-vpc-endpoint-policies.html"> Create VPC endpoint policies for private APIs in API Gateway</a></p> 
<h2>Example scripts for automatic activation</h2> 
<p>You can use the API Gateway Invoke URL that you copied from the output section of the CloudFormation stack in your Shell/PowerShell script when installing SSM Agent. For testing and validation, you can save and run the following example scripts on a Redhat Based server or a Windows Server. For deployment at scale, have the script run on your server launch.</p> 
<p><strong>Linux:</strong></p> 
<p style="padding-left: 40px;">– A Shell script to retrieve Hybrid Activation credentials and install SSM Agent with the obtained credentials and register to the us-east-1 region:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-bash">#!/bin/bash
sudo yum erase amazon-ssm-agent --assumeyes

credentials=$(curl -s https://cla9phiczg.execute-api.us-east-1.amazonaws.com/lambdastage)
activationcode=$(echo $credentials | jq -r '.ActivationCode')
activationid=$(echo $credentials| jq -r '.ActivationId') 

mkdir /tmp/ssm
curl https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm -o /tmp/ssm/amazon-ssm-agent.rpm
sudo dnf install -y /tmp/ssm/amazon-ssm-agent.rpm
sudo systemctl stop amazon-ssm-agent
sudo -E amazon-ssm-agent -register -code $activationcode -id $activationid -region us-east-1
sudo systemctl start amazon-ssm-agent</code></pre> 
</div> 
<p><strong>Note that you should replace the URL in the example with the URL from your CloudFormation Stack Output.</strong></p> 
<p><strong>Windows:</strong></p> 
<p style="padding-left: 40px;">– A PowerShell script to retrieve Hybrid Activation credentials and install SSM Agent with the obtained credentials and register to the us-east-1 region:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-bash">$credential = Invoke-WebRequest -URI https://cla9phiczg.execute-api.us-east-1.amazonaws.com/lambdastage | Select-Object -ExpandProperty Content
$credentialPSObject = $credential | ConvertFrom-Json
 
$code = $credentialPSObject.ActivationCode
$id = $credentialPSObject.ActivationId
$region = "us-east-1"
$dir = $env:TEMP + "\ssm"
 
New-Item -ItemType directory -Path $dir -Force
cd $dir
(New-Object System.Net.WebClient).DownloadFile("https://amazon-ssm-$region.s3.$region.amazonaws.com/latest/windows_amd64/AmazonSSMAgentSetup.exe", $dir + "\AmazonSSMAgentSetup.exe")
Start-Process .\AmazonSSMAgentSetup.exe -ArgumentList @("/q", "/log", "install.log", "CODE=$code", "ID=$id", "REGION=$region") -Wait
Get-Content ($env:ProgramData + "\Amazon\SSM\InstanceData\registration")
Get-Service -Name "AmazonSSMAgent"
</code></pre> 
</div> 
<p><strong>Note: that you should replace the URL in the example with the URL from your CloudFormation Stack Output.</strong></p> 
<h2>Cleaning up</h2> 
<p>To clean up the environment, <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-managed-instances-advanced-deregister.html">deregister</a> the servers from Systems Manager. Then, <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html">delete the AWS CloudFormation stack</a> that you created in the walkthrough by deleting <strong>Create API Gateway and Lambda</strong> CloudFormation Stack first followed by <strong>Create VPC endpoint for API Gateway</strong> CloudFormation Stack. At last, <a href="https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html">delete the KMS key</a> created in the walkthrough.</p> 
<h2>Conclusion</h2> 
<p>In this post, I demonstrated how to automate Systems Manager Hybrid Activations creation. By adopting this solution, you can quickly register your hybrid environment devices to Systems Manager and minimize the overhead of managing the Hybrid Activations.</p> 
<p><strong>Author:</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/06/30/Justin-Thomas.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Justin Thomas</h3> 
  <p>Justin Thomas is a Cloud Support Engineer with AWS Premium Support. He specializes in AWS Systems Manager, Linux and Shell Scripting. Outside of work, Justin enjoys spending time with friends &amp; family, trying out new foods and watching movies.</p> 
 </div> 
</footer>
