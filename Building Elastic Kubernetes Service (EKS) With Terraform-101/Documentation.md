# Building Elastic Kubernetes Service (EKS) With Terraform-101
Since projects 21 and 22, we have had some fragmented experience around Kubernetes bootstrapping and deployment of containerized applications. This project seeks to solidify your skills by focusing more on real-world setups.

1. We will use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
2. We will deploy multiple applications using HELM
3. We will experience more Kubernetes objects and how to use them with Helm. Such as Dynamic provisioning of volumes to make pods stateful
4. We will improve upon your CI/CD skills with Jenkins
   
In Project 21, we created a k8s cluster from Ground-Up. That was quite painful, but very necessary to help master Kubernetes. Going forward, we will not have to do that. We will hardly ever have to do that even in the real world. given that cloud providers such as AWS have managed services for Kubernetes, they have done all the hard work, and with a few API calls to AWS, you can have a production-grade cluster ready to go in minutes. Therefore, in this project, you begin by focusing on [EKS](https://aws.amazon.com/eks/), and how to get it up and running using Terraform. Before moving on to other things.

## Building EKS with Terraform
At this point, you should already have some experience with Terraform. You have some work ahead of you, but you can begin with the steps outlined below. 

_If you have Terraform code from Project 16, simply update it to include EKS starting from step 6. If not, please follow the steps beginning at number 1_.

**Note:** Use Terraform version v1.0.2 and kubectl version v1.23.6.

1. Open up a new directory on your laptop, and name it eks

         mkdir EKS_Cluster_With_Terraform
         cd EKS_Cluster_With_Terraform
   
2. Use AWS CLI to create an S3 bucket

         aws s3 mb s3://ktrontech-eks-terraform-state --region us-east-1

3. Create a file – _backend.tf_ Task for you, ensure the backend is configured for remote state in S3



























































































































































































































