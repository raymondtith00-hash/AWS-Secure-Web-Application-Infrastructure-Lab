# AWS-Secure-Web-Application-Infrastructure-Lab

## Overview
This lab demonstrates how to build a secure AWS environment with a public EC2 web server and a private RDS database. The project uses VPCs, subnets, route tables, an Internet Gateway, and security groups to separate public-facing resources from internal database resources and restrict access using least-privilege network rules. 

The environment is designed so that the web server can receive HTTP traffic from the Internet while the database remains isolated from direct public access. Communication between the web and database tiers is restricted using security groups and least-privilege network rules.

## Architecture 

![AWS Staging VPC Architecture](Screenshots/aws-staging-vpc-architecture.png)
