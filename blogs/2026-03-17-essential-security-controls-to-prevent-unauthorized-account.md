---
title: "Essential security controls to prevent unauthorized account removal in AWS Organizations"
url: "https://aws.amazon.com/blogs/mt/essential-security-controls-to-prevent-unauthorized-account-removal-in-aws-organizations/"
date: "Tue, 17 Mar 2026 15:43:50 +0000"
author: "Nivedita Tripathi"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<p>When AWS member accounts are compromised, attackers can remove them from your organization, disabling all governance controls. In this post, you’ll learn how to protect your AWS environment from account compromise leaving your AWS Organization using layered security controls, including service control policies, secure account migration, and centralized root access management.</p> 
<p>AWS secures the infrastructure that runs the cloud, while you’re responsible for securing your workloads and data in the cloud. To do so, AWS Organizations includes security controls that prevent unauthorized account removal and maintain governance across your accounts.</p> 
<p>We will cover four controls: preventing unauthorized account removal with a Service control policy (SCP), establishing break-glass procedures for legitimate migrations, transferring accounts directly between organizations, and disabling root access for member accounts.</p> 
<p><strong>Prerequisites</strong><br /> For this post, you must be familiar with&nbsp;<a href="https://aws.amazon.com/organizations/">AWS Organizations</a>&nbsp;and&nbsp;<a href="https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html">multi-account strategy concepts</a>. You must have access to the AWS Organizations management account with permissions to:</p> 
<ul> 
 <li>Create and attach service control policy — <code>organizations:CreatePolicy</code>, <code>organizations:AttachPolicy</code></li> 
 <li>Manage organizational units —<code>organizations:CreateOrganizationalUnit</code>, <code>organizations:MoveAccount</code></li> 
 <li>Enable centralized root access management — <code>iam:EnableOrganizationsRootCredentialsManagement</code></li> 
 <li></li> 
</ul> 
<p><strong>Design your organizational unit (OU) structure for security and flexibility&nbsp;</strong><br /> If your business model requires regularly allowing member accounts to leave the organization such as managed service providers, resellers, or organizations with high account turnover, then you should design your OU structure with this workflow in mind from the start.</p> 
<p>Consider creating dedicated OUs for different account lifecycle stages (onboarding, active, off-boarding) and apply the <a href="https://docs.aws.amazon.com/organizations/latest/APIReference/API_LeaveOrganization.html">DenyLeaveOrganization</a> SCP only to OUs containing accounts that should remain under long-term governance. This approach secures your core infrastructure and simplifies migration for short-lived accounts.</p> 
<p><a href="https://docs.aws.amazon.com/organizations/latest/userguide/create_ou.html">Create a hierarchical OU structure</a> that balances security controls with operational flexibility. Apply the DenyLeaveOrganization SCP to your production and development OUs to protect critical workloads, while maintaining a separate transition OU without this restriction for controlled account migrations.</p> 
<p>Recommended OU architecture</p> 
<ul> 
 <li>Production OU: Apply the DenyLeaveOrganization SCP to prevent accounts running production workloads from leaving your organization</li> 
 <li>Development OU: Apply the same SCP to maintain governance over development and testing environments</li> 
 <li>Transition OU: Keep this OU free from the DenyLeaveOrganization SCP to serve as a controlled staging area for accounts preparing to leave the organization</li> 
