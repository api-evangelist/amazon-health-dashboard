# Amazon Health Dashboard (amazon-health-dashboard)
AWS Health Dashboard provides ongoing visibility into the status of your AWS services, and alerts and remediation guidance when AWS is experiencing events that may affect your operations. It delivers personalized information about events that might affect your specific AWS resources and accounts.

**URL:** [https://aws.amazon.com/health/](https://aws.amazon.com/health/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Health Monitoring, Notifications, Operations, Service Status

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS Health API
The AWS Health API provides programmatic access to AWS Health information about events that can affect your AWS infrastructure, including service outages, planned maintenance, and account-specific notifications.

**Human URL:** [https://aws.amazon.com/health/](https://aws.amazon.com/health/)

#### Properties

- [Documentation](https://docs.aws.amazon.com/health/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-health-dashboard-openapi.yaml)
- [GettingStarted](https://docs.aws.amazon.com/health/latest/ug/getting-started-api.html)
- [APIReference](https://docs.aws.amazon.com/health/latest/APIReference/Welcome.html)
- [JSONSchema](json-schema/health-event-schema.json)
- [JSONLD](json-ld/amazon-health-dashboard-context.jsonld)

## Common Properties

- [Portal](https://health.aws.amazon.com/health/home)
- [Documentation](https://docs.aws.amazon.com/health/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/mt/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/health/)

## Features

| Name | Description |
|------|-------------|
| Personalized Health Notifications | Alerts tailored to the AWS services and resources in your accounts. |
| Proactive Event Notifications | Advance notice of planned maintenance, deprecations, and changes. |
| Remediation Guidance | Specific guidance on what actions to take to minimize impact. |
| Organization-Wide Visibility | View events across all accounts in an AWS Organization. |
| Affected Resource Identification | Identify exactly which EC2 instances or RDS databases are impacted. |
| Event History | Access up to 90 days of event history. |

## Use Cases

| Name | Description |
|------|-------------|
| Operations Monitoring | Monitor AWS service health in real-time to detect and respond to events. |
| Automated Incident Response | Trigger automated runbooks when health events affect resources. |
| Change Management | Track planned maintenance to coordinate deployments. |
| Compliance Reporting | Maintain records of AWS service events for compliance. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon EventBridge | Receive health events as EventBridge events for automated responses. |
| AWS Organizations | View events across all member accounts. |
| AWS Support | Health events link directly to AWS Support cases. |
| Amazon CloudWatch | Create CloudWatch alarms based on health event metrics. |
| AWS Chatbot | Receive notifications in Slack or Chime. |

## Artifacts

### OpenAPI

- [Amazon Health Dashboard OpenAPI](openapi/amazon-health-dashboard-openapi.yaml)

### JSON Schema

103 schema files in [json-schema/](json-schema/)

### JSON Structure

99 structure files in [json-structure/](json-structure/)

### JSON-LD

- [Amazon Health Dashboard Context](json-ld/amazon-health-dashboard-context.jsonld)

### Examples

99 example files in [examples/](examples/)

## Capabilities

### Shared Per-API Definitions

- [Amazon Health Dashboard](capabilities/shared/amazon-health-dashboard.yaml) — 5 operations for health event monitoring

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Amazon Health Dashboard Operations Monitoring](capabilities/amazon-health-dashboard-operations-monitoring.yaml) | Amazon Health Dashboard | 7 | Operations Engineer, DevOps Engineer, Cloud Administrator |

## Vocabulary

- [Amazon Health Dashboard Vocabulary](vocabulary/amazon-health-dashboard-vocabulary.yaml) — Unified taxonomy mapping 4 resources, 2 actions, 1 workflow, and 3 personas

## Rules

- [Amazon Health Dashboard Spectral Rules](rules/amazon-health-dashboard-spectral-rules.yml) — 7 rules enforcing Amazon Health Dashboard API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
