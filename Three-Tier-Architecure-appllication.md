# 3-Tier Architecture Application Deployment
### ## Project Overview
```bash

project demonstrates a highly available and secure 3-Tier Web Application Architecture on AWS using separate layers:
Web Tier → Apache HTTP Server (Presentation Layer)
Application Tier → Apache Tomcat (Business Logic Layer)
Database Tier → MySQL/MariaDB (Data Layer)
```


 The architecture follows AWS production best practices:
# 📊 High Availability Design

### Features:
```bash
✅ Multiple Availability Zones

✅ Load Balancer based traffic distribution

✅ Private Application Servers

✅ Private Database Layer

✅ Security Group isolation

✅ Bastion based administration
 ```

```bash
Multi-AZ deployment
✅ Public and Private Subnet isolation
✅ Application Load Balancer
✅ Security Group based access control
✅ Bastion Host secure access
✅ Internal Load Balancer communication
✅ Reverse Proxy configuration
✅ Layer separation
 ```
# VPC Design 
1. VPC CIDR
   ```bash
   10.0.0.0/16
   ```
2.Subnet Design 
```bash
Tier	                        Subnet                         	CIDR
Public AZ-A               	Public Subnet	                    10.0.1.0/24
Public AZ-B	               Public subnet	                    10.0.2.0/24

Web AZ-A	                  Private Subnet	                   10.0.11.0/24
Web AZ-B	                  Private Subnet	                   10.0.12.0/24

App AZ-A	                  Private Subnet	                   10.0.21.0/24
App AZ-B	                  Private Subnet	                   10.0.22.0/24

DB AZ-A	                   Private Subnet	                   10.0.31.0/24
DB AZ-B                   	Private Subnet	                   10.0.32.0/24

```

# Security Group Architecture

Bastion Host SG

```bash
Inbound:

 SSH 22
Source: My IP

Outbound:

All Traffic
```
Note : 
Purpose:

Secure administration access
No direct SSH access to private servers


Public ALB Security Group : 

```bash
 Inbound:

HTTP 80
Source: 0.0.0.0/0

HTTPS 443
Source: 0.0.0.0/0

Outbound:

HTTP 80
Destination: Web SG
```
Web Server Security Group
```bash
Inbound:

HTTP 80
Source: Public ALB SG

SSH 22
Source: Bastion SG

Outbound:
HTTP 80
Destination: Internal ALB SG
```
Internal ALB Security Group
```bash
Inbound:

HTTP 80
Source: Web SG

Outbound:

8080
Destination: App SG
```
Application Server Security Group
```bash
Inbound:

Tomcat 8080
Source: Internal ALB SG

SSH 22
Source: Bastion SG

Outbound:

MySQL 3306
Destination: DB SG
```
Database Security Group
```bash
Inbound:

MYSQL 3306
Source: App SG

Optional:

SSH 22
Source: Bastion SG

```

# ⚖️ Load Balancer Configuration
Public ALB

Configuration:
```bash

Type:
Application Load Balancer

Scheme:
Internet-facing
Listener:

HTTP: 80

Target Group:

web-tg


Protocol:

HTTP


Port:

80

Health Check:

Path: /

Matcher:

200

Internal ALB

Configuration:
```bash
Scheme:

Internal

Listener:

HTTP 80

Target Group:

app-tg

Protocol:

HTTP

Port:

8080

Health Check:

Path:

/

Matcher:
200
```
