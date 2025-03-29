#  Virtual Private Cloud (VPC) Setup with NAT Gateway, Security Groups, and NACLs

## Documentation

###Objective
This document outlines the steps to:
1. Create a VPC with public and private subnets across multiple Availability Zones (AZs).
2. Set up and test a NAT Gateway for private subnet instances.
3. Implement Security Groups and Network Access Control Lists (NACLs) for traffic control.

---

## Step 1: Create a VPC
A VPC is an isolated network within AWS where resources can be deployed.

### Steps:
1. Navigate to the AWS VPC Dashboard
   - Go to the AWS Management Console → VPC → "Your VPCs" → "Create VPC."
2.Configure VPC Settings**  
   - Name tag: `MyVPC`  
   - IPv4 CIDR block: `12.0.0.0/16` (provides 65,536 private IPs)  
   - Leave IPv6 and Tenancy as default.  
3. Click "Create VPC."

---

## Step 2: Create Subnets in Different Availability Zones
Subnets segment the VPC into smaller networks. We will create:
- Public Subnets: For resources that need direct internet access .  
- Private Subnets: 
### Steps:
1. Go to "Subnets" → "Create subnet."  
2. Select the VPC (`MyVPC`). 
3. Create Public Subnet 1: 
   - Subnet name: `M4ACES ERVER`  
   - Availability Zone: `us-east-1b`  
   - IPv4 CIDR: `12.0.0.0/24`  
4. Create Private Subnet 2:  
   - Subnet name: `M4ACE server private`  
   - Availability Zone: `us-east-1b`  
   - IPv4 CIDR:`12.0.1.0/24`  
7. Click "Create."
---
## Step 3: Set Up an Internet Gateway (IGW)
An IGW allows public subnets to communicate with the internet.
### Steps:
1. Go to "Internet Gateways" → "Create internet gateway."
2. Name:`MyIGW` → Click "Create."
3. Attach it to the VPC (`MyVPC`).
---
## Step 4: Configure Route Tables
Route tables determine where network traffic is directed.
### Public Route Table (for Public Subnets)
1. Go to "Route Tables" → "Create route table." 
   - Name: `PublicRouteTable`  
   - VPC: `MyVPC`  
2. Edit routes:
   - Add a route:  
     - Destination: `0.0.0.0/0` (all traffic)  
     - Target: `MyIGW`  
3. Associate Public Subnets (`M4ACE Server`, `).
### Private Route Table (for Private Subnets)
1. Create a new route table:
   - Name: `M4ACE RT`  
   - VPC: `MyVPC`  
2. No internet route is added here (keeps subnets private).
3. Associate Private Subnets (``M4ACE Private server `). 
---
## Step 5: Set Up a NAT Gateway
A NAT Gateway allows private subnets to access the internet (for updates, patches) without exposing them.
### Steps:
1. Go to "NAT Gateways" → "Create NAT Gateway."
   - Name: `M4ACE NGW`  
   - Subnet: `M4ACE public server` (must be in a public subnet)  
   -Allocate Elastic IP (EIP).
2. Click "Create."
3. Update Private Route Table:
   - Edit `PrivateRouteTable` → Add route:  
     - Destination: `0.0.0.0/0`  
     - Target: `MyNATGateway`  

---

## Step 6: Test NAT Gateway
1. Launch an EC2 instance in `PrivateSubnet-AZ1`.
   - Ensure no public IP is assigned.  
   - If it returns an IP (the NAT Gateway's EIP), the NAT Gateway is working.  

---
## Step 7: Implement Security Groups (SGs)
SGs act as virtual firewalls for EC2 instances (stateful—return traffic is automatically allowed).
### Steps:
1. Go to "Security Groups" → "Create security group." 
   - Name: `WebServerSG`  
   - **Description:** Allow HTTP/HTTPS/SSH  
   - VPC: `MyVPC`  
2. Inbound Rules:  
   - SSH (22): From your IP or Bastion Host.  
   - HTTP (80): From anywhere (`0.0.0.0/0`).  
   - HTTPS (443): From anywhere (`0.0.0.0/0`).  
3. Outbound Rules: 
   - Allow all outbound traffic (default).  
---
## Step 8: Configure Network ACLs (NACLs)
NACLs are stateless firewall rules at the subnet level.
### Steps:
1. Go to "Network ACLs" → "Create network ACL." 
   - Name: `MyNACL`  
   - VPC: `MyVPC`  
2. Associate with `PrivateSubnet-AZ1` and `PrivateSubnet-AZ2`.  
3. Inbound Rules: 
   - Rule 100: Allow SSH (22) from Bastion Host’s IP.  
   - Rule 200: Allow Ephemeral Ports (1024-65535) for responses.  
4. Outbound Rules:
   - Rule 100: Allow HTTP (80) to anywhere.  
   - Rule 200: Allow HTTPS (443) to anywhere.  
   - Rule 300: Allow Ephemeral Ports for responses.  

---
