# Migration to the Сloud with containerization (Docker & Docker Compose)1- 101

Until now, we have been using VMs (AWS EC2) in Amazon Virtual Private Cloud (AWS VPC) to deploy your web solutions, and it works well in many cases. We have learned how easy to spin up and configure a new EC2 manually
or with such tools as Terraform and Ansible to automate provisioning and configuration. We have also deployed two different websites on the same VM; this approach is scalable, but to some extent; imagine what if we need 
to deploy many small applications (it can be web front-end, web-backend, processing jobs, monitoring, logging solutions, etc.) and some of the applications will require various OS and runtimes of different versions and
conflicting dependencies - in such case you would need to spin up serves for each group of applications with the exact OS/runtime/dependencies requirements. When it scales out to tens/hundreds and even thousands of 
applications (e.g., when we talk of microservice architecture), this approach becomes very tedious and challenging to maintain.

In this project, we will learn how to solve this problem and begin to practice the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and imply _Docker_.
Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping your app in a container!

## Install Docker and prepare for migration to the Cloud

First, we need to install _Docker Engine_, which is a client-server application that contains:

A server with a long-running daemon process dockerd.
APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
A command-line interface (CLI) client docker.

As we have already learned - unlike a VM, Docker allocated not the whole guest OS for your application, but only isolated minimal part of it - this isolated container has all that your application needs and at the same
time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility 
issue. It is a well-known problem when a developer sends his application to you, you try to deploy it, deployment fails, and the developer replies, "- _It works on my machine!_". With Docker - if the application is 
shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine.

## MySQL in container
Let us start assembling our application from the Database layer - we will use a pre-built MySQL database container, configure it, and make sure it is ready to receive requests from our PHP application.

## Step 1: Pull MySQL Docker Image from Docker Hub Registry

Start by pulling the appropriate **Docker image for MySQL.** You can download a specific version or opt for the latest release, as seen in the following command:

    docker pull mysql/mysql-server:latest

