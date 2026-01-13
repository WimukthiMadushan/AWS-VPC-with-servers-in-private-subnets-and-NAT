# AWS Highly Available Web Architecture (VPC + ALB + Auto Scaling)

<img width="611" height="481" alt="Architecture Diagram" src="https://github.com/user-attachments/assets/8028bbeb-9c5a-4de6-8752-090432df5e57" />

---

## Project Overview

This project demonstrates how to design and implement a **highly available, scalable, and secure AWS architecture** using AWS best practices.

The infrastructure is deployed across **multiple Availability Zones**, with a clear separation between **public and private subnets**, and uses managed AWS services to ensure **fault tolerance, scalability, and security**.

---

## Objectives

- High availability across multiple Availability Zones
- Secure application deployment using private subnets
- Automatic scaling using Auto Scaling Groups
- Controlled outbound internet access using NAT Gateways
- Load distribution using an Application Load Balancer (ALB)

---

## Architecture Components

- Custom VPC
- Public Subnets (2 Availability Zones)
- Private Subnets (2 Availability Zones)
- Internet Gateway
- NAT Gateways (one per AZ)
- Application Load Balancer (ALB)
- Auto Scaling Group
- EC2 Instances (Private)
- Security Groups
- Route Tables

---

## Implementation Steps

### 01. Create a VPC
- Create a custom VPC names AWS-Demo-vpc (e.g. `10.0.0.0/16`)
  
A VPC (Virtual Private Cloud) is your own private network inside AWS where you can launch and control AWS resources.

<img width="1156" height="467" alt="Screenshot 2026-01-10 102227" src="https://github.com/user-attachments/assets/ea76bf07-92d7-4867-a9af-a6e92e9a2974" />
<img width="810" height="459" alt="Screenshot 2026-01-10 102235" src="https://github.com/user-attachments/assets/41ddc311-22fa-4510-aad4-2e83df14c478" />

---

### 02. Create Subnets
Create subnets across two Availability Zones. ("eu-north-1a" and "eu-north-1b")

**Public Subnets**
- Public Subnet 1 (eu-north-1a)
- Public Subnet 2 (eu-north-1b)

Private Subnets can comminicates with internet through Public subnets.

**Private Subnets**
- Private Subnet 1 (eu-north-1b)
- Private Subnet 2 (eu-north-1a)

For secure EC2 instance deployment. Cannot access From the internet through Directly.

---

### 03. Create and Attach Internet Gateway
- Create an Internet Gateway (AWS-Demo-igw)
- Attach it to the VPC

---

### 04. Create NAT Gateways
- Allocate Elastic IPs (Static IPs and don't change when resoures going down and again start)
- Create one NAT Gateway in each public subnet (AWS-Demo-nat-public2-eu-north-1a and AWS-Demo-nat-public2-eu-north-1b)

For the request origin from the EC2 instance can commiuncate with internet through the NAT GAteway. Because its secure way to comiunicate since ip address of the EC2 instance hiding From the internet.

---

### 05. Create Auto Scaling Group

An **Auto Scaling Group (ASG)** is used to automatically manage and maintain a desired number of EC2 instances. It helps ensure **high availability**, **fault tolerance**, and **scalability** by automatically launching or terminating instances based on demand or health status.

Auto Scaling Groups **cannot be created directly** without a launch configuration. In modern AWS setups, a **Launch Template** is required.


#### Step 1: Create a Launch Template

The launch template defines how EC2 instances should be created.

Configure the following:
- **AMI**: Amazon Linux
- **Instance Type**: Select based on workload
- **Key Pair**: For SSH access
- **Network Settings**: Do not assign a public IP (private subnet)
- **Security Group**:
  - SSH (22) → Bastion Security Group
  - TCP (e.g. 8000) → Application Load Balancer Security Group


#### Step 2: Create the Auto Scaling Group

- Use the created **Launch Template**
- Select **private subnets** across multiple Availability Zones
- Configure:
  - Minimum capacity
  - Desired capacity
  - Maximum capacity

The Auto Scaling Group will automatically:
- Launch new instances when demand increases
- Replace unhealthy instances
- Distribute instances across Availability Zones


#### Benefits of Auto Scaling Group

- Ensures application availability
- Automatically handles traffic spikes
- Reduces manual intervention
- Improves fault tolerance


### 06. Create Bastion EC2 Instance in Public Subnet

A **Bastion EC2 instance** (also known as a **Jump Host**) is used to securely access EC2 instances that are located inside **private subnets**.

Private EC2 instances do not have public IP addresses and cannot be accessed directly from the internet. The bastion host acts as a **secure entry point** that allows administrators to connect to private instances using SSH.


#### Purpose of Bastion EC2 Instance

- Provides **secure SSH access** to private EC2 instances
- Prevents direct internet exposure of private resources
- Centralizes administrative access to the infrastructure
- Improves security by limiting SSH access to a single, controlled instance
- Follows AWS best practices for network isolation


#### How It Works

1. The bastion host is deployed in a **public subnet** with a public IP.
2. SSH access to the bastion is restricted to the administrator’s IP address.
3. Private EC2 instances allow SSH access **only from the bastion’s security group**.
4. Administrators connect.


<img width="1049" height="733" alt="Screenshot 2026-01-10 125722" src="https://github.com/user-attachments/assets/af532b93-906a-4b87-a860-f878d81ac799" />


### 06. Create Load Balancer and Attach Target Group

An **Application Load Balancer (ALB)** is used to distribute incoming traffic across multiple EC2 instances. This improves **availability**, **fault tolerance**, and **scalability** of the application.

The ALB acts as the **single entry point** for users, while the backend EC2 instances remain in **private subnets**.

---

#### Step 1: Create Application Load Balancer

- Load Balancer Type: **Application Load Balancer**
- Scheme: **Internet-facing**
- IP address type: IPv4
- Network mapping:
  - Select the **VPC**
  - Select **public subnets** in multiple Availability Zones
- Security Group:
  - Allow HTTP (80) from `0.0.0.0/0`

The ALB receives traffic from the internet and forwards it to backend instances.


#### Step 2: Create Target Group

A **Target Group** defines where the load balancer sends traffic.

- Target type: **Instance**
- Protocol: HTTP
- Port: Application port (e.g. `8000` for Python application)
- VPC: Same VPC as EC2 instances
- Health check:
  - Protocol: HTTP
  - Path: `/`
  - Port: Traffic port

The target group continuously checks the health of registered instances.


#### Step 3: Attach Target Group to Load Balancer

- Create an ALB listener:
  - Protocol: HTTP
  - Port: 80
- Forward traffic to the created **Target Group**


#### Step 4: Attach Target Group to Auto Scaling Group

- Edit the Auto Scaling Group
- Attach the Target Group
- EC2 instances launched by Auto Scaling are automatically registered


#### Benefits of Load Balancer and Target Group

- Distributes traffic evenly across instances
- Automatically removes unhealthy instances
- Enables zero downtime scaling
- Improves application availability




