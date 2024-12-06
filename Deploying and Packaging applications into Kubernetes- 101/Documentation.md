# DEPLOYING AND PACKAGING APPLICATIONS INTO KUBERNETES WITH HELM

In the previous project, you explored Helm as a powerful tool for deploying applications into Kubernetes and expanded your experience by installing additional tools beyond Jenkins.

In this project, you will intensively deploy a variety of DevOps tools and tackle real-world challenges that come with such deployments. You will actively learn how to adjust Helm values files to automate the 
configuration of the applications you deploy. Once you have successfully set up the majority of the DevOps tools, you will leverage them to gain a comprehensive understanding of the DevOps cycle and how they seamlessly
integrate into the overall ecosystem.

Our focus will be on the following tools:

- Artifactory
- Hashicorp Vault
- Prometheus
- Grafana
- Elasticsearch ELK using [ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-install-helm.html)


Lets start first with Artifactory. What is it exactly?

Artifactory is part of a suit of products from a company called [Jfrog](https://jfrog.com/). Jfrog started out as an artifact repository where software binaries in different formats are stored. Today, Jfrog has 
transitioned from an artifact repository to a DevOps Platform that includes CI and CD capabilities. This has been achieved by offering more products in which **Jfrog Artifactory** is part of. Other offerings include

- JFrog Pipelines – a CI-CD product that works well with its Artifactory repository. Think of this product as an alternative to Jenkins.
- JFrog Xray – a security product that can be built-into various steps within a JFrog pipeline. Its job is to scan for security vulnerabilities in the stored artifacts. It is able to scan all dependent code.
- 
In this project, the requirement is to use Jfrog Artifactory as a private registry for the organisation’s Docker images and Helm charts. This requirement will satisfy part of the company’s corporate security policies
to never download artifacts directly from the public into production systems. We will eventually have a CI pipeline that initially pulls public docker images and helm charts from the internet, store in artifactory and
scan the artifacts for security vulnerabilities before deploying into the corporate infrastructure. Any found vulnerabilities will immediately trigger an action to quarantine such artifacts.

Lets get into action and see how all of these work.

## Deploy Jfrog Artifactory into Kubernetes

The best approach to easily get Artifactory into kubernetes is to use helm.

1. Search for an official helm chart for Artifactory on [Artifact Hub](https://artifacthub.io/)

- ![Image0](https://github.com/user-attachments/assets/e2e9dd6a-d5db-4184-a42b-7545ff6b2c83)


2. Click on **See all results**
3. Use the filter checkbox on the left to limit the return data. As you can see in the image below, “Helm” is selected. In some cases, you might select “Official”. Then click on the first option from the result.

- ![Image1](https://github.com/user-attachments/assets/72b94b45-6db4-4aba-add1-30e4cdcf80fc)

4. Review the Artifactory page
- ![Image2](https://github.com/user-attachments/assets/f23da962-fbbf-40c1-9d46-24dd03653be7)
   
5. Click on the install menu on the right to see the installation commands.

- ![Image3](https://github.com/user-attachments/assets/718fc30f-ef09-4d5f-975c-480a43de3233)

6. Add the jfrog remote repository on your laptop/computer

          helm repo add jfrog https://charts.jfrog.io
   
7. Create a namespace called tools where all the tools for DevOps will be deployed. (In previous project, you installed Jenkins in the default namespace. You should uninstall Jenkins there and install in the new namespace)

          kubectl create ns tools

   
8. Update the helm repo index on your laptop/computer

        helm repo update

  - ![Image4](https://github.com/user-attachments/assets/3f809e93-9553-4fe3-bc31-ce7754acf19c)
 
   
9. Install artifactory

                helm upgrade --install artifactory jfrog/artifactory --version 107.90.10 -n tools
    
                Release "artifactory" does not exist. Installing it now.
                NAME: artifactory
                LAST DEPLOYED: Thu Sep 26 01:37:01 2024
                NAMESPACE: tools
                STATUS: deployed
                REVISION: 1
                TEST SUITE: None
                NOTES:
                Congratulations. You have just deployed JFrog Artifactory!
                
                1. Get the Artifactory URL by running these commands:
                
                   NOTE: It may take a few minutes for the LoadBalancer IP to be available.
                         You can watch the status of the service by running 'kubectl get svc --namespace tools -w artifactory-artifactory-nginx'
                   export SERVICE_IP=$(kubectl get svc --namespace tools artifactory-artifactory-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                   echo http://$SERVICE_IP/
                
                2. Open Artifactory in your browser
                   Default credential for Artifactory:
                   user: admin
                   password: password



- ![Image5](https://github.com/user-attachments/assets/d0e1ad8e-4abb-4e82-af8d-6de83c5e969b)

**NOTE**:

We have used upgrade --install flag here instead of helm install artifactory jfrog/artifactory This is a better practice, especially when developing CI pipelines for helm deployments. It ensures that helm does an upgrade if there is an existing installation. But if there isn’t, it does the initial install. With this strategy, the command will never fail. It will be smart enough to determine if an upgrade or fresh installation is required.
The helm chart version to install is very important to specify. So, the version at the time of writing may be different from what you will see from Artifact Hub. So, replace the version number to the desired. You can see all the versions by clicking on “see all” as shown in the image below.













































































































































































































































































