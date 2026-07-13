---
title: "Using Amazon S3 Server Access Logs with Amazon CloudWatch Logs"
url: "https://aws.amazon.com/blogs/mt/using-amazon-s3-server-access-logs-with-amazon-cloudwatch-logs/"
date: "2026-07-02"
author: ""
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
This post shows how to turn raw S3 server access logs into a complete security dashboard using CloudWatch's native S3 log integration, without building a custom pipeline. It covers enabling log ingestion via Telemetry Enablement Rules, querying with CloudWatch Logs Insights, setting Metric Filters for threshold alerting, and using Contributor Insights to spot top data consumers, with a deployable CloudFormation template for detecting unauthorized access and validating TLS/encryption compliance.
