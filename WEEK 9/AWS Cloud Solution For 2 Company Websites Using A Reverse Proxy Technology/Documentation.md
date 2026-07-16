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

- ![target group1](https://github.com/user-attachments/assets/ff46b7a2-f253-4847-a994-d179974b0591)

- ![targetgroup2](https://github.com/user-attachments/assets/19d83f15-9b1e-4728-8680-7ae309d94775)

- ![alltargetgroup](https://github.com/user-attachments/assets/df20554d-f14a-481c-b014-9e723b0eed2c)

2. Configure Application Load Balancer (ALB)

External Application Load Balancer To Route Traffic To NGINX

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to validate certificates for every request.

Create an Internet facing ALB, Ensure that it listens on HTTPS protocol (TCP port 443) Ensure the ALB is created within the appropriate VPC | AZ | Subnets Choose the Certificate from ACM Select Security Group Select Nginx Instances as the target group

- ![loadbalancer1](https://github.com/user-attachments/assets/245de1f6-6d9e-48a5-9411-cee246b72e06)

- ![Loadbalancer2](https://github.com/user-attachments/assets/0b1a75e4-64a1-48ef-9fb2-4bfbd8872bcf)

- ![Loadbalancer3](https://github.com/user-attachments/assets/a91d18b3-7ef3-47a9-b148-cb6b09ba8307)

- ![Loadbalancer4](https://github.com/user-attachments/assets/ce9b8752-3a37-48ff-b580-2cd9676749df)

- ![Loadbalancer5](https://github.com/user-attachments/assets/e45db0db-8316-4d12-b38e-6cc5092ed464)

- ![Loadbalancer6](https://github.com/user-attachments/assets/fdd3897b-f669-47f2-9e22-a6212404ad88)

- ![Loadbalancer7](https://github.com/user-attachments/assets/2e38a31e-4f7b-451e-97e2-2159f5f65890)


Application Load Balancer To Route Traffic To Webservers

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

Create an Internal ALB Ensure that it listens on HTTPS protocol (TCP port 443) Ensure the ALB is created within the appropriate VPC | AZ | Subnets Choose the Certificate from ACM Select Security Group Select webserver Instances as the target group

- ![InternalLoadbalancer1](https://github.com/user-attachments/assets/5f154459-0611-4900-a1e3-bb4d1a40cfdf)

- ![InternalLoadbalancer2](https://github.com/user-attachments/assets/c210827f-50a4-43d9-a10c-be8ce9342632)
  
- ![ALBCreated](https://github.com/user-attachments/assets/09174f87-8a99-4498-86ff-5f872bfddf5f)

The default target configured on the listener while creating the internal load balancer is to forward traffic to wordpress on port 443. Hence, we need to create a rule to route traffic to tooling as well.

1. Select internal load balancer from the list of load balancers created: *Choose the load balancer where you want to add the rule.*
2.  Listeners Tab: *Click on the Listeners tab.* Select the listener (HTTPS:443) and click Manage listener. 3. Add Rules: * Click on Add rule. 4. Configure the Rule: *Give the rule a name and click next* Add a condition by selecting Host header. *Enter the hostnames for which you want to route traffic. (tooling.com and www.tooling.com).* Choose the appropriate target group for the hostname.

- ![rule](https://github.com/user-attachments/assets/7c7d92c8-df29-4e88-9948-045714cc8ef4)

- ![toolingcondition](https://github.com/user-attachments/assets/3d771b43-8579-413b-ba6c-5c1adb4f6ca7)

- ![toolingconditionSet](https://github.com/user-attachments/assets/4e82943f-cc29-4803-912c-d995d3292965)

- ![toolingconditionl](https://github.com/user-attachments/assets/900ecd8f-4ecb-4984-b695-13298d695216)

- ![toolingcondition5](https://github.com/user-attachments/assets/821b2ce8-f050-455b-811c-2495b787b396)

- ![toolingcondition6](https://github.com/user-attachments/assets/739358ef-8f43-4261-a976-e911d7683452)


PREPARE LAUNCH TEMPLATE FOR NGINX (ONE PER SUBNET)

1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet.
3. Assign appropriate security group.
4. Configure userdata to update yum package repository and install nginx. Ensure to enable auto- assign public IP in the Advance Network Configuration. you can access  the userdata below and edit it with your details

- ![AMI1](https://github.com/user-attachments/assets/2ae7aa98-1e7f-458d-8135-8c240d915e99)

- ![NgnixAMI](https://github.com/user-attachments/assets/707f4d8b-5f42-41a4-8426-39a66dba243d)




We need to update the reverse.conf file by updating proxy_pass value to the end point of the internal load balancer (DNS name) before using the userdata so as to clone the updated repository



                #!/bin/bash
                sudo yum install -y nginx
                sudo systemctl start nginx
                sudo systemctl enable nginx
                
                mkdir -p /etc/ssl/certs /etc/ssl/private
   
                cat <<EOL > /etc/nginx/nginx.conf
                user nginx;
                worker_processes auto;
                error_log /var/log/nginx/error.log;
                pid /run/nginx.pid;

                include /usr/share/nginx/modules/*.conf;
                
                events {
                    worker_connections 1024;
                }
                
                http {
                    log_format  main  '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                                      '\$status \$body_bytes_sent "\$http_referer" '
                                      '"\$http_user_agent" "\$http_x_forwarded_for"';
                
                    access_log  /var/log/nginx/access.log  main;
                
                    sendfile            on;
                    tcp_nopush          on;
                    tcp_nodelay         on;
                    keepalive_timeout   65;
                    types_hash_max_size 2048;
                
                    default_type        application/octet-stream;
                
                    include /etc/nginx/conf.d/*.conf;
                
                    server {
                        listen       80;
                        listen       443 http2 ssl;
                        listen       [::]:443 http2 ssl;
                        root          /var/www/html;
                        server_name  *.liberttinnii.xyz;
                
                        ssl_certificate /etc/ssl/certs/ktrontech.crt;
                        ssl_certificate_key /etc/ssl/private/ktrontech.key;
                        ssl_dhparam /etc/ssl/certs/dhparam.pem;
                
                        location /healthstatus {
                            access_log off;
                            return 200;
                        }
                
                        location / {
                            proxy_set_header             Host \$host;
                            proxy_pass                  https://internal-Ktrontech-Internal-LoadBalancer-1704792078.us-east-1.elb.amazonaws.com;
                        }
                    }
                }
                EOL
                
                systemctl restart nginx

Repeat the same setting for Bastion, the difference here is the userdata input, AMI and security group with the userdata.

    #!/bin/bash
    sudo yum install -y mysql
    sudo yum install -y git tmux 
    sudo yum install -y ansible

NB: Both Wordpress and Tooling make use of Webserver AMI.

Update the mount point to the file system, this should be done on access points for tooling and wordpress respectively., to get the mount point for each access points,

Go to EFS > access points and select wordpress, click on attach, and there you can seen the command to attach the access points mount points

- ![toolingaccesspoint](https://github.com/user-attachments/assets/7d3a80d3-ec1c-4925-bf10-67193d1789cf)

     sudo mount -t efs -o tls,accesspoint=fsap-065070992a8bcd118 fs-0782c677414d9f126:/ efs

The RDS end point is also needed. Go to RDS > Databases > select the database you created earlier
1. Create Template 

- ![rds](https://github.com/user-attachments/assets/ea5ba458-3ab6-4e34-8b73-855db37c3465)

- ![toolinguserdata](https://github.com/user-attachments/assets/c69749d6-c8bb-4564-b97e-d8009fa6cf90)



                #!/bin/bash
                sudo mkdir /var/www/
                sudo mount -t efs -o tls,accesspoint=fsap-065070992a8bcd118 fs-0782c677414d9f126:/  /var/www/
                sudo yum install -y httpd 
                sudo systemctl start httpd
                sudo systemctl enable httpd
                sudo yum module reset php -y
                sudo yum module enable php:remi-7.4 -y
                sudo yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
                sudo systemctl start php-fpm
                sudo systemctl enable php-fpm
                wget http://wordpress.org/latest.tar.gz
                tar xzvf latest.tar.gz
                rm -rf latest.tar.gz
                sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
                mkdir /var/www/html/
                sudo cp -R /wordpress/* /var/www/html/
                cd /var/www/html/
                sudo touch healthstatus
                sudo sed -i "s/localhost/ktrontechdatabase.c18co0o2qbmr.us-east-1.rds.amazonaws.com/g" wp-config.php 
                sudo sed -i "s/username_here/admin/g" wp-config.php 
                sudo sed -i "s/password_here/Willy211#/g" wp-config.php 
                sudo sed -i "s/database_name_here/ktrontechdatabase/g" wp-config.php 
                chcon -t httpd_sys_rw_content_t /var/www/html/ -R
                sudo systemctl restart httpd
                sudo systemctl status httpd
Tooling userdata ( Repeat same steps as you did for wordpress user-data for tooling, difference is, get the tooling access points mount point command instead)

- ![toolingtemplate](https://github.com/user-attachments/assets/d5c030fa-ab17-4eda-8ddd-f0eaeaf15f8d)



                #!/bin/bash
                sudo mkdir /var/www/
                sudo mount -t efs -o tls,accesspoint=fsap-065070992a8bcd118 fs-0782c677414d9f126://var/www/
                sudo yum install -y httpd 
                sudo systemctl start httpd
                sudo systemctl enable httpd
                sudo yum module reset php -y
                sudo yum module enable php:remi-7.4 -y
                sudo yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
                sudo systemctl start php-fpm
                sudo systemctl enable php-fpm
                git clone https://github.com/citadelict/tooling2.git
                sudo mkdir /var/www/html
                sudo cp -rf tooling2/html/*  /var/www/html/
                cd tooling2
                mysql -h ktrontechdatabase.c18co0o2qbmr.us-east-1.rds.amazonaws.com -u admin -p Willy211# toolingdb < tooling-db.sql
                cd /home/ec2-user
                sudo touch healthstatus
                cd /var/www/html
                sudo sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('ktrontechdatabase.c18co0o2qbmr.us-east-1.rds.amazonaws.com', 'admin', 'Willy211#', 'toolingdb');/g" functions.php
                chcon -t httpd_sys_rw_content_t /var/www/html/ -R
                sudo systemctl restart httpd
                sudo systemctl status httpd

## CONFIGURE AUTOSCALING FOR NGINX
- Select the right launch template
- Select the VPC
- Select both public subnets
- Enable Application Load Balancer for the AutoScalingGroup (ASG)
- Select the target group you created before
- Ensure that you have health checks for both EC2 and ALB
- The desired capacity is 2
- Minimum capacity is 2
- Maximum capacity is 4
- Set scale out if CPU utilization reaches 90%
- Ensure there is an SNS topic to send scaling notifications
  
 ## Create Auto Scaling Group for Bastion
- ![autoscaling](https://github.com/user-attachments/assets/7f986124-b362-4a30-a3b9-ebe77c468250)

- ![bastionautoscaling](https://github.com/user-attachments/assets/eb386b26-bf1c-4e29-ab47-7912fb3804f6)

- ![healthcheck](https://github.com/user-attachments/assets/75162a12-c4f0-417d-a786-d169e620879f)

- ![capacity](https://github.com/user-attachments/assets/d1df0e15-0b7d-4098-b016-5417b9255806)

- ![snstopic](https://github.com/user-attachments/assets/b832e234-786c-414b-aeb2-7c893cad1fcf)

Access RDS through Bastion connection to craete database for wordpress and tooling.

Copy the RDS endpoint to be used as host

- ![connect](https://github.com/user-attachments/assets/4fec4517-6099-4d67-a346-76ccae826226)

- ![DATABASE](https://github.com/user-attachments/assets/337556cf-6f03-4bdc-aa47-1da74596e7bd)


## Create Auto Scaling Group for Nginx

- Go to Auto Scaling Groups in the EC2 Dashboard.

- Click Create Auto Scaling group and follow these steps:

Auto Scaling group name: nginx-auto-scaling-group
- Launch Template: Select nginx-launch-template.
Network:
- Select the VPC in which to launch your instances.
- Choose Subnets where the nginx instances will launch
  
### - Configure Scaling Policies:
- Desired Capacity: Set the initial number of instances i.e 2.
- Minimum Capacity: Minimum number of instances to maintain i.e 2
- Maximum Capacity: Maximum number of instances to allow i.e 4

- Choose EC2 health checks or ELB health checks if using a load balancer.
- Review and Create:

- Review the settings and click Create Auto Scaling Group.
### -Attach a Load Balancer 
- To distribute traffic across your instances, you may attach an Application Load Balancer (ALB) to the ASG:

- In the Auto Scaling Group dashboard, choose Edit next to Load balancing.
- Select your existing ALB or create a new one and attach it to the ASG.

## Repeat the Nginx Auto Scaling Group steps above for Wordpress and Tooling with their right launch template

- ![88](https://github.com/user-attachments/assets/b7dece77-1bf6-4a2e-9c15-773168b59acf)

## Configuring DNS with Route53
- Earlier in this project we registered a free domain with Hostinger and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

- We need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

- Create other records such as CNAME, alias and A records.

**NOTE**: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read https://support.dnsimple.com/articles/differences-between-a-cname-alias-url/#:~:text=The%20A%20record%20maps%20a,a%20name%20to%20another%20name.&text=The%20ALIAS%20record%20maps%20a,the%20HTTP%20301%20status%20code to get to know more about the differences.

- Create an alias record for the root domain and direct its traffic to the ALB DNS name.

- Create an alias record for _liberttinnii.xyz.online_ and direct its traffic to the ALB DNS name.

- ![Screenshot from 2024-10-28 13-02-03](https://github.com/user-attachments/assets/9da4501a-e592-4ee7-84cb-ce5f1a66a3d9)

Test your websites.
Now let us access our tooling website via a browser using our DNS name

![website](https://github.com/user-attachments/assets/25bdb3f4-c25e-4ec7-800c-6842bb27d94a)

## Let's access our wordpress website


![93](https://github.com/user-attachments/assets/5c61316e-930c-40a3-bdf5-f522bb67e7bc)

















































































































































































































































