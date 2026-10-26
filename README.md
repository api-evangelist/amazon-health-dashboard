# Amazon Health Dashboard (amazon-health-dashboard)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
