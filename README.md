# AI FinOps Platform

Real-time cost observability for AI/ML infrastructure, streaming GPU utilisation and API spend events through Kafka for anomaly detection and optimisation.

## Overview

Organisations running AI workloads face a unique cost challenge: GPU instances and API calls to models like GPT-4 can burn through budgets in hours if left unmonitored. Traditional cloud cost tools operate on daily billing cycles—far too slow when a misconfigured training job can consume thousands in compute before anyone notices.

This platform provides real-time cost visibility by streaming infrastructure metrics and API cost events through Apache Kafka. OpenCost tracks Kubernetes resource allocation at the pod level, Prometheus scrapes utilisation metrics, and Grafana visualises cost trends as they happen. The event-driven architecture enables immediate alerting on cost anomalies rather than discovering overruns in next month's invoice.

The infrastructure runs on AWS EKS with Strimzi managing the Kafka cluster. Dedicated topics separate GPU utilisation events, AI API costs, and detected anomalies—enabling downstream consumers to process each stream independently for alerting, reporting, or automated scaling decisions.

## Architecture

![Cloud Architecture](screenshots/cloud-architecture.png)

The platform follows an event-driven architecture centred on Apache Kafka as the message backbone. The EKS cluster spans three availability zones for high availability, with Kafka running as a 3-broker cluster managed by the Strimzi operator.

**Data flow**: OpenCost and Prometheus collect cost and utilisation metrics from the Kubernetes cluster → Events are published to Kafka topics (gpu-utilization-events, ai-api-costs, cost-anomalies) → Grafana dashboards consume metrics for real-time visualisation → Anomaly detection consumes the cost streams and publishes to the anomalies topic.

The Terraform configuration provisions the complete AWS infrastructure: a VPC with public/private subnets across three AZs, an EKS cluster with managed node groups, and outputs the commands needed to access monitoring tools.

## Tech Stack

**Infrastructure**: AWS EKS, Terraform, VPC with 3 AZs, NAT Gateway  
**Streaming**: Apache Kafka 3-broker cluster, Strimzi Operator, ZooKeeper  
**Cost Monitoring**: OpenCost, Prometheus  
**Visualisation**: Grafana  
**Container Orchestration**: Kubernetes 1.28

## Key Decisions

- **Strimzi over self-managed Kafka**: Kubernetes-native operator simplifies Kafka lifecycle management, handles broker scaling, and integrates with K8s RBAC. Running Kafka on K8s without an operator requires significant operational overhead.

- **Ephemeral storage for demo environment**: Production would use persistent volumes, but ephemeral storage reduces costs and complexity for demonstrating the architecture. The configuration is structured to swap storage type with a single parameter change.

- **10 partitions for cost event topics, 3 for anomalies**: High-volume streams (GPU events, API costs) need parallelism for consumer throughput. Anomaly events are lower volume but higher priority—fewer partitions simplify ordering guarantees.

- **Single NAT Gateway**: Cost optimisation for demo purposes. Production deployments would use one NAT gateway per AZ for resilience, accepting the ~$30/month additional cost per gateway.

## Screenshots

![EKS cluster](screenshots/eks-cluster.png)

![Kafka topics](screenshots/kafka-topics.png)

![Grafana dashboard](screenshots/grafana-dashboard.png)

![OpenCost UI](screenshots/opencost-ui.png)

![Platform pods](screenshots/pods-running.png)

## Author

**Noah Frost**

- Website: [noahfrost.co.uk](https://noahfrost.co.uk)
- GitHub: [github.com/nfroze](https://github.com/nfroze)
- LinkedIn: [linkedin.com/in/nfroze](https://linkedin.com/in/nfroze)