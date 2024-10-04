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





  

