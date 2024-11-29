# Building Elastic Kubernetes Service (EKS) With Terraform-101
Since projects 21 and 22, we have had some fragmented experience around Kubernetes bootstrapping and deployment of containerized applications. This project seeks to solidify your skills by focusing more on real-world setups.

1. We will use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
2. We will deploy multiple applications using HELM
3. We will experience more Kubernetes objects and how to use them with Helm. Such as Dynamic provisioning of volumes to make pods stateful
4. We will improve upon your CI/CD skills with Jenkins
   
In Project 21, we created a k8s cluster from Ground-Up. That was quite painful, but very necessary to help master Kubernetes. Going forward, we will not have to do that. We will hardly ever have to do that even in the real world. given that cloud providers such as AWS have managed services for Kubernetes, they have done all the hard work, and with a few API calls to AWS, you can have a production-grade cluster ready to go in minutes. Therefore, in this project, you begin by focusing on [EKS](https://aws.amazon.com/eks/), and how to get it up and running using Terraform. Before moving on to other things.

## Building EKS with Terraform
At this point, you should already have some experience with Terraform. You have some work ahead of you, but you can begin with the steps outlined below. 

_If you have Terraform code from Project 16, simply update it to include EKS starting from step 6. If not, please follow the steps beginning at number 1_.

**Note:** Use Terraform version v1.0.2 and kubectl version v1.23.6.

1. Open up a new directory on your laptop, and name it eks

         mkdir EKS_Cluster_With_Terraform
         cd EKS_Cluster_With_Terraform
   
2. Use AWS CLI to create an S3 bucket

         aws s3 mb s3://ktrontech-eks-terraform-state --region us-east-1

   - ![Image01](https://github.com/user-attachments/assets/036b5f44-e778-4802-b5f3-7b80554b95ae)


3. Create a file – _backend.tf_ Task for you, ensure the backend is configured for remote state in S3

             terraform {
              backend "s3" {
                bucket         = "your-s3-bucket-name"         # Replace with your S3 bucket name
                key            = "path/to/your/terraform.tfstate" # Replace with your state file path
                region         = "your-region"                # Replace with your AWS region
                dynamodb_table = "terraform-locks"            # Replace with your DynamoDB table name
                encrypt        = true                         # Ensures the state is encrypted
              }
            }

4. Create a file – _network.tf_ and provision Elastic IP for Nat Gateway, VPC, Private and public subnets.

            # reserve Elastic IP to be used in our NAT gateway
            resource "aws_eip" "nat_gw_elastic_ip" {
            vpc = true
            
            tags = {
            Name            = "${var.cluster_name}-nat-eip"
            iac_environment = var.iac_environment_tag
            }
            }

- Create VPC using the official AWS module

               module "vpc" {
               source  = "terraform-aws-modules/vpc/aws"
               
               name = "${var.name_prefix}-vpc"
               cidr = var.main_network_block
               azs  = data.aws_availability_zones.available_azs.names
               
               private_subnets = [
               # this loop will create a one-line list as ["10.0.0.0/20", "10.0.16.0/20", "10.0.32.0/20", ...]
               # with a length depending on how many Zones are available
               for zone_id in data.aws_availability_zones.available_azs.zone_ids :
               cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) - 1)
               ]
               
               public_subnets = [
               # this loop will create a one-line list as ["10.0.128.0/20", "10.0.144.0/20", "10.0.160.0/20", ...]
               # with a length depending on how many Zones are available
               # there is a zone Offset variable, to make sure no collisions are present with private subnet blocks
               for zone_id in data.aws_availability_zones.available_azs.zone_ids :
               cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) + var.zone_offset - 1)
               ]
               
               # Enable single NAT Gateway to save some money
               # WARNING: this could create a single point of failure, since we are creating a NAT Gateway in one AZ only
               # feel free to change these options if you need to ensure full Availability without the need of running 'terraform apply'
               # reference: https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/2.44.0#nat-gateway-scenarios
               enable_nat_gateway     = true
               single_nat_gateway     = true
               one_nat_gateway_per_az = false
               enable_dns_hostnames   = true
               reuse_nat_ips          = true
               external_nat_ip_ids    = [aws_eip.nat_gw_elastic_ip.id]
               
               # Add VPC/Subnet tags required by EKS
               tags = {
               "kubernetes.io/cluster/${var.cluster_name}" = "shared"
               iac_environment                             = var.iac_environment_tag
               }
               public_subnet_tags = {
               "kubernetes.io/cluster/${var.cluster_name}" = "shared"
               "kubernetes.io/role/elb"                    = "1"
               iac_environment                             = var.iac_environment_tag
               }
               private_subnet_tags = {
               "kubernetes.io/cluster/${var.cluster_name}" = "shared"
               "kubernetes.io/role/internal-elb"           = "1"
               iac_environment                             = var.iac_environment_tag
               }
               }

