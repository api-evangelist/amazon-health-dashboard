---
title: "Introducing native histogram support in Amazon Managed Service for Prometheus"
url: "https://aws.amazon.com/blogs/mt/introducing-native-histogram-support-in-amazon-managed-service-for-prometheus/"
date: "2026-06-12"
author: "Vinod Kisanagaram"
feed_url: "https://aws.amazon.com/blogs/mt/feed/"
---
If you run Kubernetes or microservices workloads on AWS, you probably track latency, request durations, and other value distributions with Prometheus histograms. To do that with classic histograms, you predefine a set of bucket boundaries, and Prometheus emits one time series per boundary plus a sum and a count. A single latency histogram with 20 […]
