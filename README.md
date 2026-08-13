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

--- 

## Implementation

### 1. VPC and Subnet Configuration

Created a custom VPC named `Staging-VPC` using the CIDR block `10.0.0.0/24`.

The VPC was segmented into one public subnet for the web tier and two private subnets for the database tier.

| Subnet | CIDR | Availability Zone | Purpose |
|---|---|---|---|
| `Staging - Public` | `10.0.0.0/25` | `us-east-1f` | Hosts the public EC2 web server |
| `Staging - Private - DB - A` | `10.0.0.128/26` | `us-east-1a` | Private RDS subnet |
| `Staging - Private - DB - B` | `10.0.0.192/26` | `us-east-1b` | Secondary private RDS subnet |

The two database subnets were placed in separate Availability Zones and later used together in the RDS DB subnet group.

![Subnet Layout and Availability Zones](Screenshots/subnet-layout-and-availability-zones.png)

---

### 2. Internet Gateway and Routing

Created an Internet Gateway and attached it to `Staging-VPC`.

A staging route table was configured with a default route to the Internet Gateway:

```text
10.0.0.0/24  -> local
0.0.0.0/0    -> Internet Gateway

---

### 3. Web Server Security Group

Created `Staging - Web - SG` to control inbound access to the EC2 web server.

| Type | Protocol | Port | Source |
|---|---|---:|---|
| HTTP | TCP | 80 | `0.0.0.0/0` |
| SSH | TCP | 22 | Trusted administrator IP |

HTTP was allowed publicly so the web server could be reached from the Internet, while SSH access was restricted to a trusted source.

![Web Security Group](Screenshots/web-security-group.png)

---

### 4. EC2 Web Server Deployment

Deployed an Amazon Linux 2023 EC2 instance named `Staging-Web-Server` inside the public subnet.

The instance was configured with:

- `Staging - Public` subnet
- `Staging - Web - SG`
- Public IPv4 address
- `t3.micro` instance type

![EC2 Web Server Running](Screenshots/ec2-web-server-running.png)

Apache HTTP Server was installed and enabled on the instance.

![Apache HTTPD Running](Screenshots/apache-httpd-running.png)

The web server was then tested externally over HTTP.

![HTTP Web Server Test](Screenshots/http-web-server-test.png)

---

### 5. Database Security Group

Created `Staging - DB _ SG` to restrict access to the private database tier.

| Type | Protocol | Port | Source |
|---|---|---:|---|
| MySQL/Aurora | TCP | 3306 | `Staging - Web - SG` |

Instead of allowing MySQL access from the Internet, the rule references the web server security group. This allows database traffic only from approved resources in the web tier.

![Database Security Group](Screenshots/database-security-group.png)

---

### 6. RDS DB Subnet Group

Created `staging-db-subnet-group` using both private database subnets.

| Subnet | CIDR | Availability Zone |
|---|---|---|
| `Staging - Private - DB - A` | `10.0.0.128/26` | `us-east-1a` |
| `Staging - Private - DB - B` | `10.0.0.192/26` | `us-east-1b` |

The public subnet was intentionally excluded from the database subnet group.

![RDS DB Subnet Group](Screenshots/rds-db-subnet-group.png)

---

### 7. Private RDS MySQL Deployment

Deployed an Amazon RDS MySQL database named `staging-mysql-db`.

| Setting | Configuration |
|---|---|
| Engine | MySQL Community 8.4.10 |
| Instance Class | `db.t4g.micro` |
| Storage | 20 GiB General Purpose SSD |
| Encryption | Enabled |
| Public Access | Disabled |
| Multi-AZ | No |
| DB Subnet Group | `staging-db-subnet-group` |
| Security Group | `Staging - DB _ SG` |
| Port | `3306` |

The database was successfully provisioned and reached the `Available` state.

![RDS Database Available](Screenshots/rds-database-available.png)

---

### 8. EC2-to-RDS Validation

Connected from `Staging-Web-Server` to the private RDS MySQL instance over TCP port `3306`.

![EC2 to RDS Connection](Screenshots/ec2-rds-connection-test.png)

A test database and table were created to confirm successful read and write access:

```sql
CREATE DATABASE staging_app;

USE staging_app;

CREATE TABLE connection_test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    message VARCHAR(100)
);

INSERT INTO connection_test (message)
VALUES ('EC2 successfully connected to private RDS');

SELECT * FROM connection_test;
