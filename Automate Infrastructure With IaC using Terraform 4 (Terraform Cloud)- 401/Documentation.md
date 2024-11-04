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









































































































































































































