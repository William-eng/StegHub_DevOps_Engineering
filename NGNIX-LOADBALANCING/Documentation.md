# Load Balancer Solution With Nginx and SSL/TLS
This project consists of two parts:

- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates
- Target architecture will look like this:

- ![image-42-1024x623](https://github.com/user-attachments/assets/f4517308-7e13-44fa-88ba-d938cf197328)



## Part 1 - Configure Nginx As A Load Balancer

### Create a fresh installation of Linux for Nginx.

- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS
connections)
- ![Ngnix server](https://github.com/user-attachments/assets/c322673e-2a4e-4da5-bd13-a74339e5437f)

- Update /etc/hosts file for local DNS with Web Servers' names (e.g. Web1 and Web2) and their local IP addresses

- ![etchost](https://github.com/user-attachments/assets/a3e45e27-1051-4c2b-b06a-1e9436773741)

  
- Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
  
  Update the instance and Install Nginx Install Nginx

        sudo apt update
        sudo apt install nginx

  
Open the default nginx configuration file

      sudo nano /etc/nginx/nginx.conf

              
#insert following configuration into http section

           upstream myproject {
              server Web1 weight=5;
              server Web2 weight=5;
            }
          
          server {
              listen 80;
              server_name www.domain.com;
              location / {
                proxy_pass http://myproject;
              }
            }
   #comment out this line
         
           #       include /etc/nginx/sites-enabled/*;

Restart Nginx and make sure the service is up and running

      sudo systemctl restart nginx
      sudo systemctl status nginx
- ![ngrun](https://github.com/user-attachments/assets/c7ab42c1-afa4-4844-afd1-79dc9ffaf27c)

          







































































































































