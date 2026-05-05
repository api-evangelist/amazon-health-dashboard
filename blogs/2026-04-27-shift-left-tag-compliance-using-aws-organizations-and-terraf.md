---
title: "Shift-Left Tag Compliance using AWS Organizations and Terraform"
url: "https://aws.amazon.com/blogs/mt/shift-left-tag-compliance-using-aws-organizations-and-terraform/"
date: "Mon, 27 Apr 2026 19:07:55 +0000"
author: "Welly Siauw"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
<p>Maintaining consistent resource tagging across development teams is one of the most common challenges we hear from customers. Without consistent tags, cost allocation reports become unreliable, attribute-based access controls (ABAC) fail to enforce permissions correctly, and automated operations skip untagged resources leaving teams unable to track spend, meet compliance requirements and respond adequately to incidents. Security and Center of Excellence (CoE) teams define tag structures and standards, but implementation depends on individual teams translating these policies into their Terraform configurations and modules. Organizations also face ongoing challenges as tag policies evolve and require continuous compliance validation.</p> 
<p>In this post, you’ll learn how security teams, CoE teams, and practitioners can collaborate by validating tag compliance during development before resources reach production.</p> 
<p><a href="https://aws.amazon.com/organizations/">AWS Organizations</a> provides a solution to the above challenge through centralized tag policy enforcement. AWS Organizations lets you define company-wide <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html">Tag Policies</a> that apply to all AWS accounts across your organization. Tag policies help you standardize tags across resources. When you apply a tag policy to your account, you can’t create resources with non-compliant tags for the selected resource types. The <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs">HashiCorp Terraform AWS Provider</a> added this capability in <a href="https://github.com/hashicorp/terraform-provider-aws/releases/tag/v6.22.0">v6.22</a>. You’ll learn how to implement this feature through the following steps:</p> 
<ol> 
 <li>Configure an AWS Organizations tag policy using Terraform</li> 
 <li>Enable tag enforcement in the Terraform AWS provider</li> 
 <li>Build a reusable tagging Terraform module</li> 
 <li>Write unit tests that validate against live organizational policies</li> 
</ol> 
<p>The sample code for this implementation is available in the <a href="https://github.com/aws-samples/sample-terraform-test-driven-tag-policy-compliance">aws-samples repository</a>. Please make sure to review and harden these configurations before using them in a production environment.</p> 
<h2>Prerequisites</h2> 
<p>Before you start, you need the following:</p> 
<ul> 
 <li>Terraform version <a href="https://github.com/hashicorp/terraform/releases">1.8.0 or later</a>.</li> 
 <li>Terraform AWS provider version 6.22.0 or later.</li> 
 <li>AWS credentials with sufficient permissions, including the ListRequiredTags AWS Identity and Access Management (IAM) permission.</li> 
 <li>Tag policies enabled in your AWS Organizations management account.</li> 
</ul> 
<h2>Configuring Tag Policies with Terraform</h2> 
<p>With the prerequisites in place, you’re ready to configure tag policies using Terraform. The latest version of the Terraform AWS provider lets you add policies of type TAG_POLICY targeting specific resource types using the <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/organizations_policy">aws_organizations_policy</a> resource. The following Terraform configuration snippet creates a tag policy that requires the Owner tag key on Amazon CloudWatch log groups:</p> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-code">resource "aws_organizations_policy" "example" {
   name = "tag-policy-example"
   content = jsonencode({
     "tags" : {
       "Owner" : {
         "tag_key" : {
           "@@assign" : "Owner"
         },
         "report_required_tag_for" : {
           "@@assign" : [
             "logs:log-group"
           ]
         }
       }
     }
   })
   type = "TAG_POLICY"
}
</code></pre> 
 </div> 
 <p>Next, you need to specify which AWS accounts the policy should target. The <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/organizations_policy_attachment">aws_organizations_policy_attachment</a> resource handles this configuration. You can extract the Organizational Unit (OU) ID from the management account and use it as the target for the policy attachment. The following configuration attaches the policy to the root of your organization:</p> 
 <div class="hide-language"> 
  <pre><code class="lang-code">data "aws_organizations_organization" "current" {}
 
