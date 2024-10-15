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



















































































































































