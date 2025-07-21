# Real Estate Finder Platform Network Architecture

## Assumptions

-The platform will run on AWS as the primary cloud provider.

-Two AWS regions will be used: one in the US (us-east-1) and one in Europe (eu-west-1).

-Each region uses two Availability Zones (AZs) for high availability.

-Users will access the platform via a global CDN.

-Developers access services through a secure VPN and bastion hosts.

-Real-time chat uses managed WebSocket service.

-Third-party data will be ingested via secure APIs.

-Backend services will be containerized and deployed on ECS or EKS.

-The database will be a managed multi-region RDS cluster for transactional data, with Redis for caching.

-Storage of images/listings uses S3 with replication.

-Monitoring & logging with CloudWatch and centralized logging solution.

## Project Details

This project is a multi-region real estate finder platform where users can search for houses globally, place bids, and chat with buyers or sellers. It also pulls property data from third-party APIs covering multiple countries and cities. High availability, scalability, and security are essential.
Architecture Decisions.

## Architecture Decisions

The architecture uses two AWS regions with two Availability Zones each. A global AWS Route 53 DNS service with latency-based routing directs users to the nearest region. An AWS CloudFront CDN caches static content. Application Load Balancers distribute traffic to containerized backend services running on ECS/EKS in private subnets. RDS Multi-AZ Aurora clusters handle transactional data with read replicas for scale. Redis is used for session caching and frequently accessed queries.
Reasoning

## Reasoning

This design supports high availability (multi-AZ, multi-region), low latency (CDN, regional routing), and scalability (auto-scaling containers, read replicas). A VPN plus bastion hosts provide developers secure access to private subnets. Third-party API calls run in isolated VPC endpoints to secure ingress traffic. VPC peering connects both regions for data sync.

## Networking Components

- **VPCs (per region)**: Isolate all resources securely.

- **Subnets (public/private)**: Public for Load Balancers/Bastion, private for backend/data.

- **Internet Gateway**: Allow internet traffic to ALBs.

- **NAT Gateway**: Allow outbound internet for backend servers.

- **VPN & Bastion**: Secure developer access.

- **CloudFront CDN**: Global caching.

- **Route 53**: DNS & Geo-routing.

- **VPC Peering**: Cross-region connectivity.

- **Endpoints**: Secure API calls to 3rd parties.

## Cost Estimates

The following table estimates monthly costs for different user scales, based on AWS pricing (ap-southeast-1, October 2023). Costs include compute, database, caching, networking, and third-party API calls (assumed at $0.01 per 1,000 requests).

| Users                     | Compute (ECS/EKS) | RDS (Aurora Multi-AZ) | Redis   | CloudFront | Data Transfer | Total Estimate (Monthly) |
| ------------------------- | ----------------- | --------------------- | ------- | ---------- | ------------- | ------------------------ |
| **100 concurrent**        | \$150             | \$200                 | \$50    | \$20       | \$50          | **\~\$470**              |
| **10,000 concurrent**     | \$3,000           | \$1,500               | \$500   | \$300      | \$600         | **\~\$5,900**            |
| **100,000 concurrent**    | \$30,000          | \$8,000               | \$3,000 | \$2,500    | \$5,000       | **\~\$48,500**           |
| **1 lakh monthly users**  | \$500             | \$300                 | \$50    | \$20       | \$50          | **\~\$920**              |
| **10 lakh monthly users** | \$5,000           | \$2,500               | \$800   | \$500      | \$700         | **\~\$9,500**            |
| **100 million monthly**   | \$60,000          | \$15,000              | \$5,000 | \$4,000    | \$6,000       | **\~\$90,000**           |

**Notes**:
--These are approximate AWS costs. Actual pricing depends on reserved instances, traffic, storage size, API calls, and data transfer.
