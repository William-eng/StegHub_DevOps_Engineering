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




























