- ![docker-mysql](https://github.com/user-attachments/assets/e43a1494-ef80-4e91-8b97-49a242c4104c)
    

If you are interested in a particular version of MySQL, replace latest with the version number. Visit Docker Hub to check other tags [here](https://hub.docker.com/r/mysql/mysql-cluster/tags)

List the images to check that you have downloaded them successfully:

        docker image ls
- ![dockerls](https://github.com/user-attachments/assets/fd57e561-e757-40c2-a3b3-0130d1b8e349)

  
## Step 2: Deploy the MySQL Container to your Docker Engine

1. Once you have the image, move on to deploying a new MySQL container with:

        docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest

- Replace _<container_name>_ with the name of your choice. If you do not provide a name, Docker will generate a random one
- The -d option instructs Docker to run the container as a service in the background
- Replace _<my-secret-pw>_ with your chosen password
- In the command above, we used the latest version tag. This tag may differ according to the image you downloaded

2. Then, check to see if the MySQL container is running: Assuming the container name specified is mysql-server

         docker ps -a

- ![runniingContainer](https://github.com/user-attachments/assets/407bd48e-3aff-40e2-b9c7-f68bc71b20c1)
You should see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from health: starting to healthy, once the setup
 is complete.

## Step 3: Connecting to the MySQL Docker Container
We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

**Approach 1**

Connecting directly to the container running the MySQL server:

      docker exec -it <container_name> mysql -uroot -p
- ![condocksql](https://github.com/user-attachments/assets/370d151f-5e15-4209-adfc-157f817062a4)

      

Provide the root password when prompted. With that, you have connected the MySQL client to the server.

Finally, change the server root password to protect your database.

- ![passworddockchange](https://github.com/user-attachments/assets/125e6a09-bbb9-4117-b629-802454d0d52d)

**Approach 2**

First, create a network:

        docker network create --subnet=172.18.0.0/24 tooling_app_network 

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of _DRIVER Bridge_. So, also, it is the default network. You can verify this by running the _docker network ls_ command.

But there are use cases where this is necessary. For example, if there is a requirement to control the _cidr_ range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the _--subnet_

For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.

Run the MySQL Server container using the created network.

First, let us create an environment variable to store the root password:

      export MYSQL_PW=<root-secret-password>

Then, pull the image and run the container, all in one command like below:

    docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest 


Flags used

- -d runs the container in detached mode
- --network connects a container to a network
- -h specifies a hostname

If the image is not found locally, it will be downloaded from the registry.

Verify the container is running:

        docker ps -a


As you already know, it is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an SQL script that will create a user we can use to connect remotely.

Create a file and name it _create_user.sql_ and add the below code in the file:

      CREATE USER '<user>'@'%' IDENTIFIED BY '<client-secret-password>';
      GRANT ALL PRIVILEGES ON * . * TO '<user>'@'%';

Run the script:

      docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql
If you see a warning like below, it is acceptable to ignore:

    mysql: [Warning] Using a password on the command line interface can be insecure.

- ![dockernewuser](https://github.com/user-attachments/assets/994160ab-30fb-45b4-aff0-633d71a0fafc)

## Connecting to the MySQL server from a second container running the MySQL client utility
The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.

Run the MySQL Client Container:

        docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <user-created-from-the-SQL-script> -p

Flags used:

- --name gives the container a name
- -it runs in interactive mode and Allocate a pseudo-TTY
- --rm automatically removes the container when it exits
- --network connects a container to a network
- -h a MySQL flag specifying the MySQL server Container hostname
- -u user created from the SQL script
- -p password specified for the user created from the SQL script

- ![connectAppDOcker](https://github.com/user-attachments/assets/77898c16-f481-4483-8459-f95b096629c5)

## Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.

1. Clone the Tooling-app repository from [here](https://github.com/StegTechHub/tooling-02.git)

         git clone https://github.com/StegTechHub/tooling-02.git

2. On your terminal, export the location of the SQL file
   
        export tooling_db_schema=<path-to-tooling-schema-tile>/tooling_db_schema.sql

3. Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.

        docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema

- ![setup1](https://github.com/user-attachments/assets/fce0f8b8-d7b1-444e-8d89-eeef930e734c)

4. Update the db_conn.php file with connection details to the database

        $servername = "mysqlserverhost";
        $username = "<user>";
        $password = "<client-secret-password>";
        $dbname = "toolingdb";

- ![db_php](https://github.com/user-attachments/assets/4e2783d4-3b2a-4509-96da-87e744e37d61)

- Create a .env file in tooling-02/html/.env with connection details to the database.

        MYSQL_IP=mysqlserverhost
        MYSQL_USER=<username>
        MYSQL_PASS=<client-secrete-password>
        MYSQL_DBNAME=toolingdb

- ![Screenshot from 2024-11-08 21-30-13](https://github.com/user-attachments/assets/42bb00c0-dc52-4464-80e1-973e448e3b22)

Flags used:

- MYSQL_IP: mysql ip address "leave as mysqlserverhost"
- MYSQL_USER: mysql username for user exported as environment variable
- MYSQL_PASS: mysql password for the user exported as environment varaible
- MYSQL_DBNAME: mysql databse name "toolingdb"


5. Run the Tooling App
_Containerization_ of an application starts with creation of a file with a special name - _Dockerfile_ (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container. In this project, we will build our container from a pre-created _Dockerfile_, but as a _DevOps_, we must also be able to write _Dockerfiles_.

You can watch this [video](https://www.youtube.com/watch?v=hnxI-K10auY) to get an idea how to create your Dockerfile and build a container from it.

And on this [page](https://docs.docker.com/build/building/best-practices/), you can find official Docker best practices for writing Dockerfiles.

So, let us _containerize_ our _Tooling application_; here is the plan:

- Make sure you have checked out your Tooling repo to your machine with Docker engine
- First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a Dockerfile for this purpose. Explore it and make sure you understand the code inside it.
- Run _docker build_ command
- Launch the container with _docker run_
- Try to access your application via port exposed from a container
  
**Let us begin**:

Ensure you are inside the folder that has the Dockerfile and build your container:

        docker build -t php-app .

In the above command, we specify a parameter -t, so that the image can be tagged tooling"0.0.1 - Also, you have to notice the . at the end. This is important as that tells Docker to locate the _Dockerfile_ in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the _Dockerfile_.
- ![dockerbuild01](https://github.com/user-attachments/assets/c7149f86-6e9d-4649-b9c1-b8299d34e714)

- ![dockerbulid02](https://github.com/user-attachments/assets/a8e728e2-9b26-4a06-ae78-492db76e88d1)

6. Run the container:

        docker run --network tooling_app_network -p 8085:80 -it php-app
   
Let us observe those flags in the command.

We need to specify the --network flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
The -p flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80. You can confirm this with the CMD ["start-apache"] section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

- ![dockerRun](https://github.com/user-attachments/assets/cdcc19ac-d921-408d-9449-f42676915dd7)

**Note**: _You will get an error. But you must troubleshoot this error and fix it. Below is your error message_.

        AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.18.0.3. Set the 'ServerName' directive globally to suppress this message
**Hint**: _You must have faced this error in some of the past projects. It is time to begin to put your skills to good use. Simply do a google search of the error message, and figure out where to update the configuration file to get the error out of your way_.

If everything works, you can open the browser and type http://localhost:8085


## Troubleshooting
Using the ENV instruction in the Dockerfile like this:

        ENV MYSQL_IP=$MYSQL_IP // your running mysql container
        ENV MYSQL_USER=$MYSQL_USER
        ENV MYSQL_PASS=$MYSQL_PASS
        ENV MYSQL_DBNAME=$MYSQL_DBNAME
did not work as intended for dynamically injecting values from .env file at build time. Here’s why:

Build-Time Context: When Docker builds an image, it does not have access to environment variables from the host system or any .env file unless you explicitly pass them in. The variables $MYSQL_IP, $MYSQL_USER, $MYSQL_PASS, and $MYSQL_DBNAME will not be replaced with values from a .env file or the host environment because Docker does not interpret these at build time.

Result of Current ENV Usage: As a result, when we use ENV like ENV MYSQL_IP=$MYSQL_IP, Docker interprets this as setting the environment variable MYSQL_IP to an empty string unless those variables are explicitly passed to Docker at build time using the --build-arg option.

To dynamically pass environment variables from a .env file or shell environment, consider this approache:

### Using --env-file for Runtime Variables
For runtime environment variables, you should pass them when running the container, not at build time. This allows for more flexibility and keeps Docker images more environment-agnostic and is generally the preferred approach for configuration.


Let's Run the container again adding --env-file .env

        docker run --network tooling_app_network --env-file <.env-path> -p 8085:80 -it tooling:0.0.1
        
You will see the login page.


The default email is _test@gmail.com_, the password is _12345_ or you can check users' credentials stored in the toolingdb.user table.

- ![phpsite](https://github.com/user-attachments/assets/1f4140f8-588e-42d8-838e-ac0e63278045)


- ![databaseTooling](https://github.com/user-attachments/assets/0f2bcbcf-2d14-415f-8524-b59c78cb4d57)


## Practice Task №1 - Implement a POC to migrate the PHP-Todo app into a containerized application.

Download php-todo repository from [here](https://github.com/StegTechHub/php-todo)
- ![step01](https://github.com/user-attachments/assets/1b2711d3-8571-4133-9ede-11611da00171)


1. Write a Dockerfile for the TODO app
Dokerfile
- ![docker file](https://github.com/user-attachments/assets/f2ff2dca-e044-4213-828a-73c93429884c)

**Run todo app**

Build the todo app

        docker build -t php-todo:0.0.1 .

- ![dockerlaravel](https://github.com/user-attachments/assets/4cd8babd-d0b6-4cf6-bcaf-709103ef6f4b)

2. Run both database and app on your laptop Docker Engine
Run database container

            docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest
Create a script create_user.sql to create database and user

Create database and user using the script

            docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql
- ![userSQL](https://github.com/user-attachments/assets/7654b51a-1bb8-4a7f-a567-e93dedac2770)


        docker run --network tooling_app_network --rm --name php-todo --env-file .env -p 8090:8000 -it php-todo:0.0.1

- ![dockerrumlaravel](https://github.com/user-attachments/assets/efc71bea-0952-4d39-98c1-d816cdfa2b74)

3. Access the application from the browser
- ![phplocalhost](https://github.com/user-attachments/assets/15b0581c-bbc6-4493-8bcb-aab6824304c7)

### Part 2
1. Create an account in [Docker Hub](https://hub.docker.com/)

- ![dockerhub accnt](https://github.com/user-attachments/assets/fee435df-eec0-4de7-9a05-41d5c1f9331f)

2. Create a new Docker Hub repository
- ![php-todo-dockerhub](https://github.com/user-attachments/assets/afa73464-8e73-4b3c-9a30-d2b77ac5d622)

3. Push the docker images from your PC to the repository
- Sign in to docker and tag the docker image

        docker login
        
        docker tag php-app willywan/php-todo-app:0.0.1
- ![dockerLogin](https://github.com/user-attachments/assets/c8a3f476-b074-4b5d-b501-fc9f61a665c2)

- Push the docker image to the repository created

            docker push willywan/php-todo-app:0.0.1


  - ![dockerpush](https://github.com/user-attachments/assets/2f903f87-900b-481e-84fc-bb2c39a1eb18)
- ![dockpage](https://github.com/user-attachments/assets/2534736b-3501-48da-bfd0-7f43aed575f5)

### Part 3
1. Write a Jenkinsfile that will simulate a Docker Build and a Docker Push to the registry


                        pipeline {
                            agent any
                        
                            environment {
                                DOCKER_REGISTRY = "docker.io"
                                DOCKER_IMAGE = "willywan/php-todo-app"
                            }
                        
                            stages {
                                stage("Initial cleanup") {
                                    steps {
                                        dir("${WORKSPACE}") {
                                            deleteDir()
                                        }
                                    }
                                }
                        
                                stage('Checkout') {
                                    steps {
                                        checkout scm
                                    }
                                }
                        
                                stage('Build Docker Image') {
                                    steps {
                                        script {
                                            def branchName = env.BRANCH_NAME
                                            // Define tagName outside the script block for reuse
                                            env.TAG_NAME = branchName == 'main' ? 'latest' : "${branchName}-0.0.${env.BUILD_NUMBER}"
                        
                                            // Build Docker image
                                            sh """
                                            docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.TAG_NAME} .
                                            """
                                        }
                                    }
                                }
                        
                                stage('Push Docker Image') {
                                    steps {
                                        script {
                                            // Use Jenkins credentials to login to Docker and push the image
                                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                                sh """
                                                echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin ${DOCKER_REGISTRY}
                                                docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.TAG_NAME}
                                                """
                                            }
                                        }
                                    }
                                }
                        
                                stage('Cleanup Docker Images') {
                                    steps {
                                        script {
                                            // Clean up Docker images to save space
                                            sh """
                                            docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.TAG_NAME} || true
                                            """
                                        }
                                    }
                                }
                            }
                        }


- Launch ec2 instance for jenkins

- ![Jenkins-Server](https://github.com/user-attachments/assets/1a8ce4a3-984f-4a93-9fff-bbc73258f794)

2. Install docker on jenkins server
- Set up Docker's apt repository.

                # Add Docker's official GPG key:
                sudo apt-get update
                
                sudo apt-get install ca-certificates curl
                
                sudo install -m 0755 -d /etc/apt/keyrings
                
                sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
                
                sudo chmod a+r /etc/apt/keyrings/docker.asc
                
                # Add the repository to Apt sources:
                echo \
                  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
                  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
                  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
                
                sudo apt-get update
                
                sudo systemctl start docker
                sudo systemctl enable docker



3. Install docker plugins
- Go to Manage Jenkins > Manage Plugins > Available.
- Search for Docker Pipeline and install it.

- ![dockerplugin](https://github.com/user-attachments/assets/b6dc5e04-7478-4fcc-92f3-4748134b6489)


4. Add Docker credentials to Jenkins.
- Go to Jenkins Dashboard > Manage Jenkins > Credentials. Add your Docker username and password and the credential ID (from jenkinsfile) there.

- ![dockercred](https://github.com/user-attachments/assets/ebb019f9-5d2c-4734-90ea-d8709f21773a)
- Go to Manage Jenkins > Tools, Scroll to Docker installations


5. Connect your repo to Jenkins
- Add a webhook to the github repo

- ![wehhook](https://github.com/user-attachments/assets/14d3867c-578a-4b88-a6ad-83311cf906d8)

- Install Blue Ocean plugin and Open it from dashboard
- Select create New pipeline
- Select Github and your Github account
- Select the repo for the pipeline
- Select create pipeline

- ![pipeline](https://github.com/user-attachments/assets/0ced67c1-fc63-4b30-b1f2-cea3ec3ad0bd)

3. Create a multi-branch pipeline
4. Simulate a CI pipeline from a feature and master branch using previously created Jenkinsfile
- For Feature branch

- ![featurebrachdocker](https://github.com/user-attachments/assets/2229c7ba-fba4-4115-992d-83f55d3c67ed)

-  For Main branch
-  ![Mainrun](https://github.com/user-attachments/assets/198fdd23-0787-45b3-aa45-6972514737b2)

5. Ensure that the tagged images from your Jenkinsfile have a prefix that suggests which branch the image was pushed from. For example, feature-0.0.1.
6. Verify that the images pushed from the CI can be found at the registry.
- ![dockerhubmuilit](https://github.com/user-attachments/assets/74ecf42b-70a5-4b16-8320-888c6c95e55c)

## Deployment with Docker Compose
All we have done until now required quite a lot of effort to create an image and launch an application inside it. We should not have to always run Docker commands on the terminal to get our applications up and running. There are solutions that make it easy to write [declarative code](https://en.wikipedia.org/wiki/Declarative_programming) in [YAML](https://en.wikipedia.org/wiki/YAML), and get all the applications and dependencies up and running with minimal effort by launching a single command.

In this section, we will refactor the _Tooling app_ POC so that we can leverage the power of _Docker Compose_.

1. First, install Docker Compose on your workstation from [here](https://docs.docker.com/compose/install/)
With _Docker Desktop_, Docker Compose is now integrated as a _Docker CLI_ plugin, and you use it by typing docker compose instead of docker-compose (for stand alone docker compose tool).

2. Create a file, name it tooling.yaml
3. Begin to write the Docker Compose definitions with YAML syntax. The YAML file is used for defining services, networks, and volumes:

            version: "3.9"
            services:
              tooling_frontend:
                build: .
                ports:
                  - "5000:80"
                volumes:
                  - tooling_frontend:/var/www/html


The YAML file has declarative fields, and it is vital to understand what they are used for.

- version: Is used to specify the version of Docker Compose API that the Docker Compose engine will connect to. This field is optional from docker compose version v1.27.0.

service: A service definition contains a configuration that is applied to each container started for that service. In the snippet above, the only service listed there is tooling_frontend. So, every other field under the tooling_frontend service will execute some commands that relate only to that service. Therefore, all the below-listed fields relate to the tooling_frontend service.

- build

- port

- volumes

- links

- You can visit the site [here](https://www.balena.io/docs/reference/supervisor/docker-compose/) to find all the fields and read about each one that currently matters to you.

You may also go directly to the official documentation site to read about each field [here](https://docs.docker.com/compose/compose-file/compose-file-v3/).

Let us fill up the entire file and test our application:

                version: "3.9"
                services:
                  tooling_frontend:
                    build: .
                    ports:
                      - "5001:80"
                    volumes:
                      - tooling_frontend:/var/www/html
                    links:
                      - db
                  db:
                    image: mysql:5.7
                    restart: always
                    environment:
                      MYSQL_DATABASE: <The database name required by Tooling app >
                      MYSQL_USER: <The user required by Tooling app >
                      MYSQL_PASSWORD: <The password required by Tooling app >
                      MYSQL_RANDOM_ROOT_PASSWORD: '1'
                    volumes:
                      - db:/var/lib/mysql
                volumes:
                  tooling_frontend:
                  db:
- ![toolingyml](https://github.com/user-attachments/assets/d0931bf9-85a4-4d03-ac9f-40c27b9bb889)

- Run the command to start the containers

        docker compose -f tooling.yml  up -d


- ![dockcompoupd](https://github.com/user-attachments/assets/eda20e67-6988-4871-88bd-f786f343f1b3)

- ![pageshow](https://github.com/user-attachments/assets/0a614375-d8d9-4735-8952-874e0992ebde)

- ![runningps](https://github.com/user-attachments/assets/10f848b2-1d91-4048-9557-9a5de5f462b5)

Verify that the compose is in the running status:

        docker compose ls


- ![Screenshot from 2024-11-11 00-44-58](https://github.com/user-attachments/assets/177dc628-fd21-4ead-8488-234c13ebda45)

## Practice Task №2 - Complete Continous Integration With A Test Stage

1. Document your understanding of all the fields specified in the Docker Compose file tooling.yaml
The provided Docker Compose file defines two services: tooling_frontend and db. The tooling_frontend service builds its image from the current directory (.), maps port 5001 on the host to port 80 inside the container, and mounts a volume (tooling_frontend) to persist data at /var/www/html. It also links to the db service and connects to the custom network dbnet. The db service uses the official mysql:5.7 image, with environment variables to set up a MySQL database (tooling), user (Willie), and root password (Password123). The db service also mounts a volume (db) to persist MySQL data at /var/lib/mysql and is part of the dbnet network.

The file specifies that the tooling_frontend depends on the db service, which is set up to restart automatically if it fails. Both services are placed in a custom bridge network (dbnet), allowing them to communicate using their service names. The named volumes tooling_frontend and db ensure that data persists across container restarts. The db container is configured with a random root password, but it uses a fixed password (Password123) for simplicity.

Overall, this configuration sets up a frontend service with a persistent volume for web content and a MySQL database with a persistent volume for database storage. It also defines network communication between the two services using a custom bridge network. The services are linked to ensure proper communication, and automatic restarts are enabled for the database service. The use of environment variables helps initialize MySQL with necessary settings, and named volumes ensure data persistence across container restarts.

2. Update your Jenkinsfile with a test stage before pushing the image to the registry.
See repository [here](https://github.com/William-eng/tooling)
3. What you will be testing here is to ensure that the tooling site http endpoint is able to return status code 200. Any other code will be determined a stage failure.
- ![tooling ran](https://github.com/user-attachments/assets/f5aea01a-346d-43bf-af8d-240d57b9405d)

.4.  Implement a similar pipeline for the PHP-todo app.

- ![Screenshot from 2024-11-11 01-25-24](https://github.com/user-attachments/assets/ded399d1-0bf3-4a34-9f24-d43b503e950c)



5. Ensure that both pipelines have a clean-up stage where all the images are deleted on the Jenkins server.
Confirm the tooling site image in the registry
- ![tooling image](https://github.com/user-attachments/assets/9f133ee9-7bf2-4973-a5a4-511f418df45f)
## Conclusion
We have migrated our application running on virtual machines into the Cloud with containerization.

In the next project, we will expand our skills further into more advanced use cases and technologies.
























































































