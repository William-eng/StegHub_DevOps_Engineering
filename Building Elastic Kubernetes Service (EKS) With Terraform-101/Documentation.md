![image](https://github.com/user-attachments/assets/01c4760b-a4d1-41ce-8523-07e0405776b1)# Building Elastic Kubernetes Service (EKS) With Terraform-101
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


# DEPLOY APPLICATIONS WITH HELM

In Project 22, you experienced the use of manifest files to define and deploy resources like pods, deployments, and services into Kubernetes cluster. Here, you will do the same thing except that it will not be passed through _kubectl_. In the real world, Helm is the most popular tool used to deploy resources into kubernetes. That is because it has a rich set of features that allows deployments to be packaged as a unit. Rather than have multiple YAML files managed individually – which can quickly become messy.

A Helm chart is a definition of the resources that are required to run an application in Kubernetes. Instead of having to think about all of the various deployments/services/volumes/configmaps/ etc that make up your application, you can use a command like

            helm install stable/mysql

and Helm will ensure that all the required resources are installed. In addition, you can tweak the Helm configuration by setting a single variable to a particular value, allowing more or less resources to be deployed. For example, you can enable a slave for MySQL so that it can have read-only replicas.

A Helm chart is made up of several YAML manifests that outline all the resources needed for an application. Helm takes care of creating these resources in Kubernetes when they’re not present and also manages the removal of any outdated resources.

## Lets begin to gradually walk through how to use Helm (Credit – [LINK](https://andrewlock.net/series/deploying-asp-net-core-applications-to-kubernetes/))

1. Parameterising YAML manifests using Helm templates
   
Let’s consider that our Tooling app has been Dockerised into an image called _tooling-app_, and that we wish to deploy with Kubernetes. Without helm, we would create the YAML manifests defining the deployment, service, and ingress, and apply them to our Kubernetes cluster using _kubectl apply_. Initially, our application is version 1, and so the Docker image is tagged as _tooling-app:1.0.0_. A simple deployment manifest might look something like the following:

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: tooling-app-deployment
              labels:
                app: tooling-app
            spec:
              replicas: 3
              strategy: 
                type: RollingUpdate
                rollingUpdate:
                  maxUnavailable: 1
              selector:
                matchLabels:
                  app: tooling-app
              template:
                metadata:
                  labels:
                    app: tooling-app
                spec:
                  containers:
                  - name: tooling-app
                    image: "tooling-app:1.0.0"
                    ports:
                    - containerPort: 80
                    
Now lets imagine you produce another version of your app, version 1.1.0. How do you deploy that? Assuming nothing needs to be changed with the service or ingress, it may be as simple as copying the deployment manifest and replacing the image defined in the spec section. You would then re-apply this manifest to the cluster, and the deployment would be updated, performing a rolling-update.

The main problem with this is that all of the values specific to your application – the labels and the image names etc – are mixed up with the "mechanical" definition of the manifest.

Helm tackles this by splitting the configuration of a chart out from its basic definition. For example, instead of baking the name of your app or the specific container image into the manifest, you can provide those when you install the chart into the cluster.

For example, a simple templated version of the previous deployment might look like the following:

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: {{ .Release.Name }}-deployment
              labels:
                app: "{{ template "name" . }}"
            spec:
              replicas: 3
              strategy: 
                type: RollingUpdate
                rollingUpdate:
                  maxUnavailable: 1
              selector:
                matchLabels:
                  app: "{{ template "name" . }}"
              template:
                metadata:
                  labels:
                    app: "{{ template "name" . }}"
                spec:
                  containers:
                  - name: "{{ template "name" . }}"
                    image: "{{ .Values.image.name }}:{{ .Values.image.tag }}"
                    ports:
                    - containerPort: 80


This example demonstrates a number of features of Helm templates:

The template is based on YAML, with {{ }} mustache syntax defining dynamic sections.
Helm provides various variables that are populated at install time. For example, the {{.Release.Name}} allows you to change the name of the resource at runtime by using the release name. Installing a Helm chart creates a release (this is a Helm concept rather than a Kubernetes concept).
You can define helper methods in external files. The {{template "name"}} call gets a safe name for the app, given the name of the Helm chart (but which can be overridden). By using helper functions, you can reduce the duplication of static values (like tooling-app), and hopefully reduce the risk of typos.

You can manually provide configuration at runtime. The {{.Values.image.name}} value for example is taken from a set of default values, or from values provided when you call helm install. There are many different ways to provide the configuration values needed to install a chart using Helm. Typically, you would use two approaches:

A values.yaml file that is part of the chart itself. This typically provides default values for the configuration, as well as serving as documentation for the various configuration values.

When providing configuration on the command line, you can either supply a file of configuration values using the -f flag. We will see a lot more on this later on.


### Now lets setup Helm and begin to use it.

According to the official documentation [here](https://helm.sh/docs/intro/install/), there are different options to installing Helm. But we will build the source code to create the binary.


1. Download the tar.gz file from the project’s Github release page. Or simply use wget to download version 3.6.3 directly
   
         wget https://github.com/helm/helm/archive/refs/tags/v3.6.3.tar.gz

2. Unpack the tar.gz file

         tar -zxvf v3.6.3.tar.gz 

3. cd into the unpacked directory

         cd helm-3.6.3


4. Build the source code using make utility

         make build
   
5. Helm binary will be in the bin folder. Simply move it to the bin directory on your system. You can check other tools to know where that is. fOr example, check where pwd the utility is being called from by running which pwd. Assuming the output is /usr/local/bin. You can move the helm binary there.


         sudo mv bin/helm /usr/local/bin/

6. Check that Helm is installed
   
            helm version
            
            version.BuildInfo{Version:"v3.6+unreleased", GitCommit:"13f07e8adbc57b0e3f96b42340d6f44bdc8d5016", GitTreeState:"dirty", GoVersion:"go1.15.5"}

- ![Image15](https://github.com/user-attachments/assets/95ecbd54-0291-4705-a318-3041ad59c389)
  
# DEPLOY JENKINS WITH HELM
Before we start developing our own Helm charts, let's utilize publicly available charts to deploy the tools we need.

One of the great features of Helm is that it allows you to deploy applications that are already packaged in a public Helm repository with minimal configuration. A good example of this is **Jenkins**.

To get started, 
1. Visit [Artifact Hub](https://artifacthub.io/packages/search) to find packaged applications as Helm charts.
2. Search for Jenkins
3. Add the repository to Helm, so you can easily download and deploy it.


- ![Image16](https://github.com/user-attachments/assets/f229ed12-0cd8-45f9-8b19-7c7c5e5bbc04)

         helm repo add jenkins https://charts.jenkins.io

4. Update helm repo

         helm repo update

- ![Image17](https://github.com/user-attachments/assets/c1abaab8-e915-432a-a48b-8384f74a127d)

5. Install the chart

            helm install [RELEASE_NAME] jenkins/jenkins --kubeconfig [kubeconfig file]
            helm install jenkins-server jenkins/jenkins --kubeconfig kubeconfig

- ![image](https://github.com/user-attachments/assets/5a012438-c230-42eb-9dc6-9813e38eaeec)
You should see an output like this

            NAME: jenkins
            LAST DEPLOYED: Sun Aug  1 12:38:53 2021
            NAMESPACE: default
            STATUS: deployed
            REVISION: 1
            NOTES:
            1. Get your 'admin' user password by running:
              kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/chart-admin-password && echo
            2. Get the Jenkins URL to visit by running these commands in the same shell:
              echo http://127.0.0.1:8080
              kubectl --namespace default port-forward svc/jenkins 8080:8080
            
            3. Login with the password from step 1 and the username: admin
            4. Configure security realm and authorization strategy
            5. Use Jenkins Configuration as Code by specifying configScripts in your values.yaml file, see documentation: http:///configuration-as-code and examples: https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos
            
            For more information on running Jenkins on Kubernetes, visit:
            https://cloud.google.com/solutions/jenkins-on-container-engine
            
            For more information about Jenkins Configuration as Code, visit:
            https://jenkins.io/projects/jcasc/
            
            NOTE: Consider using a custom image with pre-installed plugins



- ![Image18](https://github.com/user-attachments/assets/76c658a1-d820-406a-899f-02f12f611527)


6. Check the Helm deployment

            helm ls --kubeconfig [kubeconfig file]


Output:
         
         NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
         jenkins default         1               2021-08-01 12:38:53.429471 +0100 BST    deployed        jenkins-3.5.9   2.289.3


- ![Image19](https://github.com/user-attachments/assets/1c7efb35-6bed-4aca-94ba-4fa4acf72997)



**Troubleshooting**

Get the cluster's OIDC provider URL

       aws eks describe-cluster --name tooling-app-eks --region us-west-1 --query "cluster.identity.oidc.issuer" --output text


Create IAM role, granting the **AssumeRoleWithWebIdentity** action.

Copy the following contents to a file that's named aws-ebs-csi-driver-trust-policy.json Replace 111122223333 with your account ID. Replace EXAMPLED539D4633E53DE1B71EXAMPLE and region-code with the values returned in the oidc URL.


            {
              "Version": "2012-10-17",
              "Statement": [
                {
                  "Effect": "Allow",
                  "Principal": {
                    "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
                  },
                  "Action": "sts:AssumeRoleWithWebIdentity",
                  "Condition": {
                    "StringEquals": {
                      "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com",
                      "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
                    }
                  }
                }
              ]
            }




Create the role, name it AmazonEKS_EBS_CSI_DriverRole

            aws iam create-role \
            --role-name AmazonEKS_EBS_CSI_DriverRole \
            --assume-role-policy-document file://"aws-ebs-csi-driver-trust-policy.json"


Install Amazon EBS CSI driver through the Amazon EKS add-on to improve security and reduce amount of work. (Using the eksctl)

         eksctl create addon --name aws-ebs-csi-driver --cluster tooling-app-eks --region us-west-1

- ![Image20](https://github.com/user-attachments/assets/8ef7a90d-1340-4e70-a211-d54d58fab049)

      eksctl utils migrate-to-pod-identity --cluster tooling-app-eks --region us-west-1 --approve

- ![Image21](https://github.com/user-attachments/assets/094ee028-c1ef-4d66-bc4d-cfdfab1eeb7b)


7. Check the pods

            kubectl get pods --kubeconfigo [kubeconfig file]

Output:

      NAME        READY   STATUS    RESTARTS   AGE
      jenkins-0   2/2     Running   0          6m14s

8. Describe the running pod (review the output and try to understand what you see)

         kubectl describe pod jenkins-0 --kubeconfig [kubeconfig file]
9. Check the logs of the running pod

         kubectl logs jenkins-0 --kubeconfig [kubeconfig file]

 - ![Image19](https://github.com/user-attachments/assets/7e623f31-2ea0-4d93-94eb-b76367bbb7c9)
  

You will notice an output with an error

         error: a container name must be specified for pod jenkins-0, choose one of: [jenkins config-reload] or one of the init containers: [init]


This is because the pod has a [Sidecar container](https://www.godaddy.com/forsale/magalix.com?utm_source=TDFS_BINNS2&utm_medium=parkedpages&utm_campaign=x_corp_tdfs-binns2_base&traffic_type=TDFS_BINNS2&traffic_id=binns2&) alongside with the Jenkins container. As you can see fromt he error output, there is a list of containers inside the pod [jenkins config-reload] i.e jenkins and config-reload containers. The job of the config-reload is mainly to help Jenkins to reload its configuration without recreating the pod.

Therefore we need to let kubectl know, which pod we are interested to see its log. Hence, the command will be updated like:


         kubectl logs jenkins-0 -c jenkins --kubeconfig [kubeconfig file]

Now lets avoid calling the [_kubeconfig file_] everytime. Kubectl expects to find the default kubeconfig file in the location _~/.kube/config_. But what if you already have another cluster using that same file? It doesn’t make sense to overwrite it. What you will do is to merge all the kubeconfig files together using a kubectl plugin called [konfig](https://github.com/corneliusweig/konfig) and select whichever one you need to be active.

1. Install a package manager for kubectl called krew so that it will enable you to install plugins to extend the functionality of kubectl. Read more about it [Here](https://github.com/kubernetes-sigs/krew)

- ![Image22](https://github.com/user-attachments/assets/8027facf-5d27-4d6d-bc6c-8fb71781ca83)

3. Install the [konfig plugin](https://github.com/corneliusweig/konfig)

            kubectl krew install konfig

The Event section shows that Kubernetes is waiting for the external provisioner to create the volume, but it’s not happening. This typically means that there might be an issue with the EBS CSI (Container Storage Interface) driver or the AWS EBS service itself.

- 
























