
# LEMP 
LEMP is a software stack used for hosting dynamic websites and web applications. It stands for:
Linux: The operating system on which the stack runs.
Engine-X (Nginx): The web server that handles HTTP requests. It's often used instead of Apache in LAMP stacks due to its performance and scalability.
MySQL (or MariaDB): The database management system used to store and retrieve data. 
PHP (or Python/Perl): The programming language used for server-side scripting and generating dynamic content. Let's go through the process of setting 
process of setting up  a LEMP Stack.

## STEP ONE : INSTALLING NGNIX SERVER
We start off by updating the package manager on our server and then installing the Nginx service with our _apt_
package manager( NB: We are using an Ubuntu Server) with the commands;

            sudo apt update && sudo apt install nginx -y
  ![Nginx-installation](https://github.com/user-attachments/assets/5aacb8a9-a7e6-4224-987e-f680dd690564)
         
 Once the installation is finished, Ngnix starts running, we can comfirm that by running the command;

            sudo systemctl status nginx
  ![Nginx-running](https://github.com/user-attachments/assets/1bed17fa-bd52-4ebb-88c6-b0eca82aa991)

          
  We must ensure that port 80 of our server is opened, hence makes our server allow traffic from the web, we can then check the nginix
  default page on port 80 by running the command;

   ![curl](https://github.com/user-attachments/assets/326a7368-0b07-482c-bff9-2269df145da6)
   
        

            curl http//:localhost:80 
   or access it on the web browser with the server's public ip address <public-ip-server>:80   
   ![nginx page](https://github.com/user-attachments/assets/fdb7fbaf-6e7b-4da8-8f00-499ac5797c9e)

## STEP TWO: INSTALLING MYSQL
WE now need a database management system for our websever and MYSQL is a popular RDS used in php environment. we can install this 
by running the command;

            sudo apt install myaql-server
![mysqllemp](https://github.com/user-attachments/assets/d302f0de-654f-4b89-9c60-9372433d18d0)
After the installation is complete, we can login to the mysql console using the command;

            sudo mysql
![sudo-mysql](https://github.com/user-attachments/assets/73dff05d-1905-4fdb-b217-d11a58b39f4d)

When the installation is finished, it’s recommended that you run a security script that comes pre-installed with MySQL. 
This script will remove some insecure default settings and lock down access to your database system. 
But first we need to set root password using mysql_native_password with the command;

            ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'NEW_PASSWORD';


exit and Start the interactive script by running the following command:

            sudo mysql_secure_installation
and follow the prompts as shown below or custom to your preference.
![Secure-Installation](https://github.com/user-attachments/assets/7c31727a-cecb-478f-b3c6-04832e9a8d83)

Our sql is now installed and set up, we can move to the next and the final set up.

## STEP THREE: INSTALLING PHP
You have Nginx installed to serve your content and MySQL installed to store and manage your data. 
Now you can install PHP to process code and generate dynamic content for the web server.
While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle
PHP processing and act as a bridge between the PHP interpreter itself and the web server.
This allows for better overall performance in most PHP-based websites, but it requires additional configuration. 
You’ll need to install php-fpm, which stands for “PHP fastCGI process manager” 
to tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, 
a PHP module that allows PHP to communicate with MySQL-based databases.
Core PHP packages will automatically be installed as dependencies. 
To install the php-fpm and php-mysql packages, run;

            sudo apt install php-fpm php-mysql -y
![phpinsta](https://github.com/user-attachments/assets/7951738a-63b5-41a1-97fe-ef44454508df)
Now that we have php components installed, we need to configure nginx to use them

# STEP FOUR: CONFIGURING NGINX TO USE PHP PROCESSOR
While using Nginxwebserver, we can create an NGINX server block that will make use of the above FPM pool. 
To do that, edit your NGINX configuration file and pass the path of pool’s socket file using the option fastcgi_pass inside location block for php.
On Ubuntu 20.04, Nginx has on server block enabled by default in the _/var/www/html_ directory. Instead of using the default directory, we will create our domain near this in /var/www/projectLEMP directory


            

            


            
            

              
