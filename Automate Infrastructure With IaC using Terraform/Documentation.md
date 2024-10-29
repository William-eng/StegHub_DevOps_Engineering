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
    



























































































































































































































































