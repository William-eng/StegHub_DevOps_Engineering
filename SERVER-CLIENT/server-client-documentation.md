## This Project involves the Implentation of a Client-Server Architecture Using MySQL Database Management System (DBMS)

To demonstrate a basic client-server using MySQL RDBMS, follow the below instructions;

## 1. Create and configure two Linux-based virtual servers (EC2 instances in AWS).
   
- Server A name - `mysql server`
- Server B name - `mysql client`

![createinstance](https://github.com/user-attachments/assets/5ebf01af-d654-48e3-aec7-e4ed570c39f0)

## 2. On mysql server Linux Server, we install MySQL Server software.

  1. login into the mysql-server instance.
  2. Updated package lists and installed MySQL server:
  
               sudo apt update
               sudo apt install mysql-server -y

![sqlserver](https://github.com/user-attachments/assets/e2b9eaa1-8c53-4361-83b6-0098d4ee724e)


## 3. On mysql client Linux Server install MySQL Client software.

   1. Started and enabled the MySQL service:
  
          sudo systemctl start mysql
          sudo systemctl enable mysql


   2. Set the root password:

            sudo mysql
            ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<your_own_password>';
            FLUSH PRIVILEGES;
            EXIT;
      
   3. Run the secure installation script:

          sudo mysql_secure_installation



## 4. By default, both of our EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. We will use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so we will have to open it by creating a new entry in 'Inbound rules' in 'mysql server' Security Groups. For extra security, not allowing all IP addresses to reach your 'mysql server' - allowing access only to the specific local IP address of your 'mysql client'.

![3306onIp](https://github.com/user-attachments/assets/b9d31097-d82f-4432-b7cb-198191e2ed76)

## 5. We might need to configure MySQL server to allow connections from remote hosts.

         sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
   Identified and modified two bind-address settings:

       bind-address = 0.0.0.0
       mysqlx-bind-address = 127.0.0.1

   Restart MySQL:

         sudo systemctl restart mysql
   Created a MySQL user for remote access:

       CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Password.1';
       GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'%';
       FLUSH PRIVILEGES;

## 6. Setting Up MySQL Client

- Login into the mysql-client instance.
- Install MySQL client:

      sudo apt update
      sudo apt install mysql-client -y

![mysqlclient](https://github.com/user-attachments/assets/5c67b218-ee3a-4fa1-9ddf-0071ab543048)

## 7. Establishing Connection
   We connect the MySQL client from mysql-client to the MySQL server on mysql-server. From mysql-client, using the following command to connect:

         mysql -h <mysql-server Private IP> -u remote_user -p
   
## NOTE
We can test connection from our mysql-client
   1. Ping
   To test the network connectivity between the instances, We can use the ping command:

           ping <mysql-server Private IP>
   
   2. Traceroute
   For a detailed path analysis, we can also use traceroute:

            traceroute <mysql-server Private IP>

## 8. We can check that we have successfully connected to a remote MySQL server and can perform SQL queries:

        Show databases;

We should see an output similar to the below image, then we have successfully completed this project - we have deloyed a fully functional MySQL Client-Server set up. 

    
   






   
