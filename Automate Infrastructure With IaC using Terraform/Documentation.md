# Automate Infrastructure With IaC using Terraform part 1
In project 15, we were able to build AWS infrastructure for 2 websites manually. In this project 16, we will be automating this process using IAC tool known as **Terraform**.

## Prerequisites before you begin writing Terraform code
### 1. Create an IAM user, name it terraform(I named mine Terah), ensure that the user has only programatic access to your AWS account) and grant this user AdministratorAccess permissions. to do this :

- sign in to AWS management console, navigate to IAM ( Identity and Access Management)
- Select Users at the left panel and click on Create user
- Follow the process to create a new user
- To ensure programmatic access ( that is access to aws via AWS CLI ) , select the newly created user and click on the security tab.
- Copy the secret access key and access key ID. Save them in a notepad temporarily.

- ![Terah](https://github.com/user-attachments/assets/26c01cc0-d5a6-4ae5-9eda-e5157c0f69b9)

### 2. Install AWS CLI for Ubuntu 24.04

      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install

      # Verify the Installation

      aws --version

- ![awscli](https://github.com/user-attachments/assets/19413d8e-1280-4980-a5d4-d3626f4bda82)


### 3. Configure AWS CLI in the terminal

      aws configure
      
Follow the prompt, by inputing your access key id, access key and region and press enter.

### 4. Install Boto3 (Boto3 is a AWS SDK for Python, which allows developers to write software that uses services like Amazon S3 and Amazon EC2. Here’s how to install Boto3 in your Python environment.) To install Boto3, you will need to create a virtual environment ( this is because my OS is ubuntu 24.04 )

      # Upgrade pip
      python3 -m pip install --upgrade pip
      
      # Install latest Boto3 version
      pip install boto3
      
      # Install Boto3 specific version
      pip install boto3==1.28.60
      
      # Install Boto3 without administrative privileges
      pip install boto3 --user
      
      # After installation, you can verify that Boto3 is installed correctly by running:
      python -c "import boto3; print(boto3.__version__)"
      
- ![boto3](https://github.com/user-attachments/assets/eb0e0d25-ec2b-4e09-ad4f-b060f3314081)

### 5. Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures.

This state is stored by default in a local file named _"terraform.tfstate"_, but it can also be stored remotely, which works better in a team environment.

### 6. Create a S3 bucket resource to store terraform state remotely.  
-We can create it on the console or using the sdk we just install
- Enter a Bucket name, AWS Region, Enable Bucket Versioning, Add a Tag and click create bucket.

            aws s3api create-bucket --bucket ktrontech-dev-terraform-bucket && \
            aws s3api put-bucket-tagging --bucket ktrontech-dev-terraform-bucket --tagging 'TagSet=[{Key=Owner,Value=Ktrontech}]' && \
            aws s3api put-bucket-versioning --bucket ktrontech-dev-terraform-bucket --versioning-configuration Status=Enabled

- ![creatings3bucket](https://github.com/user-attachments/assets/0d79c6e3-4754-4257-b9c9-1d652d2b6a94)

- ![awss3](https://github.com/user-attachments/assets/b46a60d2-2b76-4a90-8374-217a8166c951)

## Install terraform
To install terraform on Ubuntu 24.04, follow the steps below :

            # Ensure that your system is up to date and you have installed the gnupg, software-properties-common, and curl packages installed. 
            sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
            
            # Install the HashiCorp GPG key.
            wget -O- https://apt.releases.hashicorp.com/gpg | \
            gpg --dearmor | \
            sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
            
            # Verify the key's fingerprint.
            gpg --no-default-keyring \
            --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
            --fingerprint

            # The lsb_release -cs command finds the distribution release codename for your current system, such as buster, groovy, or sid.

            echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
            https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
            sudo tee /etc/apt/sources.list.d/hashicorp.list

            # Download the package information from HashiCorp.
            sudo apt update

            # Install Terraform from the new repository.
            sudo apt-get install terraform


- ![teraform1](https://github.com/user-attachments/assets/adfa6846-78e3-4cfc-bf3d-5f9398f76cb6)

### Confirm installation

- ![teraform1](https://github.com/user-attachments/assets/be44817f-8a91-463b-a5bf-c112bc15dce1)

    terraform --version
- ![nstalledTerraform](https://github.com/user-attachments/assets/b8e6570c-bf3e-4c4c-ace2-e659ae170e32)
    
# VPC | Subnets | Security Groups
Let us create a directory structure

1. Open your Visual Studio Code and create a folder called PBL
2. Create a file in the folder, name it main.tf
- ![main tf](https://github.com/user-attachments/assets/dec07bb6-2783-40a0-9a22-dcd3b56495ef)

# VPC resource section
3. Add AWS as a provider, and a resource to create a VPC in the main.tf file. The provider block informs Terraform that we intend to build infrastructure within AWS.
- Resource block will create a VPC.


      terraform {
        required_providers {
          aws = {
            source  = "hashicorp/aws"
            version = "~> 5.0"
          }
        }
      }
      provider "aws" {
        region = "us-east-1"
      }
      
      # Create VPC
      resource "aws_vpc" "main" {
        cidr_block                     = "10.0.0.0/16"
        enable_dns_support             = "true"
        enable_dns_hostnames           = "true"
        tags = {
            Name = "ktrontechVPC"
            }
      }


Note: You can change the configuration above to create your VPC in other region that is closer to you. The same applies to all configuration snippets that will follow.

4. The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by providers and provisioners. At this stage, we only have provider in our main.tffile. So, Terraform will just download plugin for AWS provider. Navigate to the PBL folder

- ![Dependencies](https://github.com/user-attachments/assets/91c8fe6b-efc0-417d-a7e2-dd04564edecb)
5. Let's verify what terraform intends to create ,

      terraform plan
- ![terraformPlan](https://github.com/user-attachments/assets/1e07c42e-e332-4b05-9dc7-6d05cea7eb21)
### Observations:

1. A new file is created terraform.tfstate This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.

2. If you also observed closely, you would realise that another file gets created during planning and apply. But this file gets deleted immediately. terraform.tfstate.lock.info This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same - it allows to avoid duplicates and conflicts.
Its content is usually like this. (We will discuss more about this later)

                {
                    "ID":"e5e5ad0e-9cc5-7af1-3547-77bb3ee0958b",
                    "Operation":"OperationTypePlan","Info":"",
                    "Who":"dare@Dare","Version":"0.13.4",
                    "Created":"2020-10-28T19:19:28.261312Z",
                    "Path":"terraform.tfstate"
                }
It is a json format file that stores information about a user: user's ID, what operation he/she is doing, timestamp, and location of the state file.
   
## Subnets resource section
According to our architectural design, 6 subnets are required:

2 public subnets
2 private subnets for webservers
2 private subnets for data layer
Let us create the first 2 public subnets.Add below configuration to the main.tf file:

            # Create public subnets1
                resource "aws_subnet" "public1" {
                vpc_id                     = aws_vpc.main.id
                cidr_block                 = "172.16.0.0/24"
                map_public_ip_on_launch    = true
                availability_zone          = "eu-central-1a"
                tags = {
                      Name = "ktrontech-publicsubnet1"
                    }
            
            }
            
            # Create public subnet2
                resource "aws_subnet" "public2" {
                vpc_id                     = aws_vpc.main.id
                cidr_block                 = "172.16.1.0/24"
                map_public_ip_on_launch    = true
                availability_zone          = "eu-central-1b"
                tags = {
                      Name = "ktrontech-publicsubnet2"
                 }
            }


7. Run _terraform plan_ to check the intending infrusture and terraform apply to create the infrastructure.

- ![terraformApply](https://github.com/user-attachments/assets/32628b86-fa89-4cda-b50a-e1bee035e0dc)

- ![VPC](https://github.com/user-attachments/assets/862e3463-f4e5-413d-83bf-b98869434b68)

- ![subnetvpc](https://github.com/user-attachments/assets/1debfd49-cb8d-4927-b87d-292d0ef1af9e)

## Observations:

- **Hard coded values**: Remember our best practice hint from the beginning? Both the availability_zone and cidr_block arguments are hard coded. We should always endeavour to make our work dynamic.
- **Multiple Resource Blocks**: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a count argument.
Now let us improve our code by refactoring it.

First, destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, DO NOT DESTROY an infrastructure that has been deployed to production.

To destroy whatever has been created run terraform destroy command, and type yes after evaluating the plan.

- ![Tdestroy](https://github.com/user-attachments/assets/3ca7db33-9774-46c7-a19b-14068f7b4491)
- ![Tdestroy2](https://github.com/user-attachments/assets/85f60a5a-cab4-4cdf-86b4-76b397e07108)

## Fixing The Problems By Code Refactoring
As stated ealier, while the code above worked fine, the process is inefficient and not dynamic. Some o :f the problems we will be solving includes the following

- Fixing Hard Coded Values
- Fixing multiple resource blocks
- Let’s make cidr_block dynamic
- 
### Fixing the Hard Coded Values

We will introduce variables, and remove hard coding.

- Starting with the provider block, declare a variable named region, give it a default value (if you don't declare a default value, you will be prompted each time you run terraform plan/apply), and update the provider section by referring to the declared variable.

          variable "region" {
          default = "eu-central-1"
               }
      
          provider "aws" {
              region = var.region
          }
- Do the same to cidr value in the vpc block, and all the other arguments.

                   variable "vpc_cidr" {
                default = "172.16.0.0/16"
                 }
            
            
                variable "enable_dns_support" {
                    default = "true"
                }
            
            
                variable "enable_dns_hostnames" {
                    default ="true" 
                }
            
                variable "preferred_number_of_public_subnets" {
                default = 2
                }
  
            variable "enable_classiclink" {
                    default = "false"
                    }
               variable "enable_classiclink_dns_support" {
                    default = "false"
                    }
  
                # Create VPC
                resource "aws_vpc" "main" {
                cidr_block                     = var.vpc_cidr
                enable_dns_support             = var.enable_dns_support 
                enable_dns_hostnames           = var.enable_dns_support
                enable_classiclink             = var.enable_classiclink
                enable_classiclink_dns_support = var.enable_classiclink
            
                }

### Fixing multiple resource blocks
- Terraform has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 to 4 Availability Zones. With over 20 geographic regions and over 70 AZs served by AWS, it is impossible to keep up with the latest information by hard coding the names of AZs. Hence, we will explore the use of Terraform's Data Sources to fetch information outside of Terraform. In this case, from AWS

- Let us fetch Availability zones from AWS, and replace the hard coded value in the subnet’s availability_zone section.\

          # Get list of availability zones
      
          data "aws_availability_zones" "available" {
          state = "available"
           }

To make use of this new data resource, we will need to introduce a count argument in the subnet block: Something like this.


         # Create public subnet1
          resource "aws_subnet" "public" { 
              count                   = 2
              vpc_id                  = aws_vpc.main.id
              cidr_block              = "172.16.1.0/24"
              map_public_ip_on_launch = true
              availability_zone       = data.aws_availability_zones.available.names[count.index]
      
          }

Let us quickly understand what is going on here.

- The count tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets.
- The data resource will return a list object that contains a list of AZs. Internally, Terraform will receive the data like this

       ["eu-central-1a", "eu-central-1b"]

Each of them is an index, the first one is index 0, while the other is index 1. If the data returned had more than 2 records, then the index numbers would continue to increment.

Therefore, each time Terraform goes into a loop to create a subnet, it must be created in the retrieved AZ from the list. Each loop will need the index number to determine what AZ the subnet will be created. That is why we have data.aws_availability_zones.available.names[count.index] as the value for availability_zone. When the first loop runs, the first index will be 0, therefore the AZ will be eu-central-1a. The pattern will repeat for the second loop.

But we still have a problem. If we run Terraform with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because we still have cidr_block hard coded. The same cidr_block cannot be created twice within the same VPC. So, we have a little more work to do.

 ## Let's make cidr_block dynamic.

We will introduce a function _cidrsubnet()_ to make this happen. It accepts 3 parameters. Let us use it first by updating the configuration, then we will explore its internals.



    # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
        tags = {
          Name = "ktrontech-public-subnet-${count.index}"
        }
    }

A closer look at cidrsubnet - this function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet.

Its parameters are cidrsubnet(prefix, newbits, netnum)

- The prefix parameter must be given in CIDR notation, same as for VPC.
- The newbits parameter is the number of additional bits with which to extend the prefix. For example, if given a prefix ending with /16 and a newbits value of 4, the resulting subnet address will have length /20
- The netnum parameter is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix

You can experiment how this works by entering the terraform console and keep changing the figures to see the output.

- On the terminal, run _terraform console_
- type _cidrsubnet("172.16.0.0/16", 4, 0)_
- Hit enter
- See the output
- Keep change the numbers and see what happens.
- To get out of the console, type exit

- ![console](https://github.com/user-attachments/assets/dcac065f-acfa-4ca7-af98-fa2a71bef2ef)

## The final problem to solve is removing hard coded count value.

If we cannot hard code a value we want, then we will need a way to dynamically provide the value based on some input. Since the data resource returns all the AZs within a region, it makes sense to count the number of AZs returned and pass that number to the count argument.

To do this, we can introuduce length() function, which basically determines the length of a given list, map, or string.

Since data.aws_availability_zones.available.names returns a list like ["eu-central-1a", "eu-central-1b", "eu-central-1c"] we can pass it into a lenght function and get number of the AZs.

length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])

Open up terraform console and try it

- ![console2](https://github.com/user-attachments/assets/18faa9da-3f0c-4b2c-a41e-d1b69c3b8c73)

Now we can simply update the public subnet block like this

      # Create public subnet1
          resource "aws_subnet" "public" { 
              count                   = length(data.aws_availability_zones.available.names)
              vpc_id                  = aws_vpc.main.id
              cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
              map_public_ip_on_launch = true
              availability_zone       = data.aws_availability_zones.available.names[count.index]
      
          }

**Observations:**

What we have now, is sufficient to create the subnet resource required. But if you observe, it is not satisfying our business requirement of just 2 subnets. The length function will return number 3 to the count argument, but what we actually need is 2.
Now, let us fix this.

- Declare a variable to store the desired number of public subnets, and set the default value

        variable "preferred_number_of_public_subnets" {
            default = 2
      }


- Next, update the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function. See how that is presented below.

      # Create public subnets
      resource "aws_subnet" "public" {
        count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
        vpc_id = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
      
      }

Now lets break it down:

- The first part var.preferred_number_of_public_subnets == null checks if the value of the variable is set to null or has some value defined.
- The second part ? and length(data.aws_availability_zones.available.names) means, if the first part is true, then use this. In other words, if preferred number of public subnets is null (Or not known) then set the value to the data returned by lenght function.
- The third part : and var.preferred_number_of_public_subnets means, if the first condition is false, i.e preferred number of public subnets is not null then set the value to whatever is definied in var.preferred_number_of_public_subnets
Now the entire configuration should now look like this

# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = 2
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}


# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}


- ![terraformApplycount2](https://github.com/user-attachments/assets/43978e55-c97a-459d-9432-ae737a5cb523)

- ![ManagementConsolecount2](https://github.com/user-attachments/assets/e16e85b0-0429-46d8-a975-1295021c2741)

**Note**: You should try changing the value of preferred_number_of_public_subnets variable to null and notice how many subnets get created.

- ![countnull2](https://github.com/user-attachments/assets/cd127f67-a0ff-4d66-a311-ed3b48ca71e9)
- ![countnull](https://github.com/user-attachments/assets/62e83c87-8257-4e84-b651-1b7f5de06bed)

## Introducing variables.tf & terraform.tfvars

Instead of havng a long lisf of variables in _main.tf_ file, we can actually make our code a lot more readable and better structured by moving out some parts of the configuration content to other files.

We will put all variable declarations in a separate file
1. And provide non default values to each of them
2. Create a new file and name it variables.tf
3. Copy all the variable declarations into the new file.
4. Create another file, name it terraform.tfvars
5. Set values for each of the variables.

### Main.tf

      # Get list of availability zones
      data "aws_availability_zones" "available" {
      state = "available"
      }
      
      provider "aws" {
        region = var.region
      }
      
      # Create VPC
      resource "aws_vpc" "main" {
        cidr_block                     = var.vpc_cidr
        enable_dns_support             = var.enable_dns_support 
        enable_dns_hostnames           = var.enable_dns_support
        enable_classiclink             = var.enable_classiclink
        enable_classiclink_dns_support = var.enable_classiclink
      
      }
      
      # Create public subnets
      resource "aws_subnet" "public" {
        count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
        vpc_id = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
      }

### variables.tf

      variable "region" {
            default = "eu-central-1"
      }
      
      variable "vpc_cidr" {
          default = "172.16.0.0/16"
      }
      
      variable "enable_dns_support" {
          default = "true"
      }
      
      variable "enable_dns_hostnames" {
          default ="true" 
      }
      
      variable "enable_classiclink" {
          default = "false"
      }
      
      variable "enable_classiclink_dns_support" {
          default = "false"
      }
      
        variable "preferred_number_of_public_subnets" {
            default = null
      }
- ![variable tff](https://github.com/user-attachments/assets/ecfc9c3b-4650-4be6-b4e9-9a4a7891e6c8)

### terraform.tfvars

      region = "eu-central-1"
      
      vpc_cidr = "172.16.0.0/16" 
      
      enable_dns_support = "true" 
      
      enable_dns_hostnames = "true"  
      
      enable_classiclink = "false" 
      
      enable_classiclink_dns_support = "false" 
      
      preferred_number_of_public_subnets = 2

- ![variable tf](https://github.com/user-attachments/assets/fdf530df-9f08-43d6-ac43-3536c595a25d)

You should also have this file structure in the PBL folder.

└── PBL
    ├── main.tf
    ├── terraform.tfstate
    ├── terraform.tfstate.backup
    ├── terraform.tfvars
    └── variables.tf

- ![treee](https://github.com/user-attachments/assets/ff8cf5f8-d58b-43b1-b309-8d171cc9f997)
  
### Run terraform plan and ensure everything works
- ![plan1](https://github.com/user-attachments/assets/5ee56264-0f27-4adf-b526-09036f57b8a3)

![plan2](https://github.com/user-attachments/assets/e0f2db1a-54da-4ccc-96c0-ccb6002f70e0)

![plan3](https://github.com/user-attachments/assets/42cc9cc1-1b0a-41df-b4b7-63e8e99ed962)

### terraform apply
- run _terraform apply_ to create the infrastructure
![apply1](https://github.com/user-attachments/assets/c4e8d7c6-cd00-47fe-9d94-f96dc27f5199)

- ![apply2](https://github.com/user-attachments/assets/030137bf-478b-4533-baac-a26ee131c723)

- ![apply3](https://github.com/user-attachments/assets/92250a32-0b81-487b-bc42-5886b46a8d24)

- ![applysubnet](https://github.com/user-attachments/assets/94866fc7-96ea-4287-991a-257241c43d51)

- ![applyVPC](https://github.com/user-attachments/assets/aa8aa2a0-ad1b-4302-9d06-afb1d71124bb)

### Destroy the Infrastructures
Use the command _terraform destroy_ to terminate all infrastructures

- ![destroy1](https://github.com/user-attachments/assets/08b39154-03e1-491b-9328-5c398b0f916d)

![destroy2](https://github.com/user-attachments/assets/688198a8-a103-4990-b172-369b3cf8398b)

![destroy3](https://github.com/user-attachments/assets/4df41777-8cb9-4536-a2d2-3725ce5e870b)

![destroy4](https://github.com/user-attachments/assets/5855c003-4b83-4352-9376-b8588f816436)
























































































































