resource "aws_organizations_policy_attachment" "example" {
   policy_id = aws_organizations_policy.example.id
   target_id = data.aws_organizations_organization.current.roots[0].id
}
</code></pre> 
 </div> 
 <p>After you run <code>terraform apply</code>, you should see the tag policy applied to your account with the specified targets (in this case, the entire OU).</p> 
</div> 
<div class="wp-caption aligncenter" id="attachment_64512" style="width: 854px;">
 <img alt="Figure 1: Screenshot showing organizational tag policy scope in the AWS Console" class="wp-image-64512 size-full" height="520" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/21/shift-left-tag-policy-with-terraform-figure-2.png" style="margin: 10px 0px 10px 0px;" width="844" />
 <p class="wp-caption-text" id="caption-attachment-64512">Figure 1: Organizational tag policy scope in the AWS Console</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_64513" style="width: 946px;">
 <img alt="Screenshot showing tag policy with policy details in the AWS Console" class="wp-image-64513 size-full" height="794" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2026/04/21/shift-left-tag-policy-with-terraform-figure-1.png" style="margin: 10px 0px 10px 0px;" width="936" />
 <p class="wp-caption-text" id="caption-attachment-64513">Figure 2: Tag policy with policy details in the AWS Console</p>
</div> 
<p>The <code>report_required_tag_for</code> property uses AWS service-specific resource type identifiers. In this example, <code>logs:log-group</code> maps to the <code>aws_cloudwatch_log_group</code> resource. For a complete mapping of tag resource types to Terraform resource types, see the <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/guides/tag-policy-compliance#resource-type-cross-reference">Terraform AWS Provider Tag Policy Compliance guide</a>. For detailed tag policy syntax and advanced configuration options, see the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html">AWS Organizations Tag Policies documentation</a>.</p> 
<h3>Enforcing Tag Policies Compliance in Terraform</h3> 
<p>With the tag policy in place, you can enable enforcement in your Terraform configurations. The Terraform AWS provider offers a simple one-line configuration via the <code>tag_policy_compliance</code> parameter to validate resources against organizational tag policies during terraform plan and terraform apply operations.</p> 
<p>The following code shows the basic setup. Refer to the example repository (2-testing-policy/main.tf) for the full configuration.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">provider "aws" {
   tag_policy_compliance = "error"
}
 
resource "aws_cloudwatch_log_group" "example" {
   name              = "required-tags-demo"
}
</code></pre> 
</div> 
<p>In this example, the log group has no tags. The configuration also doesn’t pass default tags using the provider block, which is a common approach to add tags across all resources provisioned in a Terraform configuration. With this configuration and the tag policy in place, running terraform plan against one of the accounts in the OU produces the following error:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">Plan: 2 to add, 0 to change, 0 to destroy.

 │ Error: Missing Required Tags - An organizational tag policy requires the following tags for aws_cloudwatch_log_group: [Owner]
</code></pre> 
</div> 
<p>This example demonstrates tag policy enforcement with a single configuration line in your Terraform provider block.</p> 
<h2>Best practices</h2> 
<p>Now that you understand how tag policy enforcement works, let’s explore best practices for implementing it in your organization. For general best practices on working with tag policies, see the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies-best-practices.html">AWS Organizations tag policies best practices</a> documentation. For Terraform-specific implementation, consider the following:</p> 
<ol> 
 <li>When you introduce tagging policies in your organization, start with <code>tag_policy_compliance</code> set to warning. This lets you and your development teams identify infrastructure workflows that don’t comply with the standards.</li> 
 <li>For enabling the <code>tag_policy_compliance</code> setting in your provider configuration at scale, use the TF_AWS_TAG_POLICY_COMPLIANCE environment variable. The environment variable provides two advantages over the provider argument: 
  <ul> 
   <li>You can enable or disable enforcement without modifying existing configuration</li> 
   <li>You can set different behavior per environment (warning in development, error in production)</li> 
   <li>When you set both the environment variable and provider argument, the provider argument takes precedence.</li> 
  </ul> </li> 
