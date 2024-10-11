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

Now your Jenkins pipeline is more neat and clean.























































































































































