- ![Image02](https://github.com/user-attachments/assets/2406890e-d534-4259-aed1-2e4c2d57d5b6)

**Note**: The tags added to the subnets are very important. The Kubernetes Cloud Controller Manager (cloud-controller-manager) and AWS Load Balancer Controller (aws-load-balancer-controller) need to identify the cluster. To do that, it queries the cluster’s subnets by using the tags as a filter.

- For public and private subnets that use load balancer resources: each subnet must be tagged
  
      Key: kubernetes.io/cluster/cluster-name
      Value: shared
  
- For private subnets that use internal load balancer resources: each subnet must be tagged

      Key: kubernetes.io/role/internal-elb
      Value: 1
  
- For public subnets that use internal load balancer resources: each subnet must be tagged
  
      Key: kubernetes.io/role/elb
      Value: 1

5. Create a file – _variables.tf_

            # create some variables
            variable "cluster_name" {
            type        = string
            description = "EKS cluster name."
            }
            variable "iac_environment_tag" {
            type        = string
            description = "AWS tag to indicate environment name of each infrastructure object."
            }
            variable "name_prefix" {
            type        = string
            description = "Prefix to be used on each infrastructure object Name created in AWS."
            }
            variable "main_network_block" {
            type        = string
            description = "Base CIDR block to be used in our VPC."
            }
            variable "subnet_prefix_extension" {
            type        = number
            description = "CIDR block bits extension to calculate CIDR blocks of each subnetwork."
            }
            variable "zone_offset" {
            type        = number
            description = "CIDR block bits extension offset to calculate Public subnets, avoiding collisions with Private subnets."
            }
- ![Image03](https://github.com/user-attachments/assets/cb8adadd-3a14-47c4-8289-9dea0a3be86b)


6. Create a file – data.tf – This will pull the available AZs for use.

            # get all available AZs in our region
            data "aws_availability_zones" "available_azs" {
            state = "available"
            }
            data "aws_caller_identity" "current" {} # used for accesing Account ID and ARN

7. Create a file – _eks.tf_ and provision EKS cluster (Create the file only if you are not using your existing Terraform code. Otherwise you can simply append it to the main.tf from your existing code) [Read more about this module from the official documentation here](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/18.20.5) – Reading it will help you understand more about the rich features of the module.

            module "eks_cluster" {
              source  = "terraform-aws-modules/eks/aws"
              version = "~> 18.0"
              cluster_name    = var.cluster_name
              cluster_version = "1.22"
              vpc_id     = module.vpc.vpc_id
              subnet_ids = module.vpc.private_subnets
              cluster_endpoint_private_access = true
              cluster_endpoint_public_access = true
            
              # Self Managed Node Group(s)
              self_managed_node_group_defaults = {
                instance_type                          = var.asg_instance_types[0]
                update_launch_template_default_version = true
              }
              self_managed_node_groups = local.self_managed_node_groups
            
              # aws-auth configmap
              create_aws_auth_configmap = true
              manage_aws_auth_configmap = true
              aws_auth_users = concat(local.admin_user_map_users, local.developer_user_map_users)
              tags = {
                Environment = "prod"
                Terraform   = "true"
              }
            }


- ![Image04](https://github.com/user-attachments/assets/3544979e-c84d-408e-9cb7-47bca96366d6)

8. Create a file – _locals.tf_ to create local variables. Terraform does not allow assigning variable to variables. There is good reasons for that to avoid repeating your code unecessarily. So a terraform way to achieve this would be to use locals so that your code can be kept [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)

               # render Admin & Developer users list with the structure required by EKS module
               locals {
                 admin_user_map_users = [
                   for admin_user in var.admin_users :
                   {
                     userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${admin_user}"
                     username = admin_user
                     groups   = ["system:masters"]
                   }
                 ]
                 developer_user_map_users = [
                   for developer_user in var.developer_users :
                   {
                     userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${developer_user}"
                     username = developer_user
                     groups   = ["${var.name_prefix}-developers"]
                   }
                 ]
               
                 self_managed_node_groups = {
                   worker_group1 = {
                     name = "${var.cluster_name}-wg"
               
                     min_size      = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
                     desired_size      = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
                     max_size  = var.autoscaling_maximum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
                     instance_type = var.asg_instance_types[0].instance_type
               
                     bootstrap_extra_args = "--kubelet-extra-args '--node-labels=node.kubernetes.io/lifecycle=spot'"
               
                     block_device_mappings = {
                       xvda = {
                         device_name = "/dev/xvda"
                         ebs = {
                           delete_on_termination = true
                           encrypted             = false
                           volume_size           = 10
                           volume_type           = "gp2"
                         }
                       }
                     }
               
                     use_mixed_instances_policy = true
                     mixed_instances_policy = {
                       instances_distribution = {
                         spot_instance_pools = 4
                       }
               
                       override = var.asg_instance_types
                     }
                   }
                 }
               }

- ![Image05](https://github.com/user-attachments/assets/6b057b9f-c5fb-4853-9531-9d6944fd8bb7)

9. Add more variables to the _variables.tf_ file


            # create some variables
            variable "admin_users" {
              type        = list(string)
              description = "List of Kubernetes admins."
            }
            variable "developer_users" {
              type        = list(string)
              description = "List of Kubernetes developers."
            }
            variable "asg_instance_types" {
              description = "List of EC2 instance machine types to be used in EKS."
            }
            variable "autoscaling_minimum_size_by_az" {
              type        = number
              description = "Minimum number of EC2 instances to autoscale our EKS cluster on each AZ."
            }
            variable "autoscaling_maximum_size_by_az" {
              type        = number
              description = "Maximum number of EC2 instances to autoscale our EKS cluster on each AZ."
            }

10. Create a file – _variables.tfvars_ to set values for variables.

            cluster_name            = "tooling-app-eks"
            iac_environment_tag     = "development"
            name_prefix             = "steghub-com-eks"
            main_network_block      = "10.0.0.0/16"
            subnet_prefix_extension = 4
            zone_offset             = 8
            
            # Ensure that these users already exist in AWS IAM. Another approach is that you can introduce an iam.tf file to manage users separately, get the data source and interpolate their ARN.
            admin_users                    = ["james", "solomon"]
            developer_users                = ["leke", "david"]
            asg_instance_types             = [ { instance_type = "t3.small" }, { instance_type = "t2.small" }, ]
            autoscaling_minimum_size_by_az = 1
            autoscaling_maximum_size_by_az = 10



- ![Image06](https://github.com/user-attachments/assets/2b53f9ef-3935-4156-9ce5-53dc7f7c41c5)

11. Create file – provider.tf

               provider "aws" {
                 region = "us-west-1"
               }
               
               provider "random" {
               }


- ![Image07](https://github.com/user-attachments/assets/55f6f6b9-d432-4a14-901d-ec9dc5c8931c)

Update the file – variables.tfvars to set values for variables.

         autoscaling_average_cpu                  = 30


12. Run _terraform init_

- ![Image08](https://github.com/user-attachments/assets/7078a554-37ae-4201-935b-93e4834554ad)

    
13. Run Terraform plan – Your plan should have an output

- ![Image09](https://github.com/user-attachments/assets/cf2d11e8-ac31-452d-b2ae-04de7aab505f)


14. Run Terraform apply
This will begin to create cloud resources, and fail at some point with the error

               ╷
                        │ Error: Post "http://localhost/api/v1/namespaces/kube-system/configmaps": dial tcp [::1]:80: connect: connection refused
                        │ 
                        │   with module.eks-cluster.kubernetes_config_map.aws_auth[0],
                        │   on .terraform/modules/eks-cluster/aws_auth.tf line 63, in resource "kubernetes_config_map" "aws_auth":
                        │   63: resource "kubernetes_config_map" "aws_auth" {      
               

That is because for us to connect to the cluster using the kubeconfig, Terraform needs to be able to connect and set the credentials correctly.

## FIXING THE ERROR
To fix this problem

- Append to the file _data.tf_

            # get EKS cluster info to configure Kubernetes and Helm providers
            data "aws_eks_cluster" "cluster" {
              name = module.eks_cluster.cluster_id
            }
            data "aws_eks_cluster_auth" "cluster" {
              name = module.eks_cluster.cluster_id
            }
- ![Image10](https://github.com/user-attachments/assets/5959cc0e-db06-4f2a-b5c0-328950191ac2)


- Append to the file _provider.tf_

            # get EKS authentication for being able to manage k8s objects from terraform
            provider "kubernetes" {
              host                   = data.aws_eks_cluster.cluster.endpoint
              cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
              token                  = data.aws_eks_cluster_auth.cluster.token
            }

- Run the init and plan again – This time you will see


- ![Image11](https://github.com/user-attachments/assets/73ccc570-0f58-4a52-8b0d-19affe64e7cb)


                 # module.eks-cluster.kubernetes_config_map.aws_auth[0] will be created
                 + resource "kubernetes_config_map" "aws_auth" {
                     + data = {
                         + "mapAccounts" = jsonencode([])
                         + "mapRoles"    = <<-EOT
                               - "groups":
                                 - "system:bootstrappers"
                                 - "system:nodes"
                                 "rolearn": "arn:aws:iam::696742900004:role/tooling-app-eks20210718113602300300000009"
                                 "username": "system:node:{{EC2PrivateDNSName}}"
                           EOT
                         + "mapUsers"    = <<-EOT
                               - "groups":
                                 - "system:masters"
                                 "userarn": "arn:aws:iam::696742900004:user/james"
                                 "username": "james"
                               - "groups":
                                 - "system:masters"
                                 "userarn": "arn:aws:iam::696742900004:user/solomon"
                                 "username": "solomon"
                               - "groups":
                                 - "steghub-com-eks-developers"
                                 "userarn": "arn:aws:iam::696742900004:user/leke"
                                 "username": "leke"
                               - "groups":
                                 - "steghub-com-eks-developers"
                                 "userarn": "arn:aws:iam::696742900004:user/david"
                                 "username": "david"
                           EOT
                       }
                     + id   = (known after apply)
               
                     + metadata {
                         + generation       = (known after apply)
                         + labels           = {
                             + "app.kubernetes.io/managed-by" = "Terraform"
                             + "terraform.io/module"          = "terraform-aws-modules.eks.aws"
                           }
                         + name             = "aws-auth"
                         + namespace        = "kube-system"
                         + resource_version = (known after apply)
                         + uid              = (known after apply)
                       }
                   }
               
               Plan: 1 to add, 0 to change, 0 to destroy.



- ![Image12](https://github.com/user-attachments/assets/9dcd8191-1bb4-4573-b0e5-66afcae78288)
- ![Image13](https://github.com/user-attachments/assets/7e192dc0-da6c-46dd-bd3f-e1840b3a9b7e)
- ![Image14](https://github.com/user-attachments/assets/2ac11d79-2c24-49cf-a1db-d0362eab5862)


15. Create kubeconfig file using awscli.

            aws eks update-kubecofig --name <cluster_name> --region <cluster_region> --kubeconfig kubeconfig

































































































