</ol> 
<ol start="3"> 
 <li>To ensure consistency of tag implementation, leverage a Terraform module that can help developers implement the required tag consistently.</li> 
 <li>Ensure your tag module is always up to date and aligned with your organization’s tag policy. Periodically test your tag module against live tag policy.</li> 
</ol> 
<p>In the next section, we will dive deeper into these best practices.</p> 
<h2>Streamline the developer experience</h2> 
<p>Development teams often struggle to comply with organizational tag policies because tracking every tagging requirement manually is error prone. A Terraform module that sets the necessary tags provides a good starting point. The following example shows how to implement this approach. The complete module code is present in the repository (3-tag-module/modules/tags).</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">locals {
   required_tags = {
     Owner = coalesce(var.owner, data.aws_caller_identity.current.account_id)
   }
}
  
output "tags" {
   description = "Merged tags with required Owner tag"
   value       = merge(var.additional_tags, local.required_tags)
}</code></pre> 
</div> 
<p>The module automatically sets the Owner tag to the account ID when not explicitly provided. This ensures teams can use this module consistently across their configurations. Development teams complete two steps:</p> 
<ol> 
 <li>Invoke the module</li> 
 <li>Reference the module’s tags output in any resources or modules that use a tags input</li> 
</ol> 
<p>The following example invokes the tagging module and applies the output to a resource:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">provider "aws" {
   tag_policy_compliance = "error"
}
 
module "tags" {
   source = ".././"
}
 
resource "aws_cloudwatch_log_group" "example" {
   name              = "compliant-log-group"
   tags              = module.tags.tags
}
</code></pre> 
</div> 
<p>When the developer runs terraform plan, the module handles the required tags and passes them to the target resources being deployed. With the module’s defaults, the module adds the Owner tag to the provisioned resource even when the developer doesn’t pass a specific value for the owner input variable. This approach ensures compliance with required tagging standards while minimizing developer effort.</p> 
<h3>Test driven tag compliance in Terraform</h3> 
<p>With the tagging module in place, the next step is ensuring it remains accurate as your tag policies evolve. The reusable tag module helps development teams stay compliant, but you need to validate the module itself with tests. Without tests, module updates might silently drift from the actual tag policy. Terraform’s <a href="https://developer.hashicorp.com/terraform/language/tests">built-in test framework</a> makes this straightforward. Tests run against the module’s planned output without creating real infrastructure, giving you early feedback with minimal overhead.</p> 
<p>Terraform test files use the <code>.tftest.hcl</code> extension and live in a tests directory alongside your module. Each file has one or more run blocks, with each run block setting a command (plan or apply), optional variable overrides, and assert blocks that validate outputs. A provider block lets you set provider configuration for the test context. You can set the attribute <code>tag_policy_compliance</code> = “error” so that a missing required tag fails the test immediately.</p> 
<p>With the tag key <code>Owner</code> in mind, we can build test scenarios such as:</p> 
<ol> 
 <li>Validate the module output with an explicit value set for Owner</li> 
 <li>Validate the module output when no input value is provided for the variable Owner</li> 
 <li>Validate the module output contains all required tags</li> 
</ol> 
<p>The following test file demonstrates how to validate tag values using Terraform’s built-in test framework. For full test suite, please refer to repository (3-tag-module/modules/tags/tests)</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">provider "aws" {
   tag_policy_compliance = "error"
   region                = "us-east-1"
}
 
