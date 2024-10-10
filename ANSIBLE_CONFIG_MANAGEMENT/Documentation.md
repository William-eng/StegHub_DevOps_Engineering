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

   - Create a new Freestyle project ansible in Jenkins and point it to your 'ansible-config-mgt' repository.
   - Configure a webhook in GitHub and set the webhook to trigger ansible build
   - Configure a Post-build job to save all (**) files.

5. We will now Test your setup by making some change in README.md file in main branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

        ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
   
- ![sucessartibuild](https://github.com/user-attachments/assets/e2bf8075-92f1-4e7a-95d3-e8d1d380e061)
- ![buildshow](https://github.com/user-attachments/assets/2929554d-788e-4805-96c7-5fc6fd5d920d)


**Note**: Trigger Jenkins project execution only for main (or master) branch.

Now our setup will look like this:
- ![image-46-1024x545](https://github.com/user-attachments/assets/f290bb8a-834b-439b-b641-9108e27f8911)


## Step 2 - Prepare your development environment using Visual Studio Code

1. First part of 'DevOps' is 'Dev', which means you will require to write some codes and we shall have proper tools that will make our coding and debugging comfortable - you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and source-code Editors for different languages with their own advantages and drawbacks, we can choose whichever we are comfortable with, but I recommend one free and universal editor that will fully satisfy your needs - Visual Studio Code (VSC), 
2. After we have successfully installed VSC, configure it to connect to your newly created GitHub repository.
3. We will now Clone down our ansible-config-mgt repo to our Jenkins-Ansible instance

           git clone <ansible-config-mgt repo link>

- ![clonees](https://github.com/user-attachments/assets/e92abfc8-ac06-4d7f-abe6-97df1f2b9705)

## Step 3 - Begin Ansible Development

1. In our ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

**Tip**: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool - include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about - a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)

2. We Checkout the newly created feature branch to our local machine and start building your code and directory structure
3. We will now Create a directory and name it playbooks - it will be used to store all our playbook files.
4. Then we will Create a directory and name it inventory - it will be used to keep our hosts organised.
5. Within the playbooks folder, create our first playbook, and name it common.yml
6. Within the inventory folder, create an inventory file () for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively. These inventory files use .ini languages style to configure Ansible hosts.
   
- ![config](https://github.com/user-attachments/assets/1075a564-de5a-4314-b05c-79ec6ce17971)


## Step 4 - Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

We will Save the below inventory structure in the inventory/dev file to start configuring our development servers. Ensure to replace the IP addresses according to our own setup.

**Note**: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this you can implement the concept of ssh-agent. Now you need to import our key into ssh-agent:

        eval `ssh-agent -s`
        ssh-add <path-to-private-key>
        
Confirm the key has been added with the command below, you should see the name of your key


        ssh-add -l 

Now, we ssh into our Jenkins-Ansible server using ssh-agent

        ssh -A ubuntu@public-ip


We will Update our inventory/dev.yml file with this snippet of code:

        [nfs]
        <NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user
.

        [nfs]
        <NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user
        
        [webservers]
        <Web-Server1-Private-IP-Address> ansible_ssh_user=ec2-user
        <Web-Server2-Private-IP-Address> ansible_ssh_user=ec2-user
        
        [db]
        <Database-Private-IP-Address> ansible_ssh_user=ec2-user 
        
        [lb]
        <Load-Balancer-Private-IP-Address> ansible_ssh_user=ubuntu




















































































































