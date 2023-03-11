# __Deploying WordPress in a Three-Tier Architecture on AWS__
 #### Implementation and Best Practices for a Highly Available and Secure Environment.
---
## __Introduction__ 

This project deploys a WordPress site on AWS using a three-tier architecture approach. The solution leverages AWS services such as EC2, RDS, and ELB, and includes a custom VPC created from scratch. This documentation provides an overview of the architecture and implementation steps for a highly available and secure WordPress deployment on AWS.

## __Architecture Overview__
![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/My_architecture_overview.png)


* Custom VPC design and implementation with 6 subnets across two Availability Zones.
* VPC resources created including subnets, route tables, Internet Gateway, and NAT Gateway.
* Security considerations such as network ACLs and security groups.
* Web tier using EC2 instances and an Application Load Balancer in public subnets, with a NAT Gateway for outbound Internet traffic.
* Private app tier using EC2 instances with Apache web server and WordPress installed, and routing through the NAT Gateway in the web tier for Internet access.
* RDS Multi-AZ deployment for MySQL database with standby instance in a separate Availability Zone.

## __Implementation__
### 1. VPC and Subnets
* Created a custom VPC with a preferred CIDR block using the AWS Management Console.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/vpc-cidr-create.png)

* Created six subnets across two Availability Zones, with three subnets in each zone, using the AWS Management Console. Each subnet corresponds to one layer of the three-tier architecture.
* Configured two public subnets in the web tier, two private subnets in the app tier, and two private subnets in the database tier. Each subnet was named with a naming convention that reflects its purpose and availability zone, such as Public-Web-Subnet-AZ-1, Private-App-Subnet-AZ-1, and Private-DB-Subnet-AZ-1. The CIDR range for each subnet was defined as a subset of the VPC CIDR range.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/subnets-6-2azs.png)

* Created an Internet Gateway and attached it to the VPC to enable outbound traffic from the public subnets to the Internet.

### 2. Internet Connectivity
* Created and attached an Internet Gateway to the custom VPC in order to provide internet access to the public subnets in the web tier.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/creating-igw.png)

* A NAT Gateway was created and in the public subnet of the web tier to allow the private app and database tier servers to communicate with the internet for updates, patches, and installations.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/creating-NAT-GW.png)

### 3. Routing Configurations
* Three route tables were created to manage traffic within the VPC.
    * The first route table was assigned to the two public subnets of the web tier layer.
    * The second route table was assigned to the two private subnets for the app tier layer.
    * The third route table was assigned to the two private subnets for the database tier layer.

Each route table was associated with its respective subnets, with a screenshot below showing only one example.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/subnet-association-attach.png)

* For internet access to the public subnet of web tier layer, a route was added in the public route table with the destination CIDR block as 0.0.0.0/0, and target as Internet Gateway.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/routes-edited-public-igw.png)

* For internet access to the private subnets of app and database tier layers, a route was added in the respective private route tables with the destination CIDR block as 0.0.0.0/0, and target as the NAT Gateway

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/route-nat-private-routtable-app.png)



