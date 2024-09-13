A “LAMP” stack is a group of open source software that is typically installed together in order to enable a server to host dynamic websites and web apps written in PHP. This term is an acronym which represents the Linux operating system with the Apache web server. The site data is stored in a MySQL database, and dynamic content is processed by PHP.
In order to complete this project, we need an AWS account and a server with Ubuntu OS Server

STEP ONE: Create and EC2 instance on AWS. Using the AWS Management Console, we can set up an ubuntu server in our most preferred region and connect to the EC2 instance using SSH cryptographic key known as PEM(Privacy Enhanced Mail) file


Next, we update the list of package and install the apache web server using the ubuntu package manager “apt” by running the command


                    `sudo  apt update
                     sudo  apt install apache2 -y `
 

And verify that apache2 is running by verifying the command below


                    `sudo systemctl status apache2`



Once our server is running, we can locally access it on port 80 using the curl command or the public ip of our server on port 80 <public-ip>:80

STEP TWO: Installing MYSQL, this is the database  management system that stores and manages data for relational database we install and connect to the mysql database by the commands


                    `sudo  apt install mysql-server  #to install mysql `                                 
                    
                    `sudo mysql `  `# to connect to mysql `

We then set the user's password in MySQL, you can use the ALTER USER  command. Here's how to do it:



                    `ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPassword123';`

Exit the mysql console by typing the exit command and run the installation script with the command;


                    `sudo mysql_secure_installation`


We follow the prompt and set based in our preference, it is important that we set our root password when prompted, after which we login to the mysql server console again with the command


                    `sudo mysql -p
                    `
Our mysql is now installed and now we install the final component of our LAMP stack which is PHP

STEP THREE: Installing PHP, PHP is the component of our setup that will display dynamic contents to our end users. For this, we need to install three packages at once to enable apache to communicate with php and php to communicate with mysql. We run the command;


                    `sudo apt install php libapache2-mod-php php-mysql `
                    
And to confirm the installation 


                    `php  -v
`



At this point, our Lamp Stack is Completely Operation.

STEP FOUR: Creating a Virtual Host For Your Website Using Apache
Here we are going to set up a domain of our choice(I’m Using lampProject). Apache has a default server block that is configured to serve documents from the var/www/html directory, we will add our own directory next to this using the command;


                    `mkdir /var/www/lampProject`


Next we assign the ownership to our current user user the command;

                    
                    `sudo chown  -R  $USER:$USER  /var/www/lampProject`

Then, we create a new configuration file in the sites-available directory using the command;


                    `sudo vi /etc/apache2/sites-available/ lampProject`

And paste the below config file in it



                        `<VirtualHost *:80>
                            ServerName lampProject
                            ServerAlias www.lampProject
                            ServerAdmin webmaster@localhost
                            DocumentRoot /var/www/lampProject
                            ErrorLog ${APACHE_LOG_DIR}/lampProject_error.log
                            CustomLog ${APACHE_LOG_DIR}/lampProject_access.log combined
                        </VirtualHost>`


save and quit with :wq. We can now enable virtual host, disable default website and check for syntax errors using the following commands respectively.


                       `sudo a2ensite lampProject.conf
                        sudo a2dissite 000-default.conf
                        sudo apache2ctl configtest
                        `
Use the sudo systemctl reload apache2 to apply the changes
We need to create an index.html file in the _/var/www/lampProject _directory and add a random html file, which can be viewed locally on port 80.

STEP FIVE: Enable PHP on the website, by default, the index.html file will always take precedence over the index.php file, this is great for maintenance, but when the maintenance is done the settings can be adjusted by editing the /etc/apache2/mods-enabled/dir.conf file and changing index.html to index.php

Save this file and reload apache. 
Create the php file with _sudo vi /var/www/lampProject/index.php_ and add the command


                        `<?php
                        phpinfo();`

Save and reload the localhost page on port 80 and a php file will appear.



 