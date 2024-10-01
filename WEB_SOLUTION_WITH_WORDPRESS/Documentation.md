# Web Solution With WordPress
In this project we will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management 
system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This consists of two parts:
  1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks,
  partitions and volumes in Linux.

  2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

As a DevOps engineer, our deep understanding of core components of web solutions and ability to troubleshoot them will play essential role in your further progress and development.

NB: We will understand by implementing the Three tier Architecture with involves the (i) Presentation Layer  (ii) Business/Logic Layer (iii) Data Layer

## Our 3-Tier Setup
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server

### We will use a RedHat Linux Based Distribution

Let's Begin

## STEP ONE : PREPARE A WEB SERVER
  1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

     ![webvol](https://github.com/user-attachments/assets/f2fe6ba7-ed36-41f5-90cd-221f7b7d7b19)

  2. We will attach all three volumes one by one to our Web Server EC2 instance
     - Open up the Linux terminal to begin configuration
     - Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there 

      ![lsblk](https://github.com/user-attachments/assets/767dbe3e-1745-4d42-8922-0b3de5ae2d2d)

      - We will use df -h command to see all mounts and free space on our server
      - and gdisk utility to create a single partition on each of the 3 disks with the command:
   
                sudo gdisk /dev/xvdb # repeat for xvdc and xvdd
    
        ![gdisk](https://github.com/user-attachments/assets/02c7c985-479a-4d9f-941e-059385f2b002)
        

              
  3. We will now use lsblk utility to view the newly configured partition on each of the 3 disks.

  ![llkprt](https://github.com/user-attachments/assets/5a703f3a-7fff-4d93-94b3-7fdc9e6cdaef)

   
  4. We now install lvm2 package using _sudo yum install lvm2_. and run _sudo lvmdiskscan_ command to check for available partitions.

     ![InstallLVM](https://github.com/user-attachments/assets/0f8ed4ef-a103-4373-8c68-8d4e1d8cd48d)
     ![lvdm](https://github.com/user-attachments/assets/53e11b23-4d98-487d-8d57-3d9ae139aa60)

  5. We will use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

            sudo pvcreate /dev/xvdb1
            sudo pvcreate /dev/xvdc1
            sudo pvcreate /dev/xvdd1
     
  ![pvcreate](https://github.com/user-attachments/assets/f7741f07-2964-433c-a39a-115bca689963)
  6. We will now verify that our Physical volume has been created successfully by running _sudo pvs_

  ![sudoPVS](https://github.com/user-attachments/assets/1b0d94f7-19cd-4ca2-813e-334a462e611c)

  7. We will now use _vgcreate_ utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

           sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
   
  8. Verify that your VG has been created successfully by running _sudo vgs_

     ![vgsgain](https://github.com/user-attachments/assets/91efb997-4e28-44fb-a9dc-b12fc0433c6a)

     
  9. We will now use _lvcreate_ utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size.
  10.  NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

              sudo lvcreate -n apps-lv -L 14G webdata-vg
              sudo lvcreate -n logs-lv -L 14G webdata-vg
  
  11. We will now Verify that your Logical Volume has been created successfully by running _sudo lvs_

      ![lvs](https://github.com/user-attachments/assets/7fb1f1a2-9c59-49c8-bbd9-e26d231dda08)

  12. We wil now Verify the entire setup

          sudo vgdisplay -v #view complete setup - VG, PV, and LV
          sudo lsblk
      
  ![fullSetup](https://github.com/user-attachments/assets/e67ea0fd-e9ea-40df-be15-4f1e1150e7dc)

  We will use _mkfs.ext4_ to format the logical volumes with ext4 filesystem

      sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
      sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
  13. Create /var/www/html directory to store website files _sudo mkdir -p /var/www/html_
  14. Create /home/recovery/logs to store backup of log data _sudo mkdir -p /home/recovery/logs_
  15. Mount /var/www/html on apps-lv logical volume

            sudo mount /dev/webdata-vg/apps-lv /var/www/html/

  17. We will Use _rsync_ utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

          sudo rsync -av /var/log/ /home/recovery/logs/

  18. We will Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)

          sudo mount /dev/webdata-vg/logs-lv /var/log

  19. Restore log files back into /var/log directory

          sudo rsync -av /home/recovery/logs/ /var/log
  20. We will Update /etc/fstab file so that the mount configuration will persist after restart of the server. The UUID of the device will be used to update the /etc/fstab file;

            sudo vi /etc/fstab

  We will Update the /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

  ![fstabfile](https://github.com/user-attachments/assets/3b2fc93d-fa08-4430-aa92-6caeece99fe4)

  21. We will now Test the configuration and reload the daemon

              sudo mount -a
              sudo systemctl daemon-reload

  22. We will now Verify our setup by running _df -h_, output must look like this:

   ![Df](https://github.com/user-attachments/assets/2ac856d9-2cec-422a-a282-2a074ed14808)

## STEP TWO: PREPARE THE DATABASE
   
We will now Launch a second RedHat EC2 instance that will have a role - 'DB Server' Repeat the same steps as for the Web Server, but instead of apps-lv we will create db-lv and mount it to /db directory instead of /var/www/html/.

![dbsetup](https://github.com/user-attachments/assets/4e0bee37-72c7-401f-a121-c197ed252295)

## STEP THREE : INSTALL WORDPRESS ON EC2 WEBSERVER

- Update the repository

        sudo yum -y update
- Install wget, Apache and it's dependencies

       sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
  ![nstall](https://github.com/user-attachments/assets/e35a8cd0-5011-4e55-9f90-3444324c216f)

- Start Apache

        sudo systemctl enable httpd
        sudo systemctl start httpd
  
- To install PHP and it's dependencies

        sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
      sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
      sudo yum module list php sudo yum module reset php
      sudo yum module enable php:remi-7.4
      sudo yum install php php-opcache php-gd php-curl php-mysqlnd
      sudo systemctl start php-fpm
      sudo systemctl enable php-fpm setsebool -P httpd_execmem 1
- Restart Apache

      sudo systemctl restart httpd
![pfm](https://github.com/user-attachments/assets/e8e07a61-a2ae-428a-bc04-af78bad9a042)

  
- Download wordpress and copy wordpress to /var/www/html
  
            mkdir wordpress
            cd wordpress
            sudo wget http://wordpress.org/latest.tar.gz
            sudo tar -xzvf latest.tar.gz
            sudo rm -rf latest.tar.gz
            cp wordpress/wp-config-sample.php wordpress/wp-config.php
            cp -R wordpress /var/www/html/
  
- Configure SELinux Policies

          sudo chown -R apache:apache /var/www/html/wordpress
          sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
          sudo setsebool -P httpd_can_network_connect=1
                 
![webserver do](https://github.com/user-attachments/assets/b2c6e729-8a36-4d3d-99be-be04ca8d543c)

## STEP FOUR : Install MySQL on your DB Server EC2


      sudo yum update
      sudo yum install mysql-server        

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

      sudo systemctl restart mysqld
      sudo systemctl enable mysqld

## Step FIVE : Configure DB to work with WordPress


          sudo mysql
          CREATE DATABASE wordpress;
          CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
          GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
          FLUSH PRIVILEGES;
          SHOW DATABASES;
          exit
![dbs0etup](https://github.com/user-attachments/assets/34ab8df7-1999-4ccb-94ae-a609291f7975)

## Step SIX : Configure WordPress to connect to remote database.
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule configuration specify source as /32

We now install MySQL client and test that we can connect from our Web Server to our DB server by using mysql-client

        sudo yum install mysql
        sudo mysql -u admin -p -h <DB-Server-Private-IP-address>

     


     

