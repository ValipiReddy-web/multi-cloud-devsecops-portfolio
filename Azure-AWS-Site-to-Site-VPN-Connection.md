# 🌐 Site-to-Site VPN Between Azure and AWS

## Step 1: Configuring Azure 
1.Create a resource group on Azure to deploy the resources on that
  
 ```bash
 Resource Group Name: rg-azure-aws
 Region: East-US
 ```

2. Create Virtual Network
 ```bash
 VNet Name        : vnet-azure
Address Space    : 172.10.0.0/16

Subnet
-------
Name             : subnet-01
CIDR             : 172.10.1.0/24

Gateway Subnet
--------------
Name             : GatewaySubnet
CIDR             : 172.10.255.0/27
 ```
Note
GatewaySubnet is mandatory for Azure VPN Gateway.

3. Create the VPN Gateway
 ```bash
 Name                : vpn-azure-aws

Gateway Type        : VPN

VPN Type            : Route-based

SKU                 : VpnGw1

Generation          : Generation2

Public IP SKU       : Standard

Public IP           : Static

Active-Active       : Disabled

BGP                 : Disabled (Static Routing Demo)
 ```

Wait approximately 30–45 minutes for deployment. 
 

##AWS Configuration

5.Create VPC
```bash
Name      : aws-vpc

CIDR      : 10.10.0.0/16
```
6.Create Public/Private Subnet
```bash
Subnet Name

private-subnet-01

CIDR

10.10.1.0/24
```
7.Create Customer Gateway --> Create a customer gateway pointing to the Public IP Address of Azure VPN Gateway
 ```bash
 IP address: Public IP Address of Azure VPN Gateway
 Customer Gateway

IP Address

Azure VPN Gateway Public IP

Routing

Static
```
8.Create Virtual Private Gateway --> Create the Virtual Private Gateway then attach to the VPC
```bash
Name

vgw-azure

ASN

Default  or 64512 you can use

IMP Note : Virtual Private Gateway then attach to the VPC
```
9.Create Site-to-Site VPN Connection 
```bash
Target gateway type: Virtual private gateway (Select your Virtual private gateway created in 8)
 Customer gateway: Existing (Select your VCustomer gateway created in 7)
 Routing options: Static
 Static IP prefixes: 172.10.1.0/24 or 172.10.0.0/16
 ```
Note : AWS creates

Tunnel 1
Tunnel 2 (High Availability)

10. Download the VPN configuration file
    Download using
    ```bash
    Vendor

Generic

Platform

Generic

Software

Vendor Agnostic
```
The configuration contains

Tunnel Outside IPs
Pre-Shared Keys (PSK)
IPSec Parameters
IKE Parameters

 ## Connecting Azure and AWS
11. Create the Local Network Gateway in Azure
```bash
 ```bash
 Name: lng-azure-aws
 Resource Group Name: rg-azure-aws
 Region: East-US
 IP address: Get the Outside IP address from the configuration file downloaded in 10.
 Address Space(s): 10.10.0.0/16
```

12. Create the connection on the Virtual Network Gateway in Azure
 ```bash
 Name: connection-azure-aws
 Connection Type: Site-to-Site
 Local Network Gateway: Select the Local Network Gateway which you created in 11.
 Shared Key: Get the Shared Key from the configuration file downloaded in 10.
 Wait till the Connection Status changes to - Connected
 In the same way, check in AWS Console wheather the 1st tunnel of Virtual Private Gateway UP.
 ```
 13. Create Internet Gateway and Attach it to VPC in AWS:
 ```bash
 Name: my-internet-gateway
 ```
13. Now let's edit the route table associated with our VPC
 ```bash
 Add the route to Azure subnet through the Virtual Private Gateway
 Destination: 172.10.1.0/24
 Target: Virtual Private Gateway that we created.
 also add,
 Destination: 0.0.0.0/0
 Target: Internet Gateway that we created in 13.

Destination          Target

10.10.0.0/16         Local

172.10.0.0/16        Virtual Private Gateway

0.0.0.0/0            Internet Gateway

 ```
15. Create VMs in both Azure and AWS and Test the connection.

16. Security
```bash
Azure NSG

Allow

ICMP

SSH (22)

RDP (3389)

Application Ports

AWS Security Group

Allow

ICMP

SSH

RDP

Application Ports
```
17. Connectivity Test
```bash
From Azure VM

ping 10.10.1.10  

and
From AWS EC2
ping 172.10.1.4
```
 
