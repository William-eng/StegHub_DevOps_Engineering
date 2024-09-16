
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
On Ubuntu 20.04, Nginx has on server block enabled by default in the _/var/www/html_ directory. Instead of using the default directory, we will create our domain near this in /var/www/projectLEMP directory using the command;

            sudo mkdir /var/www/projectLEMP
   next we assign the directory to the current users permission with the command;

            sudo chown -R $USER:$USER /var/www/projectLEMP
 Then open the configuration file of nginx in sites-available directory and here we'll use the nano editor;

             sudo nano /etc/nginx/sites-available/projectLEMP 
  and paste this configuration command     

              #/etc/nginx/sites-available/projectLEMP
            
            server {
                listen 80;
                server_name projectLEMP www.projectLEMP;
                root /var/www/projectLEMP;
            
                index index.html index.htm index.php;
            
                location / {
                    try_files $uri $uri/ =404;
                }
            
                location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
                 }
            
                location ~ /\.ht {
                    deny all;
                }
            
            }
                
Here’s what each of these directives and location blocks do:
## listen 80;
 Tells Nginx to listen on port 80, which is the default port for HTTP traffic.
## server_name projectLEMP www.projectLEMP;
 Defines the domain names for this server block. Requests sent to either projectLEMP or www.projectLEMP will be handled by this configuration.
## root /var/www/projectLEMP;
 Specifies the root directory where the website files (HTML, PHP, etc.) are located. Nginx will serve files from /var/www/projectLEMP.
## index index.html index.htm index.php;
 Defines the order in which Nginx will look for the default file (if no file is specified in the URL). It will first look for index.html, then index.htm, and finally index.php.
## location / { try_files $uri $uri/ /index.php$is_args$args; }
Defines how Nginx should handle requests for the root of the website (/).
try_files $uri $uri/ /index.php$is_args$args;: This tells Nginx to:
Try to serve the file corresponding to the requested URI ($uri).
If it's a directory, try to serve the index file inside it ($uri/).
If neither exists, pass the request to index.php, appending any query string arguments ($is_args$args) to the request.
## location ~ \.php$ {
 This block matches PHP files (e.g., *.php).
include snippets/fastcgi.conf;: Includes standard FastCGI configuration for PHP handling.
fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;: Tells Nginx to pass PHP requests to the PHP-FPM service, which is running at the socket /var/run/php/php8.3-fpm.sock. This is how PHP scripts are executed and their output is returned to Nginx.
## location ~ /\.ht {
 This block matches any files starting with .ht, such as .htaccess.
deny all;: Denies access to these files to prevent exposure of configuration or sensitive data (like .htaccess or .htpasswd files).


We can now save and close the file. we can do so by typing CTRL+X and then y and ENTER to confirm.

Next, We activate our configuration by linking to the config file from Nginx’s sites-enabled directory:



            sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
            
 You can test your configuration for syntax errors by typing:



            sudo nginx -t
We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:


            sudo unlink /etc/nginx/sites-enabled/default
![test-config](https://github.com/user-attachments/assets/4d89f862-d158-4f03-96e1-784bbf0e89fc)
We can now reload Nginx to apply the changes:

            sudo systemctl reload nginx


Our new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:
                        
            sudo echo 'Hello LEMP from hostname' $(TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
            

              
Now on our browser we can try to open your website URL using IP address:

            http://<Public-IP-Address>:80
![newp](https://github.com/user-attachments/assets/db14f9fb-7d05-4405-8373-2d449727e515)

We can leave this file in place as a temporary landing page for our application until you set up an index.php file to replace it. Once we do that, we must remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

Our LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.

## STEP FIVE – Testing PHP with Nginx

At this point, our LAMP stack is completely installed and fully operational. we can test it to validate that Nginx can correctly hand .php files off to your PHP processor. We can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
             
             nano /var/www/projectLEMP/info.php
We type the following lines into the new file. This is valid PHP code that will return information about your server:

            <?php
            phpinfo();
You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:

            http://`server_domain_or_IP`/info.php

![php](https://github.com/user-attachments/assets/779a3539-d7a0-41c2-85a7-671122a24290)


After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about our PHP environment and your Ubuntu server. You can use rm to remove that file:

            sudo rm /var/www/your_domain/info.php
## STEP SIX — Retrieving data from MySQL database with PHP
In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

We will create a database named example_database and a user named example_user, but you can replace these names with different values.

First, connect to the MySQL console using the root account:

            sudo mysql -p
![login](https://github.com/user-attachments/assets/600768c2-04ad-4f20-807e-d83fddc607a1)
To create a new database, run the following command from your MySQL console:

            CREATE DATABASE `example_database`;
Now we can create a new user and grant him full privileges on the database we have just created.

The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as PassWord.1, but you should replace this value with a secure password of your own choosing.

             CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
Now we need to give this user permission over the example_database database:

            GRANT ALL ON example_database.* TO 'example_user'@'%';
This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.

Now exit the MySQL shell with:
            
            exit()
![createUser](https://github.com/user-attachments/assets/7636599d-d742-4bf4-9cda-de6624c4ea62)

We can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:

            mysql -u example_user -p
 Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database:

           SHOW DATABASES;
Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:            

            CREATE TABLE example_database.todo_list (
                         item_id INT AUTO_INCREMENT,
                         content VARCHAR(255),
                         PRIMARY KEY(item_id)
                          );
![tableset](https://github.com/user-attachments/assets/299eba8f-be36-4c34-b1e8-674c5b15aaaa)

Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:

            INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
 To confirm that the data was successfully saved to your table, run:

             SELECT * FROM example_database.todo_list;
 After confirming that you have valid data in your test table, you can exit the MySQL console:

             exit;

![table](https://github.com/user-attachments/assets/96391883-163b-43dc-8751-412c23bbe745)

Now we can create a PHP script that will connect to MySQL and query for our content. Create a new PHP file in your custom web root directory using your preferred editor:

             nano /var/www/projectLEMP/todo_list.php

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

Copy this content into your todo_list.php script:

            <?php
            $user = "example_user";
            $password = "PassWord.1";
            $database = "example_database";
            $table = "todo_list";
            
            try {
              $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
              echo "<h2>TODO</h2><ol>";
              foreach($db->query("SELECT content FROM $table") as $row) {
                echo "<li>" . $row['content'] . "</li>";
              }
              echo "</ol>";
            } catch (PDOException $e) {
                print "Error!: " . $e->getMessage() . "<br/>";
                die();
            }

Save and close the file when you are done editing.

You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:


