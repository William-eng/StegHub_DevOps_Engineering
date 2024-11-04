# Automate Infrastructure With IaC using Terraform 4 (Terraform Cloud)- 401

By now, we should be pretty comfortable writing Terraform code to provision Cloud infrastructure using Configuration Language (HCL). Terraform is an open-source system, that we installed and
ran a Virtual Machine (VM) that you had to create, maintain and keep up to date. In Cloud world it is quite common to provide a managed version of an open-source software. Managed means that 
you do not have to install, configure and maintain it yourself - you just create an account and use it "as A Service".

## Terraform Cloud
Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

By default, Terraform CLI performs operation on the server whene it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with     
Terraform - you need a consistent remote environment with remote workflow and shared state to run Terraform commands.

Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called remote operations.

## Migrate your .tf codes to Terraform Cloud
Let us explore how we can migrate our codes to Terraform Cloud and manage our AWS infrastructure from there:

1. Create a Terraform Cloud account
Follow this [this link](https://app.terraform.io/public/signup/account) , create a new account, verify your email and you are ready to start

- ![ConfirmedEmail](https://github.com/user-attachments/assets/24bbeddb-26ad-4d9f-81e2-92cc4aca210a)


Most of the features are free, but if you want to explore the difference between free and paid plans - you can check it on [this page](https://www.hashicorp.com/products/terraform/pricing).

2. Create an organization
Select "Start from scratch", choose a name for your organization and create it.

- ![Organisation](https://github.com/user-attachments/assets/c381674c-2003-40b1-972c-ed0f9c809d22)

3. Configure a workspace
Before we begin to configure our workspace - [let's watch this part of the video](https://www.youtube.com/watch?v=m3PlM4erixY&t=287s) to better understand the difference between
 _version control workflow_, _CLI-driven workflow_ and _API-driven workflow_ and other configurations that we are going to implement.

We will use _version control workflow_ as the most common and recommended way to run Terraform commands triggered from our git repository.

Create a new repository in your GitHub and call it _terraform-cloud_, push your Terraform codes developed in the previous projects to the repository.

Choose _version control workflow_ and you will be promped to connect your GitHub account to your workspace - follow the prompt and add your newly created repository to the workspace.

- ![workspace](https://github.com/user-attachments/assets/3a6478c9-ba6e-48f1-8e63-ee044c144553)
- ![VCSWorkspace](https://github.com/user-attachments/assets/782a7a38-64b3-4b40-b44d-626226170025)

## 4. Configure variables
Terraform Cloud supports two types of variables: _environment variables_ and _Terraform variables_. Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.

Set two environment variables: **AWS_ACCESS_KEY_ID** and **AWS_SECRET_ACCESS_KEY**, set the values that you used in the last two projects. These credentials will be used to privision your AWS infrastructure by Terraform Cloud.

- ![Variables](https://github.com/user-attachments/assets/e8ac8c93-683f-4d35-b03b-8bf8a4a96da1)

After you have set these 2 environment variables - your Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

## 5. Now it is time to run our Terrafrom scripts, 
but in our previous project, we talked about using Packer to build our images, and Ansible to configure the infrastructure, so for that we are going to make few changes to our our existing respository from the last project.

The files that would be Addedd is;

- AMI: for building packer images
- Ansible: for Ansible scripts to configure the infrastucture
Before we proceed, we need to ensure we have the following tools installed on our local machine;

[packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli )

- ![Packer1](https://github.com/user-attachments/assets/cfa3cf61-6de1-41a8-adf1-62552412cf85)

- ![Packer2](https://github.com/user-attachments/assets/d6fab105-37a6-4f44-919a-58cc7d6fc889)

- ![Packer3](https://github.com/user-attachments/assets/9bb4301b-a66c-4500-bb29-9db3e993ff36)
  
- ansible
  
- ![Ansible](https://github.com/user-attachments/assets/27ed30ce-aaf0-42a8-a0a3-659cfe571567)

Refer to this [repository]([https://github.com/citadelict/terraform-cloud](https://github.com/StegTechHub/PBL-project-19)) for guidiance on how to refactor your enviroment to meet the new changes above and ensure you go through the README.md file.

Action Plan for this project
Build images using packer

confirm the AMIs in the console

update terrafrom script with new ami IDs generated from packer build

create terraform cloud account and backend

run terraform script

update ansible script with values from teraform output

RDS endpoints for wordpress and tooling
Database name, password and username for wordpress and tooling
Access point ID for wordpress and tooling
Internal load balancee DNS for nginx reverse proxy
run ansible script

check the website

To follow file structure create a new folder and name it AMI. In this folder, create Bastion, Nginx and Webserver (for Tooling and Wordpress) AMI Packer template (bastion.pkr.hcl, nginx.pkr.hcl, ubuntu.pkr.hcl and web.pkr.hcl).

- ![tr1](https://github.com/user-attachments/assets/23281be5-2eb2-4539-b1eb-5728489a55b7) ![Tr2](https://github.com/user-attachments/assets/856359f0-e457-424a-95bb-d81f0cd267d3) ![Tr3](https://github.com/user-attachments/assets/47e8d8d4-c13b-4e3d-a240-95d9bbe78071) ![Tr4](https://github.com/user-attachments/assets/84e6b8fe-3b0b-420e-ade8-071cd8cc5f8e) ![Tr5](https://github.com/user-attachments/assets/fc425b7d-6459-4cfb-b62e-6ae40e592fe6)

Packer template is a JSON or HCL file that defines the configurations for creating an AMI. Each AMI Bastion, Nginx and Web (for Tooling and WordPress) will have its own Packer template, or we can use a single template with multiple builders.

## Create packer template code for each.
To get the source AMI owner, run this command

     aws ec2 describe-images --filters "Name=name,Values=RHEL-9.4.0_HVM-20240605-x86_64-82-Hourly2-GP3" --query "Images[*].{ID:ImageId,Name:Name,Owner:OwnerId}" --output table

Ensure to update Values with the correct ami name

**Output**

- ![imag](https://github.com/user-attachments/assets/91952025-467a-4a0e-af15-eb8b032dfc9e)

## Packer code for bastion
- ![bastionpacker](https://github.com/user-attachments/assets/b3047983-785d-45d9-915c-1c1f4def8147)

## To format a specific Packer configuration file, use the following command

        packer fmt <name>.pkr.hcl
        
        packer fmt bastion.pkr.hcl
        packer fmt nginx.pkr.hcl
        packer fmt ubuntu.pkr.hcl
        packer fmt web.pkr.hcl

## Initialize the Plugins

     packer init bastion.pkr.hcl

## Validate each packer template

    packer validate bastion.pkr.hcl
    packer validate nginx.pkr.hcl
    packer validate ubuntu.pkr.hcl
    packer validate web.pkr.hcl





















































































































































































