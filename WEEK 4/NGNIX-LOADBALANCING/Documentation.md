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

          
## Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates

Let us make necessary configurations to make connections to our Tooling Web Solution secured!

In order to get a valid SSL certificate - we need to register a new domain name, we can do it using any Domain name registrar - a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Namecheap,  Domain.com, Bluehost.com. 

- We can Register a new domain name with any registrar of our choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
- We will now Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

- ![associate-elastic-ip](https://github.com/user-attachments/assets/8a7d126e-d902-4fff-82e2-1292493e762f)

### Why Elastic IP
Every time we restart or stop/start your EC2 instance - we get a new public IP address. When we want to associate your domain name - it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem

- We will now Update A record in your registrar to point to Nginx LB using Elastic IP address

- ![namecheap](https://github.com/user-attachments/assets/118d1e19-4996-4941-a7d5-0b90f39b64dc)



We will now Check that our Web Servers can be reached from our browser using new domain name using HTTP protocol - http://<your-domain-name.com>


- ![mydomainpage](https://github.com/user-attachments/assets/31d3cdee-5049-42cb-b52a-69490ad5a44d)

- We need now Configure Nginx to recognize your new domain name by Updating our nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

- ![update domain](https://github.com/user-attachments/assets/18e07a6c-318c-47be-8942-7561785d3046)


### Install certbot and request for an SSL/TLS certificate

let's check that snapd service is active and running

      sudo systemctl status snapd

- ![snaprunnin](https://github.com/user-attachments/assets/37d8e0a8-5fcf-4bfd-bbbf-bb46d60eef29)

We now install certbot


    sudo snap install --classic certbot

We will now Request our certificate ( by just following the certbot instructions - we will need to choose which domain we want our certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it in the nginx config file.

    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    sudo certbot --nginx

- ![certificate](https://github.com/user-attachments/assets/1f9c1d37-8ee8-4c64-bfb2-67e0b1a8d6bf)


We will now Test secured access to our Web Solution by trying to reach https://<your-domain-name.com>

We shall be able to access our website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser's search string. Click on the padlock icon and you can see the details of the certificate issued for your website.

- ![final](https://github.com/user-attachments/assets/4f98f6eb-9f78-45e5-95e1-af0ebf0c523b)

- We need to now Set up periodical renewal of our SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

We can test renewal command in dry-run mode

    sudo certbot renew --dry-run

Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

      crontab -e
Add following line:

      * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1

- ![cronjob](https://github.com/user-attachments/assets/9e8c04e6-2a81-4c3c-add5-59217894e0a2)

We can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

























































































































