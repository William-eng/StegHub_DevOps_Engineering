# Configure Ansible configuration repo For Jenkins Deployment 
This is project is continuation from my ansible dynamic assignment projects. I would be using a copy of the ansible-config-mgt repo which I have renamed ansible-configuration

## STEP ONE: Spinning up our and Connecting to Our Jenkins Server
We'll Start by spinning up our stopped AWS EC2 instance with a Ubuntu OS with Jenkins set up.
- ![SpinupJenkins](https://github.com/user-attachments/assets/8ca88b8e-b9cc-4563-884c-f3867d3a49ec)
## STEP TWO : Installing Blue-Ocean Plugin
To make managing our Jenkins pipelines easier and more interactive UI, we will install the Blue Ocean plugin. This plugin offers a user-friendly and visually appealing interface, 
helping us quickly understand the status of your continuous delivery pipelines.
To do this , we'll follow the steps below :

- Go to manage jenkins > manage plugins > available
- Search for BLUE OCEAN PLUGIN and install
- ![blueOcean](https://github.com/user-attachments/assets/9be29713-6c92-4699-95ff-3f257621d00c)

## STEP THREE: Configuring the Blue Ocean plugin pipeline with our github repo
To do this, we'll follow the step below:
- Open the blue oceans plugin and create a new pipeline
- Select github
- Connect github with jenkins using your github personal access token
- - ![connecttoGithub](https://github.com/user-attachments/assets/4af122c4-298b-4215-852a-ce7756d4e393)
- Select the repository
- Create the pipeline
- ![create-pipeline](https://github.com/user-attachments/assets/70f0650d-88cc-4dec-9d69-037a28909265)

At this point we may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give us some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.
- ![nojenkinsfile](https://github.com/user-attachments/assets/4eedc2b4-c287-48e1-92df-3b5a6b2518d7)

## Let us create our Jenkinsfile
Inside the Ansible project, we will create a new directory deploy and start a new file Jenkinsfile inside the directory.

and Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

    pipeline {
        agent any
    
    
      stages {
        stage('Build') {
          steps {
            script {
              sh 'echo "Building Stage"'
            }
          }
        }
        }
    }
- ![addjenkins](https://github.com/user-attachments/assets/650c1229-dc4f-4fb6-9bec-d6e175e2c5c0)
    
Now let's go back into the Ansible pipeline in Jenkins, and select configure
- ![jenkinsconfigure](https://github.com/user-attachments/assets/8a273d3a-45a5-4e48-babb-3a551475eed0)

we then Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile
- ![addJenkinspath](https://github.com/user-attachments/assets/a11dbe34-214e-4192-b79d-813ba809f3a6)


Save and go Back to the pipeline again, this time click "Build now"
- ![build](https://github.com/user-attachments/assets/59c5d71f-62f8-42d0-bb16-ba1dbc0aa9fa)


This will trigger a build and we will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

1. Click on Blue Ocean 
2. Select your project
3. Click on the play button against the branch 
Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

Let us see this in action.

Create a new git branch and name it feature/jenkinspipeline-stages
Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.

      pipeline {
        agent any
    
      stages {
        stage('Build') {
          steps {
            script {
              sh 'echo "Building Stage"'
            }
          }
        }
    
        stage('Test') {
          steps {
            script {
              sh 'echo "Testing Stage"'
            }
          }
        }
        }
    }
- ![newbranch](https://github.com/user-attachments/assets/3be6f1ba-ead3-4948-b84b-eae23f933fdc)

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.
- Click on the "Administration" button 
- Navigate to the Ansible project and click on "Scan repository now" 
- Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too. 
- In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch. 
- ![featurebranch](https://github.com/user-attachments/assets/3457b33e-707c-4a37-a7ff-e5db07d40035)

### additional Tasks to perform tp better understand the whole process.

- Let's create a pull request to merge the latest code into the main branch, after merging the PR, go back into your terminal and switch into the main branch.Pull the latest change.
- ![mainupdate](https://github.com/user-attachments/assets/191296bf-c8ee-4f97-8bec-d96bcb78ee47)

- Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
  i. Package
  ii. Deploy
  iii. Clean up

      pipeline {
          agent any
      
          stages {
              stage('Build') {
                  steps {
                      script {
                          sh 'echo "Building Stage"'
                      }
                  }
              }
      
              stage('Test') {
                  steps {
                      script {
                          sh 'echo "Testing Stage"'
                      }
                  }
              }
      
              stage('Package') {
                  steps {
                      script {
                          sh 'echo "Packaging Stage"'
                      }
                  }
              }
      
              stage('Deploy') {
                  steps {
                      script {
                          sh 'echo "Deploying Stage"'
                      }
                  }
              }
      
              stage('Clean up') {
                  steps {
                      script {
                          sh 'echo "Cleaning Up Stage"'
                      }
                  }
              }
          }
      }
- ![newbran](https://github.com/user-attachments/assets/a5f6e0b5-433f-47ee-9c63-ad744d1de86a)


## STEP FOUR : Running Ansible Playbook from Jenkins
Now that we have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

  ### 1. Installing Ansible on Jenkins
   -       sudo apt update && sudo apt upgrade -y
           sudo apt install ansible -y
  
  ### 2. Installing Ansible plugin in Jenkins UI
   - On the dashboard page, click on Manage Jenkins > Manage plugins > Under Available type in ansible and install without restart      
  - ![ansible](https://github.com/user-attachments/assets/ef4af4ed-1819-49ec-962a-f06ad57ad6a8)

  ### 3. Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

- Click on Dashboard > Manage Jenkins > Tools > Add Ansible. Add a name and the path ansible is installed on the jenkins server.
- To get the ansible path on the jnekins server, run :

      which ansible

 - ![ansibletoolconfig](https://github.com/user-attachments/assets/3c7a6726-edb8-4322-9e3c-24fd1a9ca080)
- Now, let's delete all we have in our Jenkinsfile and start writing it again. to do this, we can make use of pipeline syntax to ensure we get the exact command for what we intend to achieve. here is how the Jenkinsfile should look eventually .

              pipeline {
                     agent any
                   
                     environment {
                       ANSIBLE_CONFIG = "${WORKSPACE}/deploy/ansible.cfg"
                       ANSIBLE_HOST_KEY_CHECKING = 'False'
                     }
                   
                     stages {
                       stage("Initial cleanup") {
                         steps {
                           dir("${WORKSPACE}") {
                             deleteDir()
                           }
                         }
                       }

                       stage('Checkout SCM') {
                         steps {
                           git branch: 'main', url: 'https://github.com/citadelict/ansibllle-config-mgt.git'
                         }
                       }
                   
                       stage('Prepare Ansible For Execution') {
                         steps {
                           sh 'echo ${WORKSPACE}'
                           sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'
                         }
                       }
                   
                       stage('Test SSH Connections') {
                         steps {
                           script {
                             def hosts = [
                               [group: 'tooling', ip: '172.31.30.46', user: 'ec2-user'],
                               [group: 'tooling', ip: '172.31.25.209', user: 'ec2-user'],
                               [group: 'nginx', ip: '172.31.26.108', user: 'ubuntu'],
                               [group: 'db', ip: '172.31.24.250', user: 'ubuntu']
                             ]
                             for (host in hosts) {
                               sshagent(['private-key']) {
                                 sh "ssh -o StrictHostKeyChecking=no -i /home/ubuntu/.ssh/key.pem ${host.user}@${host.ip} exit"
                               }
                             }
                           }
                         }
                       }
                   
                       stage('Run Ansible playbook') {
                         steps {
                           sshagent(['private-key']) {
                             ansiblePlaybook(
                               become: true,
                               credentialsId: 'private-key',
                               disableHostKeyChecking: true,
                               installation: 'ansible',
                               inventory: "${WORKSPACE}/inventory/dev.yml",
                               playbook: "${WORKSPACE}/playbooks/site.yml"
                             )
                           }
                         }
                       }
                   
                       stage('Clean Workspace after build') {
                         steps {
                           cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
                         }
                       }
                     }
                   }


## Here is what each part of my jenkinsfile does :
- Environment variables are set for the pipeline: ANSIBLE_CONFIG specifies the path to the Ansible configuration file. while - - - ----------------- ANSIBLE_HOST_KEY_CHECKING disables host key checking to avoid interruptions during SSH connections.
- Stage: Initial cleanup : This cleans up the workspace to ensure a fresh environment for the build by deleting all files in the workspace directory.
- Stage: Checkout SCM : This checks out the source code from the specified Git repository, and alos uses git step to clone the repository.
- Stage: Prepare Ansible For Execution : Prepares the Ansible environment by configuring the Ansible roles path by printing the workspace path, and modifying the Ansible configuration file to add the roles path.
- Stage: Test SSH Connections : Verifies SSH connectivity to each server.
- Stage: Run Ansible playbook : Executes the Ansible playbook. : - Uses the sshagent step to ensure the SSH key is available for Ansible. - Runs the ansiblePlaybook step with the specified parameters . #### To ensure jenkins properly connects to all servers, you will need to install another plugin known as ssh agent , after that, go to manage jenkins > credentials > global > add credentials , usee ssh username and password , fill out the neccesary details and save.
### Now back to your inventory/dev.yml , update the inventory with thier respective servers private ip address
- ![dev yml](https://github.com/user-attachments/assets/a9c09e68-32a6-4535-9f93-08ad5c70c8b7)

### Update the ansible playbook in playbooks/site.yml for the tooling web app deployment. Click on Build Now.
- ![site yml](https://github.com/user-attachments/assets/25abb5e4-49fa-4d9a-8dbb-f911236f2219)

- ![successful-build](https://github.com/user-attachments/assets/239305bc-9e4c-492b-92c8-5b779ee62d8d)

## Parameterizing Jenkinsfile For Ansible Deployment
- let's Update our `/inventory/sit.yml file with the code below

                [tooling]
              <SIT-Tooling-Web-Server-Private-IP-Address>
              
              [todo]
              <SIT-Todo-Web-Server-Private-IP-Address>
              
              [nginx]
              <SIT-Nginx-Private-IP-Address>
              
              [db:vars]
              ansible_user=ec2-user
              ansible_python_interpreter=/usr/bin/python
              
              [db]
              <SIT-DB-Server-Private-IP-Address>

- There are always several environments that need configuration, such as CI, site, and pentest environments etc. To manage and run these environments dynamically, we need to update the Jenkinsfile.

                 parameters {
          string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
        }
- In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}

From now on, each time we hit on execute, it will expect an input.

- ![image-78-1024x476](https://github.com/user-attachments/assets/a0004491-1dd7-4b42-aed9-e18288662c4a)

Notice that the default value loads up, but we can now specify which environment we want to deploy the configuration to. Simply type sit and hit Run

- ![buildparamet](https://github.com/user-attachments/assets/1175da7d-26cf-4351-9f98-d40f0a7e0774)



- update the jenkins file to included the ansible tags before it runs playbook
- Click on build with parameters and update the inventory field to sit and the the ansible_tags to webserver

        string(name: 'ansible_tags', defaultValue: 'webserver', description: 'Tags for the Ansible playbook')
- ![updateWEBSER](https://github.com/user-attachments/assets/a78a4b6f-6c75-4ad4-bcfa-534a725e45dc)

## STEP FIVE: CI/CD Pipeline for TODO application
We already have tooling website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). 

### Phase 1 - Prepare Jenkins
Let's Fork the repository below into your GitHub account

    https://github.com/StegTechHub/php-todo.git
- On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)
  
          sudo apt update
          sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}
          php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
          sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
          php -r "unlink('composer-setup.php');"
          php -v
          composer -v

  - ![composer](https://github.com/user-attachments/assets/f4779c20-812c-4364-ba44-a0fe7968f299)
  
### - Install the required jenkins plugin, which is plot and Artifactory plugins
- Plot Plugin Installation : We will use plot plugin to display tests reports, and code coverage information.
- ![plotplugin](https://github.com/user-attachments/assets/78deacb6-e6ef-49c7-bdac-63f617844671)

-  Artifactory Plugin Installation : The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.
-  ![Artifactoryplugin](https://github.com/user-attachments/assets/f60954e0-e7e1-4d21-aa33-3479e8ac49e8)

  ## Phase 2 – Set up Ansible roles for artifactory
- Create roles to install artifactory just the same way we set up apache, mysql and nginx in the previous project.
- ![artifactory](https://github.com/user-attachments/assets/b90d78bd-acbc-4623-9a52-62faf9acd1de)

and add the code below to static-assignments/artifactory.yml

    ---
    - hosts: artifactory
      roles:
         - artifactory
      become: true

- update the site.yml with

                    ---
                    - name: Include dynamic variables
                      hosts: all
                      become: yes
                      tasks:
                        - include_vars: ../dynamic-assignments/env-vars.yml
                          tags:
                            - always
                    - import_playbook: ../static-assignments/artifactory.yml
                      tags: 
                        - artifactory

Run the playbook against the inventory/ci.yml

    # [jenkins]
    # <Jenkins-Private-IP-Address>
    
    # [nginx]
    # <Nginx-Private-IP-Address>
    
    # [sonarqube]
    # 172.31.20.50 ansible_ssh_user=ubuntu
    
     [artifactory]
    172.31.30.88 ansible_ssh_user='ubuntu'


- ![updateartifactory](https://github.com/user-attachments/assets/f69ffdb8-b799-402b-a210-8ac97c8a8c19)
- ![upodateartifactory](https://github.com/user-attachments/assets/e617030c-9f9d-44f5-bc2a-6579c9adfb03)

- Configure Artifactory plugin by going to manage jenkins > system configurations, scroll down to jfrog and click on add instance
- Input the ID, artifactory url , username and password
- Click on test connection to test your url




- Visit your <your-artifactory-ip-address:8081
- Sign in using the default artifactory credentials : admin and password

- ![articatorytest](https://github.com/user-attachments/assets/6a6dd681-6879-43ca-9393-75ede3026db5)
- Create a local repository and call it _todo-dev-local_, set the repository type to generic
- ![newartifactoryrepo](https://github.com/user-attachments/assets/24c9f4cc-2d83-4ae9-b8a9-cc00d99fdce8)

- Update the database configuration in roles/mysql/vars/main.yml to create a new database and user for the Todo App. use the details below :

                Create database homestead;
                CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
                GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
- ![Screenshot from 2024-10-22 23-19-11](https://github.com/user-attachments/assets/526c15a1-c56f-4755-a261-3e9aa3c59ca3)


- Create a Multibranch pipeline for the Php Todo App.
- ![multibrach-pipeline](https://github.com/user-attachments/assets/1f9e8296-81f2-4987-8c4a-43ba0fcb3e6a)

- Create a .env.sample file and update it with the credentials to connect the database, use sample the code below :


                       APP_ENV=local
                      APP_DEBUG=true
                      APP_KEY=SomeRandomString
                      APP_URL=http://localhost
                      
                      DB_HOST=172.31.24.250
                      DB_DATABASE=homestead
                      DB_USERNAME=homestead
                      DB_PASSWORD=sePret^i
                      
                      CACHE_DRIVER=file
                      SESSION_DRIVER=file
                      QUEUE_DRIVER=sync
                      
                      REDIS_HOST=127.0.0.1
                      REDIS_PASSWORD=null
                      REDIS_PORT=6379
                      
                      MAIL_DRIVER=smtp
                      MAIL_HOST=mailtrap.io
                      MAIL_PORT=2525
                      MAIL_USERNAME=null
                      MAIL_PASSWORD=null
                      MAIL_ENCRYPTION=null

# install mysql client on jenkins server
                        sudo yum install mysql -y 
- Update Jenkinsfile with proper pipeline configuration
                            pipeline {
                           agent any
                       
                         stages {
                       
                            stage("Initial cleanup") {
                                 steps {
                                   dir("${WORKSPACE}") {
                                     deleteDir()
                                   }
                                 }
                               }
                         
                           stage('Checkout SCM') {
                             steps {
                                   git branch: 'main', url: 'https://github.com/William-eng/php-todo.git'
                             }
                           }
                       
                           stage('Prepare Dependencies') {
                             steps {
                                    sh 'mv .env.sample .env'
                                    sh 'composer install'
                                    sh 'php artisan migrate'
                                    sh 'php artisan db:seed'
                                    sh 'php artisan key:generate'
                             }
                           }
                         }
                       }


- Ensure that all neccesary php extensions are already installed .
- Run the pipeline build , you will notice that the database has been populated with tables using a method in laravel known as migration and seeding.

- ![phpinstall](https://github.com/user-attachments/assets/65f0da50-a2ac-41a7-940c-c67a3a399680)

- Update the Jenkinsfile to include Unit tests step

                         stage('Execute Unit Tests') {
                      steps {
                             sh './vendor/bin/phpunit'
                      }


  
