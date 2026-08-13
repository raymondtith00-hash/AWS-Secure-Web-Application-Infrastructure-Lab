# AWS-Secure-Web-Application-Infrastructure-Lab

## Overview
This project demonstrates the deployment of a segmented AWS environment using a custom VPC, public and private subnets, routing controls, security groups, an EC2 web server, and a private Amazon RDS database.

The environment is designed so that the web server can receive HTTP traffic from the Internet while the database remains isolated from direct public access. Communication between the web and database tiers is restricted using security groups and least-privilege network rules.

## Infrastructure Components

| Component | Configuration | Purpose |
|---|---|---|
| Custom VPC | `Staging-VPC` - `10.0.0.0/24` | Provides an isolated network for the staging environment |
| Public Subnet | `Staging - Public` - `10.0.0.0/25` | Hosts the public-facing EC2 web server |
| Private DB Subnet A | `10.0.0.128/26` - `us-east-1a` | Provides private network space for the database tier |
| Private DB Subnet B | `10.0.0.192/26` - `us-east-1b` | Provides a second private subnet for the RDS subnet group |
| Routing Controls | Public Route Table + Internet Gateway | Allows Internet traffic to the public subnet while keeping the database subnets private |
| Security Groups | Web-SG / DB-SG | Restricts traffic between the Internet, web server, and database |
| EC2 Web Server | Amazon EC2 | Hosts the public HTTP service |
| Private Database | Amazon RDS MySQL | Hosts the database with no direct public Internet access |

## Architecture 
The architecture separates the public-facing web tier from the private database tier. Internet traffic is routed to the EC2 web server through the public subnet, while the RDS database remains isolated within private subnets across separate Availability Zones.

![AWS Staging VPC Architecture](Screenshots/aws-staging-vpc-architecture.png)
