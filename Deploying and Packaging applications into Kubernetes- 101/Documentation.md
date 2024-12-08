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

-   ![Image6](https://github.com/user-attachments/assets/455c534b-7dc6-4b1a-913d-d13d39fa28ba)

- ![Image7](https://github.com/user-attachments/assets/071d2ba6-8fd7-4461-9e8f-0c3bfe1fba89)


The output from the installation already gives some Next step directives.

## Getting the Artifactory URL
Lets break down the first Next Step.

1. The artifactory helm chart comes bundled with the Artifactory software, a PostgreSQL database and an Nginx proxy which it uses to configure routes to the different capabilities of Artifactory. Getting the pods after some time, you should see something like the below.

- ![Image8](https://github.com/user-attachments/assets/388c5caa-ca26-4cf7-8cd0-7c0809d225fc)

This output shows that the artifactory pod and nginx pod are running but not in a Ready state.

- ![Image9](https://github.com/user-attachments/assets/a3f3adf5-572c-4fd2-b308-ea7355c9baa3)






- ![Image6](https://github.com/user-attachments/assets/f4339d97-2c17-484f-90e5-255c13222946)



2. Each of the deployed application have their respective services. This is how you will be able to reach either of them.


- ![Image7](https://github.com/user-attachments/assets/f621aeaa-dea4-4f53-b5ff-8536a75c59a6)

3. Notice that, the Nginx Proxy has been configured to use the service type of LoadBalancer. Therefore, to reach Artifactory, we will need to go through the Nginx proxy’s service. Which happens to be a load balancer created in the cloud provider. Run the kubectl command to retrieve the Load Balancer URL.


            kubectl get svc artifactory-artifactory-nginx -n tools

- ![Image8](https://github.com/user-attachments/assets/3d82d5c7-5806-4b2b-80a8-66c159db4a10)

4. Copy the URL and paste in the browser


- ![Image9](https://github.com/user-attachments/assets/2b156311-0529-4006-aba3-7bf17dd482a8)


5. The default username is _admin_
6. The default password is _password_

- ![Image10](https://github.com/user-attachments/assets/3cea9c21-1632-47f0-87b5-90b7bbe705ac)

## How the Nginx URL for Artifactory is configured in Kubernetes
Without clicking further on the **Get Started** page, lets dig a bit more into Kubernetes and Helm. How did Helm configure the URL in kubernetes?

Helm uses the values.yaml file to set every single configuration that the chart has the capability to configure. THe best place to get started with an off the shelve chart from artifacthub.io is to get familiar with the **DEFAULT VALUES**

- click on the DEFAULT VALUES section on Artifact hub

- ![Image11](https://github.com/user-attachments/assets/031507ca-860c-4848-a249-fdbb144c0803)


- Here you can search for key and value pairs

- ![Image12](https://github.com/user-attachments/assets/ae7ffc15-9b42-45e6-a27a-da57cb430d0e)

For example, when you type nginx in the search bar, it shows all the configured options for the nginx proxy.

- ![Image13](https://github.com/user-attachments/assets/668b5ffb-be3f-434b-a406-2e77969e0892)


- Selecting nginx.enabled from the list will take you directly to the configuration in the YAML file.

- ![Image14](https://github.com/user-attachments/assets/3102a906-310a-4700-96ba-94d23283da72)

- Search for nginx.service and select nginx.service.type

- ![Image15](https://github.com/user-attachments/assets/a6920616-7d4b-4dc5-ba68-ed0382907887)

- You will see the confired type of Kubernetes service for Nginx. As you can see, it is LoadBalancer by default


- ![Image16](https://github.com/user-attachments/assets/cea79748-b81f-4273-aec5-dd917171b647)


- To work directly with the values.yaml file, you can download the file locally by clicking on the download icon.

- ![Image17](https://github.com/user-attachments/assets/33b335d1-369e-4120-ba9f-a91ebd8545df)

## Is the Load Balancer Service type the Ideal configuration option to use in the Real World?
Setting the service type to **Load Balancer** is the easiest way to get started with exposing applications running in kubernetes externally. But provisioning load balancers for each application can become very expensive over time, and more difficult to manage. Especially when tens or even hundreds of applications are deployed.

The best approach is to use [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) instead. But to do that, we will have to deploy an [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

A huge benefit of using the ingress controller is that we will be able to use a single load balancer for different applications we deploy. Therefore, Artifactory and any other tools can reuse the same load balancer. Which reduces cloud cost, and overhead of managing multiple load balancers. more on that later.


For now, we will leave Artifactory, move on to the next phase of configuration (Ingress, DNS(Route53) and Cert Manager), and then return to Artifactory to complete the setup so that it can serve as a private docker registry and repository for private helm charts.





































































































































































































































