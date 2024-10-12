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

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.































































































































































