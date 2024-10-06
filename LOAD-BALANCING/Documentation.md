# Load Balancer Solution With Apache
We will Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. We will Make sure that users can be
served by Web servers through the Load Balancer.
To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

## Prerequisites
1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server

## Configure Apache As A Load Balancer
1. Create an Ubuntu Server 20.04 EC2 instance and name it _Project-8-apache-lb_, so your EC2 list will look like this:
2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

- ![ngnixload](https://github.com/user-attachments/assets/5018182d-66de-48d3-b9c6-f087427a3750)

3. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

           #Install apache2
        sudo apt update
        sudo apt install apache2 -y
        sudo apt-get install libxml2-dev
        
        #Enable following modules:
        sudo a2enmod rewrite
        sudo a2enmod proxy
        sudo a2enmod proxy_balancer
        sudo a2enmod proxy_http
        sudo a2enmod headers
        sudo a2enmod lbmethod_bytraffic
        
        #Restart apache2 service
        sudo systemctl restart apache2
- ![cofig](https://github.com/user-attachments/assets/106e1a5f-36be-4cda-8906-7375fa9cffdd)
- ![installation](https://github.com/user-attachments/assets/5d5fc5cf-8bef-4e1b-874b-6b238f886945)

Make sure apache2 is up and running

      sudo systemctl status apache2
- ![apacherunning](https://github.com/user-attachments/assets/bf3f8d27-b8e7-4191-bbe7-02d8b18beac7)

Configure load balancing


            sudo vi /etc/apache2/sites-available/000-default.conf
    
    #Add this configuration into this section <VirtualHost *:80>  </VirtualHost>
    
    <Proxy "balancer://mycluster">
                   BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
                   BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
                   ProxySet lbmethod=bytraffic
                   # ProxySet lbmethod=byrequests
            </Proxy>
    
            ProxyPreserveHost On
            ProxyPass / balancer://mycluster/
            ProxyPassReverse / balancer://mycluster/
    
    #Restart apache server
    
    sudo systemctl restart apache2

- ![siteconfig](https://github.com/user-attachments/assets/5e9b843d-7f03-48c4-bdf5-485648768a0c)
bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

You can also study and try other methods, like: _bybusyness, byrequests, heartbeat_

4. Verify that our configuration works - try to access your LB's public IP address or Public DNS name from your browser:


        http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
- ![load-balancverpage](https://github.com/user-attachments/assets/87656cd7-14c3-42de-971c-ff5e1aa7ba1b)

Open two ssh/ consoles for both Web Servers and run following command:


        sudo tail -f /var/log/httpd/access_log

Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB - new records must appear in each server's log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers - it means that traffic will be disctributed evenly between them.

If you have configured everything correctly - your users will not even notice that their requests are served by more than one server.

- ![serverA](https://github.com/user-attachments/assets/cc89b6bf-1f4c-485b-9428-2f1539d9120c)


- ![SERVERB](https://github.com/user-attachments/assets/29b15799-183d-40f7-a17d-efd6da0afed0)














    



