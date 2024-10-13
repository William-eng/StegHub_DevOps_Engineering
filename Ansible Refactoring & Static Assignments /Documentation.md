# Ansible Refactoring & Static Assignments (Imports and Roles)
## Code Refactoring
**Refactoring** is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

## Step 1 - Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job - now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job - we will require Copy Artifact plugin.

1. Let's go to our Jenkins-Ansible server and create a new directory called ansible-config-artifact - we will store there all artifacts after each build.
   
        sudo mkdir /home/ubuntu/ansible-config-artifact
2. We'll now Change permissions to this directory, so Jenkins could save files there -

        sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact
3. Now we go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins
4. Create a new Freestyle project and name it _save_artifacts_.
5. This project will be triggered by completion of our existing ansible project. Configure it accordingly:
- ![image-50-1024x739](https://github.com/user-attachments/assets/99a5411a-0ca8-4553-bbaf-9af168b372ae)
- ![image-51-1024x946](https://github.com/user-attachments/assets/a43b4471-05d4-4c16-8a8b-2d18d2a040ad)
Note: We can configure number of builds to keep in order to save space on the server, for example, one might want to keep only last 2 or 5 build results. we can also make this change to your ansible job.
6. The main idea of save_artifacts project is to save artifacts into _/home/ubuntu/ansible-config-artifact_ directory. To achieve this, we'll create a Build step and choose Copy artifacts from other project, specify ansible as a source project and _/home/ubuntu/ansible-config-artifact_ as a target directory.
- ![image-52-1024x771](https://github.com/user-attachments/assets/5031f796-dafb-4373-a7fb-6725e7c42a33)

7. Let's now Test our set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside main branch).
- ![sucesscopy](https://github.com/user-attachments/assets/b24dc6fa-6514-43ab-95b5-459e467ac2ff)

If both Jenkins jobs have completed one after another - we shall see our files inside _/home/ubuntu/ansible-config-artifact_ directory and it will be updated with every commit to your master branch.

Now our Jenkins pipeline is more neat and clean.

## Step 2 - Refactor Ansible code by importing other playbooks into site.yml
Before we start to refactor the codes, we must ensure that we have pulled down the latest code from master (main) branch, and create a new branch, name it refactor.
DevOps philosophy implies constant iterative improvement for better efficiency - refactoring is one of the techniques that can be used, but we will always have an answer to question "why?". Why do we need to change something if it works well?
In previous project, we wrote all tasks in a single playbook _common.yml_, now it is pretty simple set of instructions for only 2 types of OS, but let's imagine we have many more tasks and we need to apply this playbook to other servers with different requirements. In this case, we will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Our DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use our playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.
1. Within playbooks folder, we'll create a new file and name it site.yml - This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, it will understand more what this means shortly.
2. We 'll Create a new folder in root of the repository and name it _static-assignments_. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of our work. It is not an Ansible specific concept, therefore we can choose how we want to organize our work. We should now see why the folder name has a prefix of static very soon. For now, let's just follow along.
3. Let's Move common.yml file into the newly created static-assignments folder.
4. Inside site.yml file, we'll import common.yml playbook.

         ---
         - hosts: all
         - import_playbook: ../static-assignments/common.yml
The code above uses built in import_playbook Ansible module.

Our folder structure should look like this;

      ├── static-assignments
      │   └── common.yml
      ├── inventory
          └── dev
          └── stage
          └── uat
          └── prod
      └── playbooks
          └── site.yml

5. Run ansible-playbook command against the dev environment
Since we need to apply some tasks to your dev servers and wireshark is already installed - we can go ahead and create another playbook under static-assignments and name it _common-del.yml_. In this playbook, we'll configure deletion of wireshark utility.

         ---
         - name: update web, nfs and db servers
           hosts: webservers, nfs, db
           remote_user: ec2-user
           become: yes
           become_user: root
           tasks:
           - name: delete wireshark
             yum:
               name: wireshark
               state: removed
         
         - name: update LB server
           hosts: lb
           remote_user: ubuntu
           become: yes
           become_user: root
           tasks:
           - name: delete wireshark
             apt:
               name: wireshark-qt
               state: absent
               autoremove: yes
               purge: yes
               autoclean: yes

update site.yml with - _import_playbook: ../static-assignments/common-del.yml_ instead of common.yml and run it against dev servers:

      cd /home/ubuntu/ansible-config-mgt/
      
      ansible-playbook -i inventory/dev.yml playbooks/site.yaml

Make sure that wireshark is deleted on all the servers by running wireshark --version
- ![ansiblerun](https://github.com/user-attachments/assets/8b60c089-5173-425d-a74b-830b44bd9cf4)

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

## Step 3 - Configure UAT Webservers with a role 'Webserver'
We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.
1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly - Web1-UAT and Web2-UAT.

2. To create a role, we must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

There are two ways how we can create this folder structure:

We can Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (we need to create roles directory upfront)

      mkdir roles
      cd roles
      ansible-galaxy init webserver
or we can Create the directory/files structure manually


The entire folder structure should look like below, but if you create it manually - we can skip creating tests, files, and vars or remove them if we used ansible-galaxy

      └── webserver
          ├── README.md
          ├── defaults
          │   └── main.yml
          ├── files
          ├── handlers
          │   └── main.yml
          ├── meta
          │   └── main.yml
          ├── tasks
          │   └── main.yml
          ├── templates
          ├── tests
          │   ├── inventory
          │   └── test.yml
          └── vars
              └── main.yml

After removing unnecessary directories and files, the roles structure should look like this

      └── webserver
          ├── README.md
          ├── defaults
          │   └── main.yml
          ├── handlers
          │   └── main.yml
          ├── meta
          │   └── main.yml
          ├── tasks
          │   └── main.yml
          └── templates

3. Let's now Update our inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers


         [uat-webservers]
         <Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
         <Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
- ![uat-webserver](https://github.com/user-attachments/assets/2fbc1ba4-dac8-4f8c-9d57-d2ab245af2bd)

4. In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.
5. It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

- Install and configure Apache (httpd service)
- Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
- Make sure httpd service is started

our main.yml may consist of following tasks:

      --
      - name: install apache
        become: true
        ansible.builtin.yum:
          name: "httpd"
          state: present
      
      - name: install git
        become: true
        ansible.builtin.yum:
          name: "git"
          state: present
      
      - name: clone a repo
        become: true
        ansible.builtin.git:
          repo: https://github.com/<your-name>/tooling.git
          dest: /var/www/html
          force: yes
      
      - name: copy html content to one level up
        become: true
        command: cp -r /var/www/html/html/ /var/www/
      
      - name: Start service httpd, if not started
        become: true
        ansible.builtin.service:
          name: httpd
          state: started
      
      - name: recursively remove /var/www/html/html/ directory
        become: true
        ansible.builtin.file:
          path: /var/www/html/html
          state: absent
- ![updatemain](https://github.com/user-attachments/assets/e405b76b-3919-4f7f-adbf-a48cc4727c0f)


## Step 4 - Reference 'Webserver' role
Within the static-assignments folder, we will create a new assignment for uat-webservers uat-webservers.yml. This is where we will reference the role.

      ---
      - hosts: uat-webservers
        roles:
           - webserver
We Remember that the entry point to our ansible configuration is the site.yml file. Therefore, we need to refer our uat-webservers.yml role inside site.yml.

So, we should have this in site.yml

      ---
      - hosts: all
      - import_playbook: ../static-assignments/common.yml
      
      - hosts: uat-webservers
      - import_playbook: ../static-assignments/uat-webservers.yml

## Step 5 - Commit & Test

Let's Commit our changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory and see what happens:

      cd /home/ubuntu/ansible-config-mgt
      
      ansible-playbook -i /inventory/uat.yml playbooks/site.yaml

We should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

or

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

















































































































