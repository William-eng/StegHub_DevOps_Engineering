# AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY

## General Overview
You will create a secure infrastructure within AWS VPC (Virtual Private Cloud) for a fictional company named KTrontech. This infrastructure will support Ktrontech's main business website, 
which uses the WordPress Content Management System (CMS), as well as a Tooling Website (PHP/MySQL) for their DevOps team. To enhance security and performance, the company has decided to 
implement **NGINX reverse proxy technology**. The project prioritizes cost-efficiency, security, and scalability. The goal is to develop an architecture that ensures both the WordPress and 
Tooling websites are resilient to web server failures, can handle increased traffic, and maintain reasonable costs.

- ![image-96-1024x955](https://github.com/user-attachments/assets/9c4f4747-0162-431b-8f0d-6f64b9601fe0)
## - Requirements
- There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit
Create an AWS Master account. (Also known as Root Account)
Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

- ![devOU](https://github.com/user-attachments/assets/c6fa7c2e-305b-4ed7-b017-a672d553887f)
- ![move](https://github.com/user-attachments/assets/593e5177-5d67-42a6-a4d7-468c5af412e2)

- Login to the newly created AWS account using the new email address.
2. Create a domain name for your company. I used Hostinger.
3. Create a hosted zone in AWS, and map it to your domain. to do this ,
 - On your AWS console search bar, type route 53 and click on it, > select create hosted zones
- ![hosted-zone](https://github.com/user-attachments/assets/e368c2ad-2d89-4323-9d8f-da5c3d0a32bd)

- Set your already purchased domain name, and set the type o public hosted zone, click on create hosted zone
- copy the namservers and go back to your domain management , and add the nameservers, allow it sometime to propagate, usually between few hours to a day

## SET UP A VIRTUAL PRIVATE CLOUD (VPC)
1. Create a VPC
   - ![vpc](https://github.com/user-attachments/assets/001ab052-32e0-41c6-8eca-386d22a5546f)

2. Create the subnets as shown in the Architecture
   
- ![subnet](https://github.com/user-attachments/assets/7e3c1434-99ae-48ae-a880-075ddb5a23a0)

3. Create a route table and associate it with public subnets. to do this ,
- on the side vpc dashboard side medu, click on route tables > create route table
- Associate Public subnet to the Public route table. Click the tab Actions > edit subnet associations - select the subnets and click save associations
- ![routeTable](https://github.com/user-attachments/assets/d6fefc67-58f5-4d1a-bd4b-829c6d3915dd)
  
- ![associatesubnet](https://github.com/user-attachments/assets/a6203b97-06cc-41d3-8b6a-3a85728b6a01)

4. Create a route table and associate it with private subnets. to do this
Repeat same process as step 3 above
 - ![route](https://github.com/user-attachments/assets/513cd3c9-9a0c-4d39-abab-7fe74661dfd9)
   
5. Create internet gateway as shown in the architecture.and attach it to the VPC
- ![Internet-Gateway](https://github.com/user-attachments/assets/d3579958-6c17-43fc-af4f-84efe143fd74)

Next, attach the internet gateway to the VPC we created earlier. On the same page click Attach to VPC, select the vpc and click attach internet gateway

- ![attach-Igw](https://github.com/user-attachments/assets/778747b5-6c17-479a-ab01-3f00fb415000)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet).To achieve this,
Select the public route table we created, the click on actions > edit route

- ![Screenshot from 2024-10-26 09-27-34](https://github.com/user-attachments/assets/afdb8a54-546b-4eff-ad18-6803b2caa285)

7. Create Elastic IP to configured with the NAT gateway. The NAT gateway enables connection from the public subnet to private subnet and it needs a static ip to make this happen.VPC > Elastic IP addresses > Allocate Elastic IP address - add a name tag and click on allocate
- ![elastic-ip](https://github.com/user-attachments/assets/771226fc-a4a3-49da-b5ee-443993dd6f77)

8. Create a Nat Gateway and assign the Elastic IPs Click on VPC > NAT gateways > Create NAT gateway

_Select a Public Subnet
Connection Type: Public
Allocate Elastic IP_

9. Update the Private route table - add allow anywhere ip and associate it the NAT gateway.
- ![NAT-Gateway](https://github.com/user-attachments/assets/d4ed9c84-a830-4297-a9db-0dd4afc994b6)

10. Create a Security Group for the following

- _Nginx Servers_: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
- _Bastion Servers_: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
- ![Screenshot from 2024-10-26 10-34-23](https://github.com/user-attachments/assets/0fc3612b-6d72-458f-bb5c-491bb0e6d5e0)

- _Application Load Balancer_: ALB will be available from the Internet
- _Webservers_: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
- _Data Layer_: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

- ![security-group](https://github.com/user-attachments/assets/81ec5917-586d-4996-9bde-061569bef484)

## TLS Certificates From Amazon Certificate Manager (ACM)
You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB). to do this, navigate to amazon certificate manager (acm) > request certificate and fill in rquired details
- ![certificatemanager](https://github.com/user-attachments/assets/6591c892-79b6-47aa-903f-c00fec6b1e01)
- ![certissued](https://github.com/user-attachments/assets/000c480b-8eaf-40f5-8c37-b79dc6c6a21d)

## Configure EFS
- Create a new EFS File system. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer. Associate the Security groups created earlier for data layer.
- ![EFScreate](https://github.com/user-attachments/assets/2b18ce3c-45d0-4a91-8fec-c51aa056198e)
- ![efs create](https://github.com/user-attachments/assets/cb462b6a-83e0-4c30-8c98-e955e0332216)

- Create 2 access points - For each of the website (wordpress and tooling) so that the files do not overwrite each other when we mount
- ![Screenshot from 2024-10-26 12-18-26](https://github.com/user-attachments/assets/d0cab7ca-41ab-4b4d-bb00-2e0d9d566334)
- ![Screenshot from 2024-10-26 12-18-32](https://github.com/user-attachments/assets/2136a135-da9e-41cc-a0ec-5455faee880c)

- ![aceespoints](https://github.com/user-attachments/assets/85150449-83b1-4913-9b90-9de887a89556)


## Configure RDS
- Pre-requisite:
- Create a KMS Key

- ![KeyManagementService](https://github.com/user-attachments/assets/c479963d-35b5-46f3-b094-43fdfe73cba9)
leave all as default and add the key policy 

       {
           "Id": "key-consolepolicy-3",
           "Version": "2012-10-17",
           "Statement": [
               {
                   "Sid": "Enable IAM User Permissions",
                   "Effect": "Allow",
                   "Principal": {
                       "AWS": "arn:aws:iam::905418098244:root"
                   },
                   "Action": "kms:*",
                   "Resource": "*"
               },
               {
                   "Sid": "Allow access for Key Administrators",
                   "Effect": "Allow",
                   "Principal": {
                       "AWS": "arn:aws:iam::905418098244:user/<ur user>"
                   },
                   "Action": [
                       "kms:Create*",
                       "kms:Describe*",
                       "kms:Enable*",
                       "kms:List*",
                       "kms:Put*",
                       "kms:Update*",
                       "kms:Revoke*",
                       "kms:Disable*",
                       "kms:Get*",
                       "kms:Delete*",
                       "kms:TagResource",
                       "kms:UntagResource",
                       "kms:ScheduleKeyDeletion",
                       "kms:CancelKeyDeletion",
                       "kms:RotateKeyOnDemand"
                   ],
                   "Resource": "*"
               },
               {
                   "Sid": "Allow use of the key",
                   "Effect": "Allow",
                   "Principal": {
                       "AWS": "arn:aws:iam::905418098244:user/Terah"
                   },
                   "Action": [
                       "kms:Encrypt",
                       "kms:Decrypt",
                       "kms:ReEncrypt*",
                       "kms:GenerateDataKey*",
                       "kms:DescribeKey"
                   ],
                   "Resource": "*"
               },
               {
                   "Sid": "Allow attachment of persistent resources",
                   "Effect": "Allow",
                   "Principal": {
                       "AWS": "arn:aws:iam::905418098244:user/Terah"
                   },
                   "Action": [
                       "kms:CreateGrant",
                       "kms:ListGrants",
                       "kms:RevokeGrant"
                   ],
                   "Resource": "*",
                   "Condition": {
                       "Bool": {
                           "kms:GrantIsForAWSResource": "true"
                       }
                   }
               }
           ]
       }

- ![managementkeycreated](https://github.com/user-attachments/assets/916e9325-36d3-425a-b502-3421ea86acf1)

- To ensure that your databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones.

- To configure RDS, follow steps below:

Create a subnet group and add 2 private subnets (data Layer)

- ![dbsubnet](https://github.com/user-attachments/assets/ef551d66-54d5-48e6-8a26-e7acfa09f536)
-![subnetsetup](https://github.com/user-attachments/assets/09519097-ee8e-4e44-80ff-8c1f0c0610f1)
- ![Screenshot from 2024-10-26 12-38-48](https://github.com/user-attachments/assets/63474711-9005-4ab7-b334-507b58608a9c)

2. Create the DataBase by navigating to amazon rds > databases > create database and follow the diagram below

- ![Screenshot from 2024-10-26 12-47-50](https://github.com/user-attachments/assets/0ece4aa0-4420-4f6a-b03a-7771882102e4)
- ![Screenshot from 2024-10-26 12-45-55](https://github.com/user-attachments/assets/e26aa33d-d815-49db-b666-eb8b6e9a949e)
- ![Screenshot from 2024-10-26 12-45-11](https://github.com/user-attachments/assets/dd671a82-a069-473f-9e74-911273a31e90)
- ![Screenshot from 2024-10-26 12-45-29](https://github.com/user-attachments/assets/aabf6cf0-fe45-4e0a-8a80-9adc64983add)
- ![Screenshot from 2024-10-26 12-46-12](https://github.com/user-attachments/assets/6a3bbeb6-cb19-4c42-9207-9eebba5dc046)
- ![Screenshot from 2024-10-26 12-48-28](https://github.com/user-attachments/assets/0a39ca24-a817-43e4-b5b6-006e21dde6ff)
- ![Screenshot from 2024-10-26 12-49-07](https://github.com/user-attachments/assets/badc352d-34dc-4e12-a607-1cc66ece227a)

## Set Up Compute Resources for Bastion
### Provision the EC2 Instances for Bastion
- Create an EC2 Instance based on Red Hat Enterprise Linux (AMI) (You can search for this ami, RHEL-8.7.0_HVM-20230215-x86_64-13-Hourly2-GP2 ) per each Availability Zone in the same Region and same AZ where you created Nginx server

Ensure that it has the following software installed
. python
. ntp (we used chrnony instead, The is because the ntp package is not available in the repositories for our Enterprise Linux 9 system. Instead of ntp, chrony can be used, which is the default NTP implementation in newer versions of RHEL and its derivatives, including Enterprise Linux 9.)
. net-tools
. vim
. wget
. telnet
. epel-release
. htop


We will use instance to create an ami for launching instances in Auto-scaling groups so all the installations will be done before creating the ami from the instance

## Bastion ami installation

    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-89.rpm 
    sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y 
    sudo systemctl start chronyd 
    sudo systemctl enable chronyd

## Nginx ami installation

     sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
     
     sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
     
     sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
     
     sudo systemctl start chronyd
     
     sudo systemctl enable chronyd

## configure selinux policies for the nginx servers

    sudo setsebool -P httpd_can_network_connect=1
    sudo setsebool -P httpd_can_network_connect_db=1
    sudo setsebool -P httpd_execmem=1
    sudo setsebool -P httpd_use_nfs 1

## This section will install amazon efs utils for mounting the target on the Elastic file system

     git clone https://github.com/aws/efs-utils
     
     cd efs-utils
     
     sudo yum install -y make
     
     yum install -y rpm-build
     
     # openssl-devel is needed by amazon-efs-utils-2.0.4-1.el9.x86_64
     sudo yum install openssl-devel -y
     
     # Cargo command needs to be installed as it is necessary for building the Rust project included in the source.
     sudo yum install cargo -y
     
     make rpm 
     
     yum install -y  ./build/amazon-efs-utils*rpm

## Setting up self-signed certificate for the nginx instance

     sudo mkdir /etc/ssl/private
     
     sudo chmod 700 /etc/ssl/private
     
     openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/citatech.key -out /etc/ssl/certs/citatech.crt
     
     sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

## Webserver ami installation

     sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
     
     sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
     
     sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
     
     sudo systemctl start chronyd
     
     sudo systemctl enable chronyd

## configure selinux policies for the webservers

     sudo setsebool -P httpd_can_network_connect=1
     sudo setsebool -P httpd_can_network_connect_db=1
     sudo setsebool -P httpd_execmem=1
     sudo setsebool -P httpd_use_nfs 1

## This section will install amazon efs utils for mounting the target on the Elastic file system

      git clone https://github.com/aws/efs-utils
      
      cd efs-utils
      
      sudo yum install -y make
      
      yum install -y rpm-build
      
      # openssl-devel is needed by amazon-efs-utils-2.0.4-1.el9.x86_64
      sudo yum install openssl-devel -y
      
      # Cargo command needs to be installed as it is necessary for building the Rust project included in the source.
      sudo yum install cargo -y
      
      make rpm 
      
      yum install -y  ./build/amazon-efs-utils*rpm

## Setting up self-signed certificate for the apache webserver instance

yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/citatech.key -x509 -days 365 -out /etc/pki/tls/certs/citatech.crt

cat <<EOL > /etc/httpd/conf.d/your_domain_or_ip.conf
<VirtualHost *:443>
    ServerName your_domain_or_ip
    DocumentRoot /var/www/ssl-test
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/pki/tls/private/apache-selfsigned.key
</VirtualHost>
EOL



### Use these references to understand more
[IP ranges](https://ipinfo.io/ips)

[Nginx reverse proxy server](https://www.nginx.com/resources/glossary/reverse-proxy-server/)

[Understanding ec2 user data
](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
[Manually installing the Amazon EFS client](https://docs.aws.amazon.com/efs/latest/ug/installing-amazon-efs-utils.html#installing-other-distro)

[creating target groups for AWS Loadbalancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

[Self-Signed SSL Certificate for Apache](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-on-centos-8)

[Create a Self-Signed SSL Certificate for Nginx](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7)

### Create AMIs from the 3 instances
- To create an AMI (Amazon Machine Image) from the 3 instances , you need to go to the instance page > select the instance > actions > images and templates > create image. Repeat same process for bastion, nginx and webservers

-![createImage](https://github.com/user-attachments/assets/b973410b-acda-4040-8fb4-464795ab9fce)
- ![Amis](https://github.com/user-attachments/assets/a000ffc5-3df6-4ee1-be56-c0262928ff78)

## Configure Load balancers and Target Groups
1. Create Target group for NGINX, tooling amd wordpress targets






































































































































































































































































