run "test_with_explicit_owner" {
  command = plan
 
  variables {
     owner = "test-team"
     additional_tags = {
       Environment = "test"
     }
  }
 
  assert {
     condition     = output.tags["Owner"] == "test-team"
     error_message = "Owner tag should be set to test-team"
  }
 
  assert {
     condition     = output.tags["Environment"] == "test"
     error_message = "Environment tag should be set to test"
  }
}
</code></pre> 
</div> 
<p>Because the tests only validate module outputs and data source reads, there’s no need to provision real infrastructure, keeping the tests fast and cost-free.</p> 
<p><strong>Running the tests</strong></p> 
<p>To start the test, ensure you’re authenticated into an AWS account and run <code>terraform init</code> in the root directory of the module. Once all modules are initialized, run <code>terraform test</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">$terraform test
 
tests/tags.tftest.hcl... in progress
   run "test_with_explicit_owner"... pass
   run "test_with_default_owner"... pass
   run "test_required_tags_present"... pass
 tests/tags.tftest.hcl... tearing down
 tests/tags.tftest.hcl... pass
 
Success! 3 passed, 0 failed.</code></pre> 
 <p>These tests validate that the module correctly handles both explicit and default Owner tag values, ensuring compliance with your organizational tag policies.</p> 
 <p><strong>Risk of test scenario drift</strong></p> 
 <p>Though the tests run successfully, they have a blind spot. The tests are only aware of the hardcoded tag keys in the module or tests. When the team who owns the compliance policies updates the policy to include new required tags, the tests won’t catch it, leading to plan failures for development teams who use the tag module. For example, assume the CoE adds another tag key called <code>Environment</code> as required with some allowed values. Now if you run the same Terraform test scenario, it will pass even though the <code>Environment</code> tag is not present. This demonstrates the risk of test drift. Your tests pass, but your module no longer complies with the actual organizational policy, leading to failures when developers use it. Full example available in the repository directory (4a-configure-policy-updated/policies/).</p> 
 <p><strong>Dynamic tag compliance test</strong></p> 
 <p>To solve the test drift problem, you need a dynamic approach. Instead of hardcoding expected tags, the test can read the live organizational policy and derive its assertions from that. This way, if the policy changes, the test automatically reflects it. To do this, you need to know the organization root ID and policies attached to it. This requires you to be on a management account or a delegated administrator account.</p> 
 <p>In a Terraform test, you can instruct the run block to execute another helper module. The helper module creates supporting resources for the test. We create a helper module called setup as a subdirectory under the tests directory. The setup module uses data source to list all tag policies attached to that account or the OU and fetch the full content of each policy. The module also parses the JSON content of every tag policy from the data source output and extracts the <code>@@assign</code> under each <code>tag_key</code> block, which reflects the actual tag keys. Full example available in the repository (4b-tag-module-test-driven/modules/tags/tests/setup).</p> 
 <div class="hide-language"> 
  <pre><code class="lang-code"># Retrieve all tag policies from target account + root
locals {
   all_policy_ids = distinct(concat(
     data.aws_organizations_policies_for_target.account.ids,
     data.aws_organizations_policies_for_target.root.ids,
   ))
}
 
data "aws_organizations_policy" "tag_policy" {
   count     = length(local.all_policy_ids)
   policy_id = local.all_policy_ids[count.index]
}
 
locals {
   # Parse all tag policies
   all_policies = [for p in data.aws_organizations_policy.tag_policy : jsondecode(p.content)]
   
   # Extract actual tag keys from policies
   required_tags = length(local.all_policies) &gt; 0 ? flatten([
     for policy in local.all_policies : [
       for tag_name, tag_config in lookup(policy, "tags", {}) :
         lookup(lookup(tag_config, "tag_key", {}), "@@assign", tag_name)
     ]
   ]) : []
}
 
output "policy_content" {
   value = [for p in data.aws_organizations_policy.tag_policy : p.content]
}
 
output "has_policies" {
   value = length(data.aws_organizations_policy.tag_policy) &gt; 0
}
 
