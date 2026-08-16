---
title: "Multi-Cloud Observability with Amazon CloudWatch Using Bearer Token Auth and OpenTelemetry"
url: "https://aws.amazon.com/blogs/mt/achieve-multi-cloud-observability-with-amazon-cloudwatch-using-bearer-token-auth-and-opentelemetry/"
date: "2026-08-10"
author: "Imaya Kumar Jagannathan"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
Organizations running serverless workloads across multiple cloud providers face a specific observability challenge. There is no persistent compute to host a telemetry collector, no sidecar to attach, and no daemon running between invocations. The standard OpenTelemetry deployment model (application to local collector to a telemetry backend) does not apply in this environment.
