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

## __Networking and Security__
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

### 4. Security Groups
In order to control inbound traffic to the instances, I configured four security groups:

1. ALB security group: This security group allows HTTP/S traffic from anywhere.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/SG1-EXT-ALB-2.png)

1. Web tier instance security group: This security group allows SSH traffic from anywhere.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/SG2-web-2(1).png)

1. App tier instance security group: This security group allows SSH traffic only from the security group of the web tier instance and HTTP traffic only from the ALB security group.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/SG3-APP-TIER(1).png)

1. Database instance security group: This security group only allows traffic from the app tier security group.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/vpc/SG4-DB-2.png)

## Server Configurations

Server Configuration for Three Tier Architecture:
### 1. Web Tier:
The Web Tier consists of two jump servers (web servers) located in different availability zones (AZs) with my-two public subnets. These servers are used to access the private App Tier instances to install Apache app server and WordPress. These servers should be highly secured.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/ec2-web-tier-3.png)

I used the web tier instances as jump servers to SSH into the app tier instances from my local machine. This allowed me to securely access and configure the app tier instances without exposing them to the public internet.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/Screenshot%20from%202023-03-09%2014-39-14.png)

### 2. App Tier:
We have configured two app server instances in different AZs within private subnets. These instances hold Apache server and WordPress, which we installed using the help of NAT gateway and public jump servers (web tier instances). The security group has been configured to allow traffic only from the ELB security group. Our application is located in private subnets for security reasons, so it is not exposed to the internet.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/ec2-app-tier-az1-3.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/ec2-app-tier-az2-1.png)

### 3. Database Tier:
The Database Tier consists of a multi-AZ standby Amazon RDS running with MySQL engine. We have deployed the database in two AZs for high availability, and it is securely storing our WordPress data in private subnets. The database is only accessible from the App Tier security group, and it does not allow direct public access.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-subnet-group-creation-1.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-subnet-group-creation-2.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-CREATE-1.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-CREATE-3.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-CREATE-4.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-CREATE-5.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-CREATE-6.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/DB-CREATE-7.png)

## ELB and Wordpress Configurations
### Elastic Load Balancer (ELB)
Configured an Application Load Balancer (ALB) in two public subnets that routes traffic into two AZs app server instances with the ALB DNS name that routes traffic to both App Tier instances. The ALB security group allows HTTP/S traffic from anywhere.

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/load%20balencer%20summary.png)

### Configuring Wordpress on EC2
To set up the app tier, install the WordPress application and its dependencies on the EC2 instance. on two app server instances in different availability zones using Amazon Linux 2. We accessed these instances using SSH within the jump servers.
#### Installing the apache web server:
```bash
sudo yum update -y
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
find /var/www -type f -exec sudo chmod 0664 {} \;
```
#### Download and Configure Wordpress:
```bash
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cd wordpress
cp wp-config-sample.php wp-config.php
nano wp-config.php
```
Edit two areas of configuration

* First, edit the database configuration by changing the following lines:
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'database_name_here' );

/** MySQL database username */
define( 'DB_USER', 'username_here' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password_here' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```
* The second configuration section need to configure is the Authentication Unique Keys and Salts. It looks as follows in the configuration file:

```
/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
```
[This link](https://api.wordpress.org/secret-key/1.1/salt/) to generate values for this configuration section. Can replace the entire content in that section with the content from the link

Finally, the WordPress website was accessed using the ALB DNS name, which routes traffic to both app tier instances in different availability zones, ensuring high availability. Here is a screenshot of the WordPress website that was accessed successfully:

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/Wordpress-login-page.png)

![Markdown Logo](https://github.com/Sathish-DK/aws-cloudpress-3tier-architecture-project/blob/main/architectures_and_Scrnshots/Three-tier-config/worpress.png)

## Troubleshooting
* __Issue__: Could not establish database connection with WordPress.
* __Solution Attempted__: Checked the wp-config.php file, but everything seemed fine.
* __Solution__: Discovered with the help of ChatGPT that in the database section, I had mistakenly typed the database identifier instead of the database name. After correcting the error, the database connection was successfully established with WordPress.

## Conclusion
After successfully deploying a highly available WordPress website on AWS, the project can be concluded as a success. The website has been deployed in two different availability zones, ensuring high availability and fault tolerance.

The project involved setting up and configuring different AWS services, such as Amazon EC2, Amazon RDS, Amazon VPC, and Amazon ALB. Additionally, the Apache web server, MySQL database, and WordPress were installed and configured on the EC2 instances.

During the project, several challenges were encountered, such as configuring the VPC, setting up the subnets, and troubleshooting issues with the WordPress database connectivity. However, these challenges were overcome with the help of AWS documentation, forums, and ChatGPT.

Overall, the project provided a great learning experience and valuable knowledge and skills in deploying and managing highly available web applications on AWS. This documentation can serve as a helpful guide for anyone who wishes to deploy their WordPress website on AWS and gain a good understanding of the different AWS services and configurations required for the same.
___
---
