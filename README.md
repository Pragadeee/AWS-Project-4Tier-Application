# AWS-Project-4Tier-Application
This repository provides an end-to-end deployment guide for a PHP mailing application on AWS, designed using a secure, multi-tier architecture. The setup includes a VPC with isolated subnets for web, app, and database layers, leveraging Amazon RDS for MySQL, EC2 instances, and S3 for configuration management.


Implementing a PHP Mailing deployment with a multi-tier architecture on AWS involves creating an isolated network environment that connects frontend and backend servers to a database, ensuring secure and efficient data flow. Here’s a more detailed look at each step of the setup:

STEP 1: Create the Base Networking Infrastructure
A) Create the VPC Network
Name: Prod-VPC
CIDR Block: 10.0.0.0/16
The VPC (Virtual Private Cloud) isolates the entire application environment. The /16 CIDR block provides ample IP addresses for subnets and allows segmentation across different tiers of the architecture.

B) Subnet Creation
Each layer of the application requires dedicated subnets, ensuring each component is isolated yet connected within the VPC.

NAT/ALB Subnets: For load balancing and NAT gateway setup.

Prod-NAT-ALB-Subnet-1: 10.0.5.0/28 in us-west-1a
Prod-NAT-ALB-Subnet-2: 10.0.10.0/28 in us-west-1c
Webserver Subnets: For hosting frontend web servers.

Prod-Webserver-Subnet-1: 10.0.15.0/24 in us-west-1a
Prod-Webserver-Subnet-2: 10.0.20.0/24 in us-west-1c
Appserver Subnets: For application logic and backend services.

Prod-Appserver-Subnet-1: 10.0.25.0/24 in us-west-1a
Prod-Appserver-Subnet-2: 10.0.30.0/24 in us-west-1c
Database Subnets: For MySQL RDS, isolated to enhance security.

Prod-db-Subnet-1: 10.0.35.0/24 in us-west-1a
Prod-db-Subnet-2: 10.0.40.0/24 in us-west-1c



STEP 2: Route Table Creation and Association
For proper routing and failover:

Public Route Tables: Assigned to NAT and Webserver subnets for internet access.
Private Route Tables: Assigned to Appserver and Database subnets, routed via NAT gateways for external traffic.


STEP 3: Associate Route Tables with Subnets
Each subnet’s route table defines traffic flow rules. The NAT/ALB subnets, Webserver subnets, Appserver subnets, and Database subnets each get a dedicated route table for optimal control.

STEP 4: Configure IGW and NAT Gateways
Internet Gateway: Attached to Prod-VPC, allowing the NAT and Webserver subnets to access the internet.

Route Update: For the NAT/ALB and Webserver route tables to direct traffic to the IGW.
NAT Gateways: Enable outbound internet traffic from the private Appserver and Database subnets.

Prod-NAT-Gateway-1: In Prod-NAT-ALB-Subnet-1, associated with Appserver and Database Route Table 1.
Prod-NAT-Gateway-2: In Prod-NAT-ALB-Subnet-2, associated with Appserver and Database Route Table 2.


STEP 5: Create Security Groups
Each layer has specific security groups, defining allowed traffic per port and source.

Bastion Host Security Group: SSH access on port 22 for secure administrative access.
Frontend Load Balancer Security Group: Public HTTP/HTTPS access on ports 80 and 443.
Webservers Security Group: Allows connections from the frontend load balancer and the bastion host.
Backend Load Balancer Security Group: Manages access to backend services (Appservers).
Appservers Security Group: Accepts traffic from backend load balancer and allows SSH access via the bastion host.
Database Security Group: Limits access to the Appservers for MySQL traffic on port 3306, with optional access for the bastion host.


STEP 6: Create Load Balancers
Load balancers distribute traffic across web and application servers, ensuring high availability.

Frontend Load Balancer: Publicly accessible, directing requests to web servers.

Target Group: HTTP target group (Frontend-LB-HTTP-TG), using /VenturaMailingApp.php for health checks.
Security Group: Frontend-LB-Security-Group
Backend Load Balancer: Internal, directing traffic to app servers.

Target Group: HTTP target group (Backend-LB-HTTP-TG), using /VenturaMailingApp.php for health checks.
Security Group: Backend-LB-Security-Group

STEP 7: Database Setup
Database Subnet Group: Designated for RDS with multi-AZ redundancy.
Subnet Group: prod-db-subnet-group
Subnets: Prod-db-Subnet-1 and Prod-db-Subnet-2
MySQL RDS Instance: Deployed as a multi-AZ, password-authenticated instance in the isolated database subnets, configured for automated backups and encryption.

STEP 8: S3 Bucket Setup
An S3 bucket stores automation scripts and configuration files for easy retrieval by EC2 instances.

Bucket Configuration: Name the bucket following prod-proxy-app-db-config-YOUR-LAST-NAME-MONTH-OF-BIRTH.

Enable versioning and encryption, ensuring secure, organized storage.
Content: Upload configuration files after modifying with your specific settings.

STEP 9: Bastion Host Configuration
A bastion host allows secure access to private subnets.

Create EC2 Instance: Prod-Bastion-Host, using Ubuntu 20.04 and a public IP for SSH access.
IAM Role: Assign AmazonS3ReadOnlyAccess to allow web and app servers to access configuration files on S3.

STEP 10: SSH Agent Forwarding
Use SSH agent forwarding to access private resources through the bastion host:

SSH Commands:
Start an SSH agent session and add the key to SSH agent.
SSH from bastion to app servers using agent forwarding.
