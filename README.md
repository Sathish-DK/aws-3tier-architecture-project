# Deploying WordPress in a Three-Tier Architecture on AWS
 #### Implementation and Best Practices for a Highly Available and Secure Environment.
---
___
## Introduction 

This project deploys a WordPress site on AWS using a three-tier architecture approach. The solution leverages AWS services such as EC2, RDS, and ELB, and includes a custom VPC created from scratch. This documentation provides an overview of the architecture and implementation steps for a highly available and secure WordPress deployment on AWS.
<!--Horizontal Rule-->
---
## Architecture Overview

* Custom VPC design and implementation with 6 subnets across two Availability Zones.
* VPC resources created including subnets, route tables, Internet Gateway, and NAT Gateway.
* Security considerations such as network ACLs and security groups.
* Web tier using EC2 instances and an Application Load Balancer in public subnets, with a NAT Gateway for outbound Internet traffic.
* Private app tier using EC2 instances with Apache web server and WordPress installed, and routing through the NAT Gateway in the web tier for Internet access.
* RDS Multi-AZ deployment for MySQL database with standby instance in a separate Availability Zone.
