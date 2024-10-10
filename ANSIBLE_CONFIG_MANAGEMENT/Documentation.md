# Ansible Configuration Management

This Project will make us appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time we will become confident 
with writing code using declarative languages such as YAML.

## Tasks

- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

## Step 1 - Install and Configure Ansible on EC2 Instance
1. We need to Update the Name tag on our Jenkins EC2 Instance to _Jenkins-Ansible_ We will use this server to run playbooks.
2. In our GitHub account, we will also create a new repository and name it _ansible-config-mgt_.

- ![githubrepo](https://github.com/user-attachments/assets/654c95f9-2273-4c03-8095-88aa046f06f9)

3. Install Ansible

        sudo apt update
        
        sudo apt install ansible

  We can Check our Ansible version by running _ansible --version_

- ![ansible-version](https://github.com/user-attachments/assets/b6a41e15-a3f7-45c1-afe5-9f33f2dffc7c)

4. Now we will Configure Jenkins build job to archive your repository content every time you change it - this will solidify our Jenkins configuration skills
5. 



















































































































































