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

- ![packerfmt](https://github.com/user-attachments/assets/919ac00e-dd82-4a66-b762-fcc4b9a0d460)

## Run the packer commands to build AMI for Bastion server, Nginx server and webserver
## For Bastion

           packer build bastion.pkr.hcl

- ![bastion0](https://github.com/user-attachments/assets/106c94a8-20d3-44bb-a2e7-d4fde67e7196)
- ![bastion1](https://github.com/user-attachments/assets/5383e594-6745-4c44-8b73-1d44c62bd77a)

- ![bastion2](https://github.com/user-attachments/assets/3b3fb407-b169-4da9-ac8c-5903de5dd077)

- ![bastion3](https://github.com/user-attachments/assets/d6094903-4da8-41e1-ac8d-ab9b522b24f6)

- ![bastionpacker](https://github.com/user-attachments/assets/bb565426-acd3-46bf-ba80-e8b6c29606df)


           

## For Nginx

           packer build nginx.pkr.hcl


- ![packerNgnnix](https://github.com/user-attachments/assets/61d48528-9ec0-4585-a386-469424c8ae87)
- ![ngnixpacker](https://github.com/user-attachments/assets/6369b82a-53be-4edb-a85b-09d71751ad70)


## For Webservers

     packer build web.pkr.hcl

- ![wepacker1](https://github.com/user-attachments/assets/88240353-caea-4746-8066-2369eea04bba)
- ![webserverpacker](https://github.com/user-attachments/assets/7655e584-8452-4d11-aa41-58e5ee49a648)


## For Ubuntu (Jenkins, Artifactory and sonarqube Server)

      packer build ubuntu.pkr.hcl

- ![packerubuntu](https://github.com/user-attachments/assets/9b9c500c-66c9-4353-b27e-8a4da930cffb)
- ![packerubuntu1](https://github.com/user-attachments/assets/cd96475d-a810-498d-9c4e-4129967780ab)

**The new AMI's from the packer build in the terraform script**
- ![AMIs](https://github.com/user-attachments/assets/8034b35c-9299-47f1-8409-5ec3403a1e65)
In the terraform director, update the terraform.auto.tfvars with the new AMIs IDs built with packer which terraform will use to provision Bastion, Nginx, Tooling and Wordpress server

- ![tfvars](https://github.com/user-attachments/assets/8970b9bc-62db-4855-90b7-e924af3907ce)

## 6. Run terraform plan and terraform apply from web console
Switch to Runs tab and click on Queue plan manualy button.

If planning has been successfull, you can proceed and confirm Apply - press Confirm and apply, provide a comment and Confirm plan

- ![Screenshot from 2024-11-06 15-48-18](https://github.com/user-attachments/assets/22a498aa-0fbd-4de2-9b4d-4e9b94811a4c)
- ![tttta](https://github.com/user-attachments/assets/4437aec5-e1d6-48ce-8de3-09637547d6f4)
- ![tttapp](https://github.com/user-attachments/assets/4a4011f9-c8bd-44f7-ac1d-70dd76a11d2b)

![targettt](https://github.com/user-attachments/assets/3f1a00f0-6941-4c02-b2b5-de2010c7040d)

- ![lb](https://github.com/user-attachments/assets/5c1bee57-e264-423b-808c-ea418d7a1e2a)

- ![kvpc](https://github.com/user-attachments/assets/6ef36aa4-4b49-4cfe-85b1-7cc45fb9d440)

- ![ksubnet](https://github.com/user-attachments/assets/6cbf2337-108e-454c-9c13-b2c9bda7fa6d)

- ![krtb](https://github.com/user-attachments/assets/5e4ba203-4bfa-47bc-be68-3cecb5e6fb02)

- ![lauchtemp](https://github.com/user-attachments/assets/4cac9ba5-460a-4d1d-b681-24bff0ae6952)

## 7. Test automated terraform plan
By now, you have tried to launch plan and apply manually from Terraform Cloud web console. But since we have an integration with GitHub, the process can be triggered automatically. Try to change something in any of .tf files and look at Runs tab again - plan must be launched automatically, but to apply you still need to approve manually.

Since provisioning of new Cloud resources might incur significant costs. Even though you can configure Auto apply, it is always a good idea to verify your plan results before pushing it to apply to avoid any misconfigurations that can cause 'bill shock'.

Follow the steps below to set up automatic triggers for Terraform plans and apply operations using GitHub and Terraform Cloud:

Configure a GitHub account as a Version Control System (VCS) provider in Terraform Cloud and follow steps
Add a VCS provider

Go to Version Control and click on Change source

- ![Vcs1](https://github.com/user-attachments/assets/a8c8e6ee-ffd9-49ff-819b-f61572960791)

Click on GitHub.com (Custom)

Select the repository

- ![VSC](https://github.com/user-attachments/assets/b84fcb2f-1524-4f7c-9ee1-85d281eda846)

## Make a change to any Terraform configuration file (.tf file)
Security group decription was edited in the variables.tf file and pushed to the repository on github that is linked to our Terraform Cloud workspace.

Check Terraform Cloud
Click on Runs tab in the Terraform Cloud workspace. Notice that a new plan has been automatically triggered as a result of the push.

Note: First, try to approach this project on your own, but if you hit any blocker and could not move forward with the project, refer



## Configuring The Infrastructure With Ansible

- After a successful execution of terraform apply, connect to the bastion server through ssh-agent to run ansible against the infrastructure.
Run this commands to forward the ssh private key to the bastion server.

          eval `ssh-agent -s`
          ssh-add <private-key.pem>
          ssh-add -l
Update the nginx.conf.j2 file to input the internal load balancer dns name generated.

- ![iloadbal](https://github.com/user-attachments/assets/7df016cb-e139-49d0-922c-96e3201f1a0e)

Update the RDS endpoints, Database name, password and username in the setup-db.yml file for both the tooling and wordpress role.
**For Tooling**
- ![toolingdb](https://github.com/user-attachments/assets/9f8d194c-ca4b-45fb-914e-34e951fbbb9f)

**For Wordpress**
- ![wordpressdb](https://github.com/user-attachments/assets/a068b8c9-547e-4324-b7be-7347bf6c7515)

Update the _EFS_ _Access point ID_ for both the _wordpress_ and _tooling_ role in the **main.yml**

**For Tooling**
- ![toolingefs](https://github.com/user-attachments/assets/2fb9824f-61fa-4a21-84de-f90cd8b9b4c7)

**For Wordpress**

- ![wordacess](https://github.com/user-attachments/assets/bcd7a4d5-696f-4425-b9b6-0a07b0012c16)

- Access the bastion server with ssh agent

       ssh -A ec2-user@<bastion-pub-ip>

  Confirm ansible is installed on bastion server

- ![ansibletsest](https://github.com/user-attachments/assets/d44ffdd6-98c9-46c8-9da6-9ec339c3d66f)

- Verify the inventory

      ansible-inventory -i inventory/aws_ec2.yml/ --graph


- ![Ansible host](https://github.com/user-attachments/assets/b2b308e9-f407-4b94-a1e7-1271d73bafdf)

Export the environment variable ANSIBLE_CONFIG to point to the ansible.cfg from the repo and run the ansible-playbook command:

     export ANSIBLE_CONFIG=/home/ec2-user/terraform-cloud/ansible/roles/ansible.cfg
     
     ansible-playbook -i inventory/aws_ec2.yml playbook/site.yml

## Practice Task №1
1. Configure 3 branches in the terraform-cloud repository for dev, test, prod environments
- ![diffbranch](https://github.com/user-attachments/assets/b7852a64-46fb-47ae-a3d8-6ffc2c01d0f0)

2. Make necessary configuration to trigger runs automatically only for dev environment
- Create a workspace each for the 3 environments (i.e, dev, test, prod).

- ![branches](https://github.com/user-attachments/assets/0487a694-6fb9-4ae8-8b1f-29514f6c1592)
  
- Configure Auto-Apply for dev workspace to trigger runs automatically
- Go to the dev workspace in Terraform Cloud > Navigate to Settings > Vsersion Control > Check boxes for Auto Apply

- ![other-workspace](https://github.com/user-attachments/assets/1d818633-1444-4a06-944b-e0adabfd6e07)

3. Create an Email and Slack notifications for certain events (e.g. started plan or errored run) and test it.
Email Notification: In the dev workspace, Go to Settings > Notifications > Add a new notification

- ![devemailnotification](https://github.com/user-attachments/assets/b1665a5a-25bc-4a6e-ac8d-cafc2d690bbd)

The bastion instance type was changed to t3.small in order to test it

This will automatically apply after a successful plan

Confirm notification has bben sent to the provided email address



















































































