</ul> 
<p>When an account needs to leave your organization for legitimate business reasons, your management account can move it to the Transition OU, where the leave operation for member account becomes possible. This creates a clear approval workflow and maintains visibility throughout the migration process.</p> 
<p><em>Please note that the management account can remove member accounts from the organization, so if there’s no legitimate need for member accounts to leave on their own, you don’t need to implement the Transition OU approach for leave accounts.</em></p> 
<p>For detailed guidance on organizing and managing your OU structure, follow&nbsp;<a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html">AWS best practices for managing organizational units with AWS Organizations</a>.</p> 
<p><strong>Implement a service control policy that denies leave organizations actions&nbsp;</strong><br /> To prevent member accounts from leaving your organization, implement an SCP that denies the <code>organizations:LeaveOrganization</code> and action for member accounts. This preventative control ensures accounts remain within your governance framework, keeping your security controls and organizational policies in place.</p> 
<h3><strong>Create an SCP using AWS Management Console</strong></h3> 
<p>1. Sign in to the AWS Organizations console with your management account.<br /> 2. In the navigation pane, choose <strong>Policies</strong>.<br /> 3. Choose and <a href="https://docs.aws.amazon.com/organizations/latest/userguide/enable-policy-type.html">enable service control policies</a>.<br /> 4. Choose <strong>Create policy</strong>.<br /> 5. Enter a policy name, e.g., <em>“DenyLeaveOrganization”.</em><br /> 6. In the policy editor, enter the following JSON:</p> 
<p><code>{</code></p> 
<p><code>&nbsp; "Version": "2012-10-17",</code></p> 
<p><code>&nbsp; "Statement": [</code></p> 
<p><code>&nbsp;&nbsp;&nbsp; {</code></p> 
<p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Effect": "Deny",</code></p> 
<p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Action": "organizations:LeaveOrganization",</code></p> 
<p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Resource": "*"</code></p> 
<p><code>&nbsp;&nbsp;&nbsp; }</code></p> 
<p><code>&nbsp; ]</code></p> 
<p><code>}</code></p> 
<p>7. Click <strong>Create policy</strong>.</p> 
<h3><strong>Create the SCP using AWS CLI</strong></h3> 
<p>Run the following command to create the policy.</p> 
<p><code>aws organizations create-policy \</code></p> 
<p><code>--name DenyLeaveOrganization \</code></p> 
<p><code>--type SERVICE_CONTROL_POLICY \</code></p> 
<p><code>--description "Prevents member accounts from leaving the organization" \</code></p> 
<p><code>--content '{"Version":"2012-10-17","Statement":[{"Effect":"Deny","Action":"organizations:LeaveOrganization","Resource":"*"}]}'</code></p> 
<p><em>Note the policy ID from the output for use in the next step.</em></p> 
<p>2. After creating the policy, apply the DenyLeaveOrganization SCP to your <strong>Production</strong> and <strong>Development OUs</strong> while keeping the <strong>Transition OU</strong> unrestricted.</p> 
<h3><strong>Attach the SCP to your OUs using AWS Management Console</strong></h3> 
<ol> 
 <li>Navigate to AWS Organizations.</li> 
 <li>Select the target OU , <strong>Production</strong> or <strong>Development</strong>.</li> 
 <li>Choose the <strong>Policies</strong> tab.</li> 
 <li>Choose <strong>Attach</strong> and select the ‘DenyLeaveOrganization’ policy.</li> 
 <li>Repeat for each OU that requires the policy.</li> 
</ol> 
<h3><strong>Attach the SCP to your OUs using AWS CLI</strong></h3> 
<ol> 
 <li>Attach the policy to your target OUs using the policy ID from the creation step:<code>aws organizations attach-policy --policy-id p-xxxxxxxx --target-id r-xxxx&nbsp;</code></li> 
 <li>Verify the policy attachment: <code>aws organizations list-targets-for-policy --policy-id p-xxxxxxxx</code></li> 
 <li>Repeat for each OU that requires the policy.</li> 
