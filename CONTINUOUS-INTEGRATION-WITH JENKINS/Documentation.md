# Tooling Website deployment automation with Continuous Integration. (Introduction to Jenkins)

We need to configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how our updated architecture will look upon competion of this project:

- ![image-25-1024x603](https://github.com/user-attachments/assets/0dbbc244-0bc2-4a2e-a404-ef6202c67dac)

## STEP ONE: INSTALL JENKINS SERVER
1. We will first Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins-Instance"
2. Install JDK (since Jenkins is a Java-based application)

            sudo apt update
            sudo apt install default-jdk-headless

- ![JavaInstall](https://github.com/user-attachments/assets/c07ae6aa-f5b2-4676-9f19-930603df3a64)

3. Install Jenkins

          wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
          sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
              /etc/apt/sources.list.d/jenkins.list'
          sudo apt update
          sudo apt-get install jenkins

Make sure Jenkins is up and running

        sudo systemctl status jenkins

- ![Jenkrunning](https://github.com/user-attachments/assets/7c624317-ae0b-4cca-9e14-f0177a33ac70)

        
4. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group
- ![8080](https://github.com/user-attachments/assets/e6c0e4ac-5c4a-43f0-bdf3-6df71ff6432e)

5. Perform initial Jenkins setup.
From our browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

We are prompted to provide a default admin password
- ![image-27-1024x388](https://github.com/user-attachments/assets/4d5db605-e695-4dda-b7ec-f32dd32ce008)



We will Retrieve it from our server:

        sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Then we are asked which plugings to install - we will choose suggested plugins.

- ![image-28-1024x481](https://github.com/user-attachments/assets/75dbd8f1-b7ab-4061-a65f-eaa14103d6bc)

Once plugins installation is done - We will create an admin user and will get our Jenkins server address.

The installation is completed!

- ![image-29-1024x680](https://github.com/user-attachments/assets/7caa0e64-72c7-4f80-960c-090b22b1275f)


## STEP TWO: CONFIGURE JENKINS TO RETRIEVE SOURCE CODE FROM GITHUB USING WEBHOOKS
In this part, we will configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a 'build' task to retrieve codes from GitHub and store it locally on Jenkins server.

   1. Enable webhooks in your GitHub repository settings like this:

- ![webhook](https://github.com/user-attachments/assets/c8defafb-9238-4784-abcc-2da0e466a1f2)

   2. On our Jenkins web console, click "New Item" and create a "Freestyle project" like this :
 
- ![newproj](https://github.com/user-attachments/assets/8dc1acd4-4440-470c-b701-7cf80b3ef3fc)

 
- ![image-30-1024x584](https://github.com/user-attachments/assets/0eb883ab-d055-4cbd-a578-14bd4f840886)

 
  To connect our GitHub repository, we will need to provide its URL or copy from the repository itself
  
- ![image-31-1024x498](https://github.com/user-attachments/assets/68045306-0f80-4a60-a808-d738f10767bf)

In configuration of our Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
- ![newproj](https://github.com/user-attachments/assets/a47aa765-6c7e-4a81-aac1-8cb4a3765614)

Save the configuration and let us try to run the build. For now we can only do it manually. Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

- ![buildnow](https://github.com/user-attachments/assets/e30528e8-a6b2-4fff-887e-62c4e512f27f)

We can open the build and check in "Console Output" if it has run successfully.

If so - congratulations! You have just made your very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

- ![gittrigger](https://github.com/user-attachments/assets/949bf3e6-1b5b-4d01-92b6-1ae49933822b)

Configure "Post-build Actions" to archive all the files - files resulted from a build are called "artifacts".

- ![ArchiveActifacts](https://github.com/user-attachments/assets/e0516a7a-c7a7-405c-8a93-f0ed79e127cd)
  
Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.



We have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally

      ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/

- ![ls var](https://github.com/user-attachments/assets/8ff610fe-c19b-47c8-b968-0fd8717c0d3d)


## STEP THREE: Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

 1. Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it
- ![Screenshot from 2024-10-07 13-32-00](https://github.com/user-attachments/assets/3fc293cf-9912-4023-974a-e1c17167e30a)

2. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

a. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
b. Arbitrary name
c. Hostname - can be private IP address of your NFS server
d. Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
e. Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
![Screenshot from 2024-10-07 13-43-37](https://github.com/user-attachments/assets/3a4e7d07-30be-4e2c-8b86-0871962ea487)


Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"



Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories - so we use **. If you want to apply some particular pattern to define which files to send - use this syntax.



- ![postbuild](https://github.com/user-attachments/assets/41340ec2-f17c-4684-af25-3fcb3f3b416d)



Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:


- ![consoleoutput](https://github.com/user-attachments/assets/280f25f1-adb3-455f-97c4-3cde3d54543c)


To make sure that the files in /mnt/apps have been udated - connect via SSH/Putty to your NFS server and check README.MD file


                  cat /mnt/apps/README.md
                  

- ![updatedReadMe](https://github.com/user-attachments/assets/1513f722-4d00-4a41-b5a5-fe5833e42507)