output "required_tag_keys" {
   value = local.required_tags
   description = "List of all required tag keys from tag policies"
}
</code></pre> 
 </div> 
 <p>With the setup module in place, you can write an integration test that reads the current tag policy and derives assertions dynamically. Using Terraform’s provider aliasing feature, you can specify which account or AWS profile to use for specific operations in the tests. The test uses an AWS provider with a local profile called <code>management</code> and an alias set to <code>management</code>. This way the setup module runs against the management account, retrieving the necessary details to identify the current tag policies that are applied, while the actual tests can use the default provider associated to the account you’re targeting for resource provisioning.</p> 
 <div class="hide-language"> 
  <pre><code class="lang-code">provider "aws" {
   tag_policy_compliance = "error"
}
 
provider "aws" {
   profile = "management"
   alias   = "management"
}
 
run "setup" {
   providers = {
     aws = aws.management
   }
 
  command = plan
 
  module {
     source = "./tests/setup"
   }
}
 
run "test_tag_key_policy_compliance" {
   command = plan
 
  variables {
     owner = "compliance-test"
   }
 
  assert {
     condition     = run.setup.has_policies
     error_message = "Prereq: Should be able to read tag policies from organization"
   }
 
  assert {
     condition     = alltrue([for required_key in run.setup.required_tag_keys : contains(keys(output.tags), required_key)])
     error_message = "Tags must include all required keys from tag policy: ${join(", ", run.setup.required_tag_keys)}"
   }
}
</code></pre> 
 </div> 
 <p>The introduction of the helper module requires you to run <code>terraform init</code> again before <code>terraform test</code>. Once you re-run <code>terraform test</code>, it provides immediate feedback that the Environment key is missing from the required tags:</p> 
 <div class="hide-language"> 
  <pre><code class="lang-code">tests/tags.tftest.hcl... in progress
   run "setup"... pass
   run "test_tag_key_policy_compliance"... fail

 │ Error: Test assertion failed
 │
   . . .
                                                                                                      
 │ Tags must include all required keys from tag policy: Environment, Owner
 ╵
 tests/tags.tftest.hcl... tearing down
 tests/tags.tftest.hcl... fail
 
Failure! 1 passed, 1 failed.
</code></pre> 
 </div> 
 <p>Tag policies evolve and your CoE team can update the required tags or accepted tag values based on the organizational changes. As a module author, you can stay ahead of policy changes by scheduling your tests to run automatically on your Continuous Integration (CI) tool on a cadence (nightly or weekly) pulling the live policies with each run. When your CoE adds a new required tag such as CostCenter, the test execution catches the failure and notifies you of the changes. This notification prompts you to update the module, release a new version, and notify consumers to upgrade.</p> 
 <h2>Cleaning up</h2> 
 <p>To avoid incurring future charges, clean up the resources you created in this walkthrough. Navigate to the directory where you deployed the tag policy and run <code>terraform destroy</code>. This command removes the tag policy and its attachment to your OU.</p> 
 <h2>Conclusion</h2> 
 <p>In this post, you learned about a layered approach to tag policy enforcement in your infrastructure provisioning workflow using Terraform. This post covered AWS Organizations tag policies, the <code>tag_policy_compliance</code> Terraform provider setting, a reusable tagging module that automatically applies required tags, and a test-driven approach that dynamically validates against your organizational policies. This approach validates tag compliance during development, closing the gap between policy definition and implementation. By reading live tag policies at test time, your Terraform modules automatically stay synchronized with organizational requirements, eliminating manual updates when policies change.</p> 
 <p>To get started, check out the <a href="https://github.com/aws-samples/sample-terraform-test-driven-tag-policy-compliance">sample repository</a> with the complete working examples with dynamic tests. For the latest policy updates, see the <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/guides/tag-policy-compliance">Terraform AWS Provider Tag Policy Compliance guide</a>, and the <a href="https://developer.hashicorp.com/terraform/language/tests">Terraform test documentation</a>. For additional information about AWS Organizations tag policy enforcement for Infrastructure as Code, see the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/enforce-required-tag-keys-iac.html">Enforce “Required tag key” with IaC</a> doc page.</p> 
</div>
