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




