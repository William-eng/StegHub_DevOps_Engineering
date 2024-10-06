# STEP ONE: PREPARE AN NFS SERVER
1. Spin up a new EC2 instance with RHEL Linux 8 Operating System.

   ![NFSCreate](https://github.com/user-attachments/assets/04ff6008-5fd5-43b6-909f-f80a3128e513)

2. Configure LVM on the Server.

  ![lvmconfig](https://github.com/user-attachments/assets/3c441ce8-5c74-4660-a042-6f2a6906a91a)

3. Instead of formatting the disks as ext4 we will have to format them as xfs
  ![format](https://github.com/user-attachments/assets/eaf9c6a8-686d-4514-bf69-212c16a10035)

- Create mount points on /mnt directory for the logical volumes as follows: Mount lv-apps on /mnt/apps -
- To be used by webservers Mount lv-logs on /mnt/logs - To be used by webserver logs Mount lv-opt on /mnt/opt - To be used by Jenkins server in
   Project 8

4. Install NFS server, configure it to start on reboot and make sure it is u and running
![fstabmount](https://github.com/user-attachments/assets/a11fe5ae-9c45-4b52-a071-e6d9431d6151)
![mounted](https://github.com/user-attachments/assets/95d57461-0467-45c2-99ea-a93595dd2f87)

        sudo yum -y update
        sudo yum install nfs-utils -y
        sudo systemctl start nfs-server.service
        sudo systemctl enable nfs-server.service
        sudo systemctl status nfs-server.service

Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

      sudo chown -R nobody: /mnt/apps
      sudo chown -R nobody: /mnt/logs
      sudo chown -R nobody: /mnt/opt
      
      sudo chmod -R 777 /mnt/apps
      sudo chmod -R 777 /mnt/logs
      sudo chmod -R 777 /mnt/opt
      
      sudo systemctl restart nfs-server.service

Configure access to NFS for clients within the same subnet (example of Subnet CIDR - 172.31.32.0/20 ):

      sudo vi /etc/exports
      
      /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
      /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
      /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
      
      Esc + :wq!
      
      sudo exportfs -arv

 ![onfig](https://github.com/user-attachments/assets/f609491b-bd43-415b-8a62-66cee8c47f10)

 5. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

        rpcinfo -p | grep nfs
    
![nfsinfo](https://github.com/user-attachments/assets/241bcdd3-855e-4d79-86a9-237cdd1957f0)



Important note: In order for NFS server to be accessible from our client, we must also open following ports: TCP 111, UDP 111, UDP 2049

![sg](https://github.com/user-attachments/assets/0a8e138a-5fe4-4057-aea1-6799439f3c58)

# STEP TWO: CONFIGURE THE DATABASE
1. Install MySQL server

       sudo yum update
       sudo yum install mysql-server


         sudo systemctl restart mysqld
         sudo systemctl enable mysqld

2. Create a database and name it _tooling_
  
         sudo mysql
         CREATE DATABASE tooling;
   
4. Create a database user and name it _webaccess_
   
        CREATE USER 'webaccess'@'%'172.31.16.10/20' IDENTIFIED BY 'mypass';


5. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr
   
         GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.16.10/20';
         FLUSH PRIVILEGES;
         SHOW DATABASES;
         exit;

![DBconf](https://github.com/user-attachments/assets/b2538714-ea74-45b0-95f0-59df5903d7a4)


# STEP THREE: Prepare the Web Servers
We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case - NFS Server and MySQL database. We already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use - we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.
During the next steps we will do following:

- Configure NFS client (this step must be done on all three servers)
- Deploy a Tooling application to our Web Servers into a shared NFS folder
- Configure the Web Servers to work with a single MySQL database
  
1. Launch a new EC2 instance with RHEL 8 Operating System
  
2. Install NFS client

      sudo yum install nfs-utils nfs4-acl-tools -y

![nfsinst](https://github.com/user-attachments/assets/d1c230f1-820a-49e1-9a7a-3211f65e70b3)



3. Mount /var/www/ and target the NFS server's export for apps

            sudo mkdir /var/www
            sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

4. Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:
   
         sudo vi /etc/fstab

 and add this line  

          172.31.18.140:/mnt/apps /var/www nfs defaults 0 0
![correct munt](https://github.com/user-attachments/assets/3aebef2c-2849-4193-bdd0-bc7bcc3160ce)


5. Install Remi's repository, Apache and PHP

            sudo yum install httpd -y
            
            sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
            
            sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
            
            sudo dnf module reset php
            
            sudo dnf module enable php:remi-7.4
            
            sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
            
            sudo systemctl start php-fpm
            
            sudo systemctl enable php-fpm
            
            setsebool -P httpd_execmem 1

**Repeat steps 1-5 for another 2 Web Servers.**
6. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files - it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.
![Screenshot from 2024-10-04 22-28-44](https://github.com/user-attachments/assets/b4eaec10-9f41-48ee-98f1-4183ae9d3f4b)
![Screenshot from 2024-10-04 22-28-39](https://github.com/user-attachments/assets/1219c4ec-8472-4a7a-82e6-148fb48d139c)
![Screenshot from 2024-10-04 22-28-32](https://github.com/user-attachments/assets/51ab7acd-a20d-45e1-b43b-3fb8a85b1deb)


7. Locate the log folder for Apache on the Web Server and mount it to NFS server's export for logs. Repeat step №4 to make sure the mount point will persist after reboot.

8. Fork the tooling source code from StegHub Github Account to your Github account. 
![fork](https://github.com/user-attachments/assets/1f518323-0935-4035-826f-36e8e642f081)

9. Deploy the tooling website's code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html
![clone](https://github.com/user-attachments/assets/d8c02c38-5d99-4c99-a831-aa716b0921c7)


Note 1: Do not forget to open TCP port 80 on the Web Server.

Note 2: If you encounter 403 Error - check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0 To make this change permanent - open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabled, then restrt httpd.
![loginN](https://github.com/user-attachments/assets/485a1385-1860-4180-8d25-5273483fa4a1)

10. Update the website's configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
![reconfig tooling](https://github.com/user-attachments/assets/f4ad5949-c80f-483c-8221-b33bc9b30da1)

11. Create in MySQL a new admin user with username: myuser and password: password:

    ![insertuser](https://github.com/user-attachments/assets/38b8bbc1-b7c6-4712-bae4-c047d9811d3e)

12. Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the websute with myuser user.    

![Screenshot from 2024-10-06 12-26-45](https://github.com/user-attachments/assets/553ff4a4-e853-4410-ac5e-618740d11962)

![Screenshot from 2024-10-05 21-08-37](https://github.com/user-attachments/assets/1fa1ae00-273e-4736-bccd-fff7210c92fd)






























          
          

   
