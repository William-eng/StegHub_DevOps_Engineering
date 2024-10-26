# AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY

## General Overview
You will create a secure infrastructure within AWS VPC (Virtual Private Cloud) for a fictional company named KTrontech. This infrastructure will support Ktrontech's main business website, 
which uses the WordPress Content Management System (CMS), as well as a Tooling Website (PHP/MySQL) for their DevOps team. To enhance security and performance, the company has decided to 
implement **NGINX reverse proxy technology**. The project prioritizes cost-efficiency, security, and scalability. The goal is to develop an architecture that ensures both the WordPress and 
Tooling websites are resilient to web server failures, can handle increased traffic, and maintain reasonable costs.

- ![image-96-1024x955](https://github.com/user-attachments/assets/9c4f4747-0162-431b-8f0d-6f64b9601fe0)
## - Requirements
- There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit
Create an AWS Master account. (Also known as Root Account)
Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

- ![devOU](https://github.com/user-attachments/assets/c6fa7c2e-305b-4ed7-b017-a672d553887f)
- ![move](https://github.com/user-attachments/assets/593e5177-5d67-42a6-a4d7-468c5af412e2)

- Login to the newly created AWS account using the new email address.
2. Create a domain name for your company. I used Hostinger.
3. Create a hosted zone in AWS, and map it to your domain. to do this ,
 - On your AWS console search bar, type route 53 and click on it, > select create hosted zones
- ![hosted-zone](https://github.com/user-attachments/assets/e368c2ad-2d89-4323-9d8f-da5c3d0a32bd)

- Set your already purchased domain name, and set the type o public hosted zone, click on create hosted zone
- copy the namservers and go back to your domain management , and add the nameservers, allow it sometime to propagate, usually between few hours to a day








































































































