</ol> 
<p>This control blocks leave attempts at the API level through the console, CLI, or SDK. For best practices of protecting your management account, refer to the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_best-practices_mgmt-acct.html">documentation</a>.</p> 
<p><em>Please note that SCPs don’t apply to the management account and only affect member accounts in the organization. This is why protecting your management account with the strongest possible security controls – including MFA, restricted access, temporary credentials only, and comprehensive monitoring is essential.</em></p> 
<h4><strong>Document break-glass procedures</strong></h4> 
<p>If you need to allow specific accounts to leave the organization during account migrations, mergers, acquisitions, or divestitures, establish a formal process with documented approval workflows and technical mechanisms.</p> 
<p>Choose the appropriate break-glass mechanism for your scenario: When an account needs to leave your organization, move it from its current OU to the Transition OU through your standard change management process. Once in the Transition OU, the account can execute the leave operation without the DenyLeaveOrganization SCP restriction. This approach maintains security controls on all active accounts without requiring you to temporarily remove SCPs from your entire organization root.</p> 
<h4><strong>Establish a documented exception process for member account departures</strong></h4> 
<p>Member accounts leaving your organization independently – outside of the secure invitation-based migration process – should be treated as exceptions requiring formal approval. Each exception request must include clear business justification explaining why the account cannot use the standard organization-to-organization transfer process, along with review and approval from security and IT administration teams to validate alignment with security and governance policy requirements.</p> 
<p>Document this exception in your security baseline, explicitly noting that the Transition OU exists solely for temporary account staging during approved migrations where the standard invitation-based transfer cannot be used. This documentation establishes accountability, creates an audit trail for compliance reviews, and ensures that exceptions to your security controls are intentional, justified, and time-bound.</p> 
<h4><strong>Securely migrate accounts during mergers and acquisitions&nbsp;</strong></h4> 
<p>During mergers, acquisitions, or organizational consolidations, AWS Organizations supports secure, direct account transfers between organizations- preventing accounts from becoming standalone. The destination organization’s management account sends an invitation to the migrating account. When accepted, the account transitions seamlessly from the source to the destination organization without ever operating outside organizational governance. Service control policies (SCPs), logging, and monitoring apply continuously throughout the migration, maintaining security posture and creating complete audit trails via <a href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html">AWS CloudTrail</a>.</p> 
<p>This invitation-based approach addresses most legitimate migration use cases while preventing the security gaps that occur when accounts operate independently. Management accounts don’t need to manually remove member accounts – the invitation process handles transitions automatically. After migration, validate that appropriate policies apply, update IAM policies with the correct organization ID, and review billing configurations, tax settings, and Reserved Instances transfers. For detailed guidance, refer to the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_account_migration.html">AWS Organizations User Guide on account migration</a>.</p> 
<h4><strong>Eliminate root access vulnerabilities in member accounts</strong></h4> 
<p>Root user credentials in member accounts represent the highest level of privileged access in your AWS environment. Since the 2025 launch of AWS Centralized Root Access Management, newly created member accounts no longer have root credentials by default – this is now the standard behavior for all new accounts in your organization. Newly created member accounts are automatically provisioned without root credentials, eliminating the need for post-provisioning security measures like configuring multi-factor authentication</p> 
<p>For existing accounts created before this default took effect, removing long-lived root credentials is quick and straightforward. AWS Centralized root access management lets you delete root credentials – including passwords, access keys, signing certificates, and MFA devices across your entire organization directly from the management account. You don’t need to sign in to each member account individually.</p> 
<p>This centralized approach maintains consistent security across member accounts. It also reduces operational overhead by removing the vulnerability gap between account creation and security configuration. For detailed guidance on implementing centralized root access management, refer to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-enable-root-access.html">centrally managing root access for member accounts</a> for more information.</p> 
<h3><strong>Delete root credentials using the AWS Management Console&nbsp;</strong></h3> 
<ol> 
 <li>Open the IAM console using the management account.</li> 
 <li>In the navigation pane under <strong>Access Management</strong>, choose<strong> root access management</strong>.</li> 
 <li>If centralized root access management has not yet been enabled, a banner or prompt will appear at the top of the page.</li> 
 <li>Choose <strong>Enable</strong> to activate it. If no such prompt appears, the feature is already enabled in your organization. Once enabled, the page displays your organizational structure with member accounts.</li> 
 <li>Select an account and review the Root user credentials panel on the right. For any account that still has active root credentials, choose Delete root credentials to remove them.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_64261" style="width: 1440px;">
 <img alt="Root access management" class="wp-image-64261 size-full" height="440" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/03/16/Root-access-management.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-64261">Image 1: Root access management</p>
</div> 
<h3><strong>Enable centralized root access management using the AWS CLI</strong></h3> 
<ol> 
 <li>Enable centralized root access management by running the following command (safe to run even if already enabled – returns current feature state), aws iam enable-organizations-root-credentials-management</li> 
 <li>Verify enabled features to confirm the configuration, aws iam list-organizations-features</li> 
 <li>Review the output to confirm successful enablement. If the feature has been enabled, you’ll see the below:</li> 
</ol> 
<p><code>{</code></p> 
<p><code>"OrganizationId": "o-xxxxxxxxxxxx",</code></p> 
<p><code>"EnabledFeatures": [</code></p> 
<p><code>"RootSessions",</code></p> 
<p><code>"RootCredentialsManagement"</code></p> 
<p><code>]</code></p> 
<p><code>}</code></p> 
<p>If the feature has not yet been enabled,&nbsp;<code>aws iam list-organizations-features</code> will return an empty EnabledFeatures array as shown here.</p> 
<p><code>{</code></p> 
<p><code>"OrganizationId": "o-xxxxxxxxxxxx",</code></p> 
<p><code>"EnabledFeatures": []</code></p> 
<p><code>}</code></p> 
<p><strong>Conclusion</strong><br /> Protecting your AWS Organizations from account compromise requires layered security controls that balance protection with operational flexibility. The DenyLeaveOrganization service control policy blocks unauthorized account removal and maintains continuous governance oversight. The invitation-based account migration capability across organizations supports legitimate business needs like mergers, acquisitions, and consolidations, without creating security gaps. Eliminating root access through AWS Centralized Root Access Management removes the highest-privilege pathway that could bypass your security controls.</p> 
<p>These controls prevent compromised credentials from removing accounts from your organization, keep service control policies and logging active during migrations, and to ensure that security incidents remain containable within your governance framework, so you can detect, respond to, and remediate issues faster.</p> 
<p>Start by designing your OU structure, document your break-glass procedures, then apply the DenyLeaveOrganization SCP and enable AWS Centralized Root Access Management. Regularly review your OU structure, audit exception requests, and monitor for unauthorized access attempts through AWS CloudTrail. Treat account governance as a critical security control to keep your AWS environment secure, compliant, and aligned with your business objectives.&nbsp;For more service control policy examples and templates, explore the&nbsp;<a href="https://github.com/aws-samples/service-control-policy-examples">AWS SCP Examples GitHub repository</a>.</p>
