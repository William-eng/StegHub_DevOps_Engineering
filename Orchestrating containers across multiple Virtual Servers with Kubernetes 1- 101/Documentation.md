## Orchestrating containers across multiple Virtual Servers with Kubernetes 1- 101
This project is the first of a series Kubernetes-related practice projects, so get ready to become a professional containers' fleet "pilot"*.

*Kubernetes (κυβερνήτης, Greek for "helmsman" or "pilot" or "governor", and the etymological root of cybernetics)

- ![image-114-1024x681](https://github.com/user-attachments/assets/0d00ce77-9bd7-4d05-a43e-aed43a428257)

It is important to understand that, DevOps is about "Culture" and NOT "Tools" Therefore, it is not correct to say that one tool is better than another; different organizations have different needs and a good tool for 
one team may be bad for another just because their needs are not the same. In some teams, Docker Compose fit their needs perfectly, despite the perceived limitations. The major limitation of Docker Compose is that it 
can only be used to run workloads on a single computer host. Now, that is an obvious limitation because if our **Tooling Application** and its **MySQL Database** are all running on a single VM, like we did in Project 20, then
this host is considered as a **SPOF (i.e. - Single Point Of Failure)**.

So, could we say there is something wrong with Docker Compose? Not exactly, as a matter of fact, it is being used a lot in the industry. It fits well into some use cases that require speedy development and Proof of
Concepts. As you will soon see, Kubernetes is a lot more complex technology, and it may be an overkill for some use cases.


**Container orchestration** is a concept that allows to address these two scenarios, it provides automation of all the aspects of coordinating and managing containers. Container orchestration is focused on managing 
life cycle of containers and their dynamic environments.

It is about automating the entire lifecycle of containers running across multiple nodes:

- Configuring and scheduling of containers on nodes
- Ensuring the availability of containers, even when they die
- Scaling of containers to equally balance application workloads across infrastructure
- Allocation of resources between containers
- Load balancing, traffic routing and service discovery of containers
- Health monitoring of containers
- Securing the interactions between containers.
Kubernetes is a tool designed to do Container Orchestration and it does its job very well when correctly configured.

As mentioned earlier, there are other alternatives to Docker Compose. But, throughout the entire PBL program, we will not focus on Docker Swarm. We will rather spend more time with Kubernetes. 
Part of the reason for this is because Kubernetes has more functionalities and is widely in use in the industry.

## Kubernetes architecture
Kubernetes is a not a single package application that you can install with one command, it is comprised of several components, some of them can be deployed as services, some can be also deployed as separate containers.

Let us take a look at Kubernetes architecture diagram below:

- ![image-115-1024x600](https://github.com/user-attachments/assets/863ef095-03ed-474a-b238-fb1771396884)

      Docker Swarm is preferred in environments where simplicity and fast development is preferred, whereas Kubernetes is fit for the environments where medium to large clusters are running complex applications.

- [official documentation.](https://kubernetes.io/docs/concepts/overview/components/)

## "Kubernetes From-Ground-Up"
### K8s installation options
To successfully implement "K8s From-Ground-Up", the following and even more will be done by you as a K8s administrator
1. Install and configure master (also known as control plane) components and worker nodes (or just nodes).
2. Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
3. In transit encryption means encrypting communications over the network using HTTPS
4. At rest encryption means encrypting the data stored on a disk
5. Plan the capacity for the backend data store etcd
6. Configure network plugins for the containers to communicate
7. Manage periodical upgrade of the cluster
8. Configure observability and auditing

## Tools to be used and expected result of the Project 20
- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster
  
  We will create **6 EC2 Instances**, and in the end, we will have the following parts of the cluster properly configured:
- Three Kubernetes Master
- Three Kubernetes Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

## Step 0 Install client tools before bootstrapping the cluster.
First, you will need some client tools installed and configurations made on your client workstation:

- [awscli](https://aws.amazon.com/cli/) - is a unified tool to manage your AWS services
- [kubectl](https://kubernetes.io/docs/reference/kubectl/) - this command line utility will be your main control tool to manage your K8s cluster. You will use this tool so many times, so you will be able to type 'kubetcl' on your keyboard with a speed of light. You can always make a shortcut (alias) to just one character 'k'. Also, add this extremely useful official kubectl Cheat Sheet to your bookmarks, it has examples of the most used 'kubectl' commands.
- [cfssl](https://blog.cloudflare.com/introducing-cfssl/) - an open source toolkit for everything TLS/SSL from [Cloudflare](https://www.cloudflare.com/en-gb/)
- [cfssljson](https://github.com/cloudflare/cfssl) - a program, which takes the JSON output from the cfssl and writes certificates, keys, [CSRs](https://en.wikipedia.org/wiki/Certificate_signing_request), and bundles to disk.

## Install and configure AWS CLI
Configure AWS CLI to access all AWS services used, for this we need to have a user with programmatic access keys configured in AWS Identity and Access Management (IAM):
- ![K8s1](https://github.com/user-attachments/assets/e9a6692f-522d-4b78-96ac-5b5c856cc3f0)

- ![k8s2](https://github.com/user-attachments/assets/079adaf5-babc-4fdf-9af8-74f3384c9473)
Generate access keys and store them in a safe place.

On your local workstation download and install the latest version of [AWS CLI](https://aws.amazon.com/cli/)

[To configure your AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) - run your shell (or cmd if using Windows) and run:

      $ aws configure --profile %your_username%
      AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
      AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
      Default region name [None]: us-west-2
      Default output format [None]: table

Test your AWS CLI by running:

      aws ec2 describe-vpcs

- ![vpccheck](https://github.com/user-attachments/assets/f3a0e0a6-7aaa-4520-864a-75b6e3330d39)
  
### Install kubectl

Kubernetes cluster has a Web API that can receive HTTP/HTTPS requests, but it is quite cumbersome to curl an API each and every time you need to send some command, so kubectl command tool was developed to ease a K8s administrator's life.

With this tool you can easily interact with Kubernetes to deploy applications, inspect and manage cluster resources, view logs and perform many more administrative operations.

**Mac OS X**

- Download the binary

      curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/darwin/amd64/kubectl
- Make it executable

      chmod +x kubectl
- Move to the Bin directory

      sudo mv kubectl /usr/local/bin/
  
**Linux Or Windows using Gitbash or similar tool**

- Download the binary

      wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
- Make it executable

      chmod +x kubectl
- Move to the Bin directory

      sudo mv kubectl /usr/local/bin/
- Verify that kubectl version 1.21.0 or higher is installed:

      kubectl version --client
**Output:**

      Client Version: version.Info{Major:"1", Minor:"20+", GitVersion:"v1.20.4-dirty", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aafd07c", GitTreeState:"dirty", BuildDate:"2021-03-15T10:03:32Z", GoVersion:"go1.16.2", Compiler:"gc", Platform:"darwin/amd64"}
      
- ![kubectlinstall](https://github.com/user-attachments/assets/e1cb3205-fd81-448d-a5be-740b6cf6b3d2)

### Install CFSSL and CFSSLJSON
_cfssl_ is an open source tool by **Cloudflare** used to setup a **Public Key Infrastructure** ( [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) ) for generating, signing and bundling TLS certificates. In previous projects you have experienced the use of **Letsencrypt** for the similar use case. Here, _cfssl_ will be configured as a Certificate Authority which will issue the certificates required to spin up a Kubernetes cluster.

Download, install and verify successful installation of cfssl and cfssljson:

**Mac OS X**

      curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
      curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
      chmod +x cfssl cfssljson
      sudo mv cfssl cfssljson /usr/local/bin/
      
If you have issues using the binaries directly, you should consider using the package manager [Homebrew](https://brew.sh/) and that might be a better option:

      brew install cfssl
      
Verify that _cfssl_ version 1.4.1 or higher is installed:

**Output**: cfssl version

      Version: 1.4.1
      Runtime: go1.12.12
      
cfssljson --version

      Version: 1.4.1
      Runtime: go1.12.12

### Linux Or Windows using Gitbash or similar tool

      wget -q --show-progress --https-only --timestamping \
        https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
        https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
        
      chmod +x cfssl cfssljson
      
      sudo mv cfssl cfssljson /usr/local/bin/
- ![cfsslinstalled](https://github.com/user-attachments/assets/f7e45093-bb6a-4d34-91a3-cd82bf77252e)


## AWS Cloud resources for Kubernetes Cluster
As we already know, we need some machines to run the control plane and the worker nodes. In this section, you will provision EC2 Instances required to run your K8s cluster. You can use Terraform for this. But it is highly recommended to start out first with manual provisioning using awscli and have thorough knowledge about the whole setup. After that, you can destroy the entire project and start all over again using Terraform. This manual approach will solidify your skills and give you the opportunity to face more challenges.

### Step 1 - Configure Network Infrastructure
**Virtual Private Cloud - VPC**

1. Create a directory named k8s-cluster-from-ground-up
2. Create a VPC and store the ID as a variable:

            VPC_ID=$(aws ec2 create-vpc \
              --cidr-block 172.31.0.0/16 \
              --output text --query 'Vpc.VpcId'
              )
3. Tag the VPC so that it is named:

            NAME=k8s-cluster-from-ground-up
            
            aws ec2 create-tags \
              --resources ${VPC_ID} \
              --tags Key=Name,Value=${NAME}
4. Domain Name System - DNS

**Enable DNS support for your VPC:**

      aws ec2 modify-vpc-attribute \
        --vpc-id ${VPC_ID} \
        --enable-dns-support '{"Value": true}'
5. Enable DNS support for hostnames:

            aws ec2 modify-vpc-attribute \
              --vpc-id ${VPC_ID} \
              --enable-dns-hostnames '{"Value": true}'

- ![VPC1](https://github.com/user-attachments/assets/5e096bf6-c69c-4adc-aa0c-fb4a663da52c)

- ![AWSVPC1](https://github.com/user-attachments/assets/25c010bd-08c1-4701-ad23-f1ec8aca9d82)

**AWS Region**

6. Set the required region

            AWS_REGION=us-east-1
**Subnet**

7. Create the Subnet:

            SUBNET_ID=$(aws ec2 create-subnet \
              --vpc-id ${VPC_ID} \
              --cidr-block 172.31.0.0/24 \
              --output text --query 'Subnet.SubnetId')
            aws ec2 create-tags \
              --resources ${SUBNET_ID} \
              --tags Key=Name,Value=${NAME}

**Internet Gateway - IGW**

8. Create the Internet Gateway and attach it to the VPC:

            INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
              --output text --query 'InternetGateway.InternetGatewayId')
            aws ec2 create-tags \
              --resources ${INTERNET_GATEWAY_ID} \
              --tags Key=Name,Value=${NAME}
            aws ec2 attach-internet-gateway \
              --internet-gateway-id ${INTERNET_GATEWAY_ID} \
              --vpc-id ${VPC_ID}
**Route tables**

9. Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:

            ROUTE_TABLE_ID=$(aws ec2 create-route-table \
              --vpc-id ${VPC_ID} \
              --output text --query 'RouteTable.RouteTableId')
            aws ec2 create-tags \
              --resources ${ROUTE_TABLE_ID} \
              --tags Key=Name,Value=${NAME}
            aws ec2 associate-route-table \
              --route-table-id ${ROUTE_TABLE_ID} \
              --subnet-id ${SUBNET_ID}
            aws ec2 create-route \
              --route-table-id ${ROUTE_TABLE_ID} \
              --destination-cidr-block 0.0.0.0/0 \
              --gateway-id ${INTERNET_GATEWAY_ID}
- ![VPC2](https://github.com/user-attachments/assets/a5cc2218-430d-4bb5-a9b0-0511fe610d38)

   
**Output:**

-------------------------------------------------
|              AssociateRouteTable              |
+----------------+------------------------------+
|  AssociationId |  rtbassoc-060acb5d6ab6e83d1  |
+----------------+------------------------------+
||              AssociationState               ||
|+----------------+----------------------------+|
||  State         |  associated                ||
|+----------------+----------------------------+|
--------------------
|    CreateRoute   |
+---------+--------+
|  Return |  True  |
+---------+--------+

**Security Groups**

10. Configure security groups

                  # Create the security group and store its ID in a variable
                  SECURITY_GROUP_ID=$(aws ec2 create-security-group \
                    --group-name ${NAME} \
                    --description "Kubernetes cluster security group" \
                    --vpc-id ${VPC_ID} \
                    --output text --query 'GroupId')
                  
                  # Create the NAME tag for the security group
                  aws ec2 create-tags \
                    --resources ${SECURITY_GROUP_ID} \
                    --tags Key=Name,Value=${NAME}
                    
                  # Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
                  aws ec2 authorize-security-group-ingress \
                      --group-id ${SECURITY_GROUP_ID} \
                      --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

    - ![VPC3](https://github.com/user-attachments/assets/25d429c1-5ec5-412a-b976-8b388ec57bde)


**output**:

                  ----------------------------------------------------
|           AuthorizeSecurityGroupIngress          |
+---------------------------+----------------------+
|  Return                   |  True                |
+---------------------------+----------------------+
||               SecurityGroupRules               ||
|+----------------------+-------------------------+|
||  CidrIpv4            |  172.31.0.0/24          ||
||  FromPort            |  2379                   ||
||  GroupId             |  sg-07995a7b33a618e3b   ||
||  GroupOwnerId        |  430118841581           ||
||  IpProtocol          |  tcp                    ||
||  IsEgress            |  False                  ||
||  SecurityGroupRuleId |  sgr-0c29fa914d4805c74  ||
||  ToPort              |  2380                   ||
|+----------------------+-------------------------+|

                  
                  # # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
                  aws ec2 authorize-security-group-ingress \
                      --group-id ${SECURITY_GROUP_ID} \
                      --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'
**output:**

----------------------------------------------------
|           AuthorizeSecurityGroupIngress          |
+---------------------------+----------------------+
|  Return                   |  True                |
+---------------------------+----------------------+
||               SecurityGroupRules               ||
|+----------------------+-------------------------+|
||  CidrIpv4            |  172.31.0.0/24          ||
||  FromPort            |  30000                  ||
||  GroupId             |  sg-07995a7b33a618e3b   ||
||  GroupOwnerId        |  430118841581           ||
||  IpProtocol          |  tcp                    ||
||  IsEgress            |  False                  ||
||  SecurityGroupRuleId |  sgr-0533c8baef6eccb62  ||
||  ToPort              |  32767                  ||
|+----------------------+-------------------------+|
                  
                  # Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
                  aws ec2 authorize-security-group-ingress \
                    --group-id ${SECURITY_GROUP_ID} \
                    --protocol tcp \
                    --port 6443 \
                    --cidr 0.0.0.0/0

**outputs:**
----------------------------------------------------
|           AuthorizeSecurityGroupIngress          |
+---------------------------+----------------------+
|  Return                   |  True                |
+---------------------------+----------------------+
||               SecurityGroupRules               ||
|+----------------------+-------------------------+|
||  CidrIpv4            |  0.0.0.0/0              ||
||  FromPort            |  6443                   ||
||  GroupId             |  sg-07995a7b33a618e3b   ||
||  GroupOwnerId        |  430118841581           ||
||  IpProtocol          |  tcp                    ||
||  IsEgress            |  False                  ||
||  SecurityGroupRuleId |  sgr-04bca28441be77db8  ||
||  ToPort              |  6443                   ||
|+----------------------+-------------------------+|
                  
                  # Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
                  aws ec2 authorize-security-group-ingress \
                    --group-id ${SECURITY_GROUP_ID} \
                    --protocol tcp \
                    --port 22 \
                    --cidr 0.0.0.0/0

**output:**
----------------------------------------------------
|           AuthorizeSecurityGroupIngress          |
+---------------------------+----------------------+
|  Return                   |  True                |
+---------------------------+----------------------+
||               SecurityGroupRules               ||
|+----------------------+-------------------------+|
||  CidrIpv4            |  0.0.0.0/0              ||
||  FromPort            |  22                     ||
||  GroupId             |  sg-07995a7b33a618e3b   ||
||  GroupOwnerId        |  430118841581           ||
||  IpProtocol          |  tcp                    ||
||  IsEgress            |  False                  ||
||  SecurityGroupRuleId |  sgr-06ed1ddf978b55fa7  ||
||  ToPort              |  22                     ||
|+----------------------+-------------------------+|
                  
                  # Create ICMP ingress for all types
                  aws ec2 authorize-security-group-ingress \
                    --group-id ${SECURITY_GROUP_ID} \
                    --protocol icmp \
                    --port -1 \
                    --cidr 0.0.0.0/0
**output:**
----------------------------------------------------
|           AuthorizeSecurityGroupIngress          |
+---------------------------+----------------------+
|  Return                   |  True                |
+---------------------------+----------------------+
||               SecurityGroupRules               ||
|+----------------------+-------------------------+|
||  CidrIpv4            |  0.0.0.0/0              ||
||  FromPort            |  -1                     ||
||  GroupId             |  sg-07995a7b33a618e3b   ||
||  GroupOwnerId        |  430118841581           ||
||  IpProtocol          |  icmp                   ||
||  IsEgress            |  False                  ||
||  SecurityGroupRuleId |  sgr-0779b92aa9cae035d  ||
||  ToPort              |  -1                     ||





























































































































































































































































































































































































