# 🌐 Site-to-Site VPN Between Azure and AWS



## Step 1: Configuring Azure 
1.Create a resource group on Azure to deploy the resources on that
 1. Crate a resource group on Azure to deploy the resources on that
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
Name      : aws-vpc

CIDR      : 10.10.0.0/16

6.Create Public/Private Subnet
Subnet Name

private-subnet-01

CIDR

10.10.1.0/24

7.Create Customer Gateway --> Create a customer gateway pointing to the Public IP Address of Azure VPN Gateway
 IP address: Public IP Address of Azure VPN Gateway
 Customer Gateway

IP Address

Azure VPN Gateway Public IP

Routing

Static

8.Create Virtual Private Gateway --> Create the Virtual Private Gateway then attach to the VPC
Name

vgw-azure

ASN

Default  or 64512 you can use



