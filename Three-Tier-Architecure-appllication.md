# 3-Tier Architecture Application Deployment
### ## Project Overview
```bash

project demonstrates a highly available and secure 3-Tier Web Application Architecture on AWS using separate layers:
Web Tier → Apache HTTP Server (Presentation Layer)
Application Tier → Apache Tomcat (Business Logic Layer)
Database Tier → MySQL/MariaDB (Data Layer)
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
