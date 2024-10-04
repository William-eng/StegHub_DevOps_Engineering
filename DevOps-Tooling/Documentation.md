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



  

