In this project we will introduce dynamic assignments by using include module.

Well, from previous project, we can already tell that static assignments use import Ansible module. The module that enables dynamic assignments is include.

Hence,

      import = Static
      include = Dynamic


## Introducing Dynamic Assignment Into Our structure

In our https://github.com/<your-name>/ansible-config-mgt GitHub repository we will start a new branch and call it _dynamic-assignments_.
Let's Create a new folder, name it _dynamic-assignments_. Then inside this folder, create a new file and name it _env-vars.yml_. We will instruct site.yml to include this playbook later.
For now, let us keep building up the structure.

Our GitHub shall have following structure by now.

    ├── dynamic-assignments
    │   └── env-vars.yml
    ├── inventory
    │   └── dev
        └── stage
        └── uat
        └── prod
    └── playbooks
        └── site.yml
    └── roles (optional folder)
        └──...(optional subfolders & files)
    └── static-assignments
        └── common.yml
- ![tree](https://github.com/user-attachments/assets/d2b70e17-750a-43ef-a0fe-d7154861e131)

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc.,
we will need a way to set values to variables per specific environment.
For this reason, we will now create a folder to keep each environment's variables file. Therefore, let's create a new folder _env-vars_, then for each environment, 
we'll create new YAML files which we will use to set variables.

Our layout should now look like this.

    ├── dynamic-assignments
    │   └── env-vars.yml
    ├── env-vars
        └── dev.yml
        └── stage.yml
        └── uat.yml
        └── prod.yml
    ├── inventory
        └── dev
        └── stage
        └── uat
        └── prod
    ├── playbooks
        └── site.yml
    └── static-assignments
        └── common.yml
        └── webservers.yml

Now we'll paste the instruction below into the env-vars.yml file.

    ---
    - name: collate variables from env specific file, if it exists
      hosts: all
      tasks:
        - name: looping through list of available files
          include_vars: "{{ item }}"
          with_first_found:
            - files:
                - dev.yml
                - stage.yml
                - prod.yml
                - uat.yml
              paths:
                - "{{ playbook_dir }}/../env-vars"
          tags:
            - always


## Let's Notice 3 things to note here:

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8,
the include module is deprecated and variants of include_* must be used. These are:
- include_role
- include_tasks
- include_vars
 In the same version, variants of import were also introduces, such as:
- import_role
- import_tasks

2. We made use of a special variables {{ playbook_dir }} and {{ inventory_file }}. {{ playbook_dir }} will help Ansible to determine the location of the running playbook,
and from there navigate to other path on the filesystem. {{ inventory_file }} on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml
so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default
 values in case an environment specific env file does not exist.

## Update site.yml with dynamic assignments
We'll Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come.)

site.yml should now look like this.

    ---
    - hosts: all
    - name: Include dynamic variables 
      tasks:
      import_playbook: ../static-assignments/common.yml 
      include: ../dynamic-assignments/env-vars.yml
      tags:
        - always
    
    -  hosts: webservers
    - name: Webserver assignment
      import_playbook: ../static-assignments/webservers.yml

## Community Roles
Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

## Download Mysql Ansible Role
We will be using a MySQL role developed by **geerlingguy**.

**Hint**: To preserve our GitHub in actual state after you install a new role - make a commit and push to master your 'ansible-config-mgt' directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it - we will be using Jenkins later for a better purpose.
On Jenkins-Ansible server make sure that git is installed with git --version, then go to 'ansible-config-mgt' directory and run

      git init
      git pull https://github.com/<your-name>/ansible-config-mgt.git
      git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
      git branch roles-feature
      git switch roles-feature

- ![gitini](https://github.com/user-attachments/assets/2d22136f-429b-46b0-bf69-2651675cfcff)
- ![gitswitch](https://github.com/user-attachments/assets/198cb7e2-a402-4750-8b09-8560dd0f7669)


Inside roles directory we'll create our new MySQL role with _ansible-galaxy install geerlingguy.mysql_ and rename the folder to mysql

      mv geerlingguy.mysql/ mysql
      
![glaaxy](https://github.com/user-attachments/assets/ec31c1ff-e076-4276-b0ad-b75b3d640fa2)

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

- ![READMERead](https://github.com/user-attachments/assets/66898669-cf8d-44eb-af7f-67f22b7a52bc)
- ![configfile](https://github.com/user-attachments/assets/57e2fa51-bd91-4217-96a1-dc735609c5c4)

Now it is time to upload the changes into your GitHub:

            git add .
            git commit -m "Commit new role files into GitHub"
            git push --set-upstream origin roles-feature


- ![push](https://github.com/user-attachments/assets/dfa1d846-2b9f-4d7c-9eae-9d113576a2b5)
Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.

## Load Balancer roles

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

- Nginx
- Apache
With our experience on Ansible so far you can:

We can Decide if we  want to develop our own roles, or find available ones from the community
let's Update both s_tatic-assignment_ and _site.yml_ files to refer the roles

## Important Hints:

- Since we cannot use both Nginx and Apache load balancer, we need to add a condition to enable either one - this is where we can make use of variables.
- We will Declare a variable in _defaults/main.yml_ file inside the Nginx and Apache roles. and Name each variables _enable_nginx_lb_ and _enable_apache_lb_ respectively.
- We'll Set both values to false like this _enable_nginx_lb: false_ and _enable_apache_lb: false._
- ![enable](https://github.com/user-attachments/assets/020fd961-87ef-4bfa-ab97-67cb5d600388)

- and Declare another variable in both roles _load_balancer_is_required_ and set its value to _false_ as well
Update both assignment and site.yml files respectively

loadbalancers.yml file

      - hosts: lb
        roles:
          - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
          - { role: apache, when: enable_apache_lb and load_balancer_is_required }
site.yml file

     - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 

Now we can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

We will activate load balancer, and enable nginx by setting these in the respective environment's env-vars file.

      enable_nginx_lb: true
      load_balancer_is_required: true

The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

To test this, we can update inventory for each environment and run Ansible against each environment.



















































































































