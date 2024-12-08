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


# DEPLOYING INGRESS CONTROLLER AND MANAGING INGRESS RESOURCES

Before we discuss what **ingress** controllers are, it will be important to start off understanding about the Ingress resource.

An ingress is an API object that manages external access to the services in a kubernetes cluster. It is capable to provide load balancing, SSL termination and name-based virtual hosting. In otherwords, Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

Here is a simple example where an Ingress sends all its traffic to one Service:

- ![Image18](https://github.com/user-attachments/assets/931ebbf9-d214-4bfe-aa76-885dd721e820)

An ingress resource for Artifactory would like below

               apiVersion: networking.k8s.io/v1
               kind: Ingress
               metadata:
                 name: artifactory-ingress
                 namespace: tools
                 labels:
                   name: artifactory-ingress
               spec:
                 ingressClassName: nginx
                 rules:
                 - host: tooling.artifactory.steghub.com
                   http:
                     paths:
                     - pathType: Prefix
                       path: "/"
                       backend:
                         service:
                           name: artifactory-artifactory-nginx
                           port: 
                             number: 80



- An Ingress needs apiVersion, kind, metadata and spec fields
- The name of an Ingress object must be a valid DNS subdomain name
- Ingress frequently uses annotations to configure some options depending on the Ingress controller.
- Different Ingress controllers support different annotations. Therefore it is important to be up to date with the ingress controller’s specific documentation to know what annotations are supported.
- It is recommended to always specify the ingress class name with the spec _ingressClassName: nginx_. This is how the Ingress controller is selected, especially when there are multiple configured ingress controllers in the cluster.
- The domain _steghub.com_ should be replaced with your own domain.

## Ingress controller
If you deploy the yaml configuration specified for the ingress resource without an ingress controller, it will not work. In order for the Ingress resource to work, the cluster must have an ingress controller running.

Unlike other types of controllers which run as part of the kube-controller-manager. Such as the **Node Controller**, **Replica Controller**, **Deployment Controller**, **Job Controller**, or **Cloud Controller**. Ingress controllers are not started automatically with the cluster.

Kubernetes as a project supports and maintains [AWS](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/), [GCE](https://github.com/kubernetes/ingress-gce/blob/master/README.md#readme), and [NGINX](https://github.com/kubernetes/ingress-nginx/blob/main/README.md#readme) ingress controllers.

There are many other 3rd party Ingress controllers that provide similar functionalities with their own unique features, but the 3 mentioned earlier are currently supported and maintained by Kubernetes. Some of these other 3rd party Ingress controllers include but not limited to the following;

- [AKS Application Gateway Ingress Controller (Microsoft Azure)](https://docs.microsoft.com/en-gb/azure/application-gateway/tutorial-ingress-controller-add-on-existing)
- [Istio](https://istio.io/latest/docs/tasks/traffic-management/ingress/kubernetes-ingress/)
- [Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/)
- [Ambassador](https://www.getambassador.io/)
- [HA Proxy Ingress](https://haproxy-ingress.github.io/)
- [Kong](https://docs.konghq.com/kubernetes-ingress-controller/)
- [Gloo](https://docs.solo.io/gloo-edge/latest/)

An example comparison matrix of some of the controllers can be found [here](https://kubevious.io/blog/post/comparing-top-ingress-controllers-for-kubernetes#comparison-matrix). Understanding their unique features will help businesses determine which product works well for their respective requirements.

It is possible to deploy any number of ingress controllers in the same cluster. That is the essence of an **ingress class**. By specifying the spec ingressClassName field on the ingress object, the appropriate ingress controller will be used by the ingress resource.

Lets get into action and see how all of these fits together.


## Deploy Nginx Ingress Controller
In this project, we will deploy and use the Nginx Ingress Controller. It is always the default choice when starting with Kubernetes projects. It is reliable and easy to use.

Since Kubernetes maintain this controller, there is an official guide to the installation process. Hence, we won't be using artifacthub.io here. Even though you can still find ready-to-go charts
there, it just makes sense to always use the [official guide](https://kubernetes.github.io/ingress-nginx/) in this scenario.

Using the Helm approach, according to the official guide;

1. Install Nginx Ingress Controller in the ingress-nginx namespace


               helm upgrade --install ingress-nginx ingress-nginx \
               --repo https://kubernetes.github.io/ingress-nginx \
               --namespace ingress-nginx --create-namespace

- ![Image16](https://github.com/user-attachments/assets/42d49b7c-01b9-4937-9603-a4e2f9f30a14)



**Notice**:

This command is idempotent:

- if the ingress controller is not installed, it will install it,
- if the ingress controller is already installed, it will upgrade it.


**Self Challenge Task** – Delete the installation after running the above command. Then try to re-install it using a slightly different method you are already familiar with. Ensure NOT to use the flag --repo
**Hint** – Run the helm repo add command before installation

- A few pods should start in the ingress-nginx namespace:

         kubectl get pods --namespace=ingress-nginx

- ![Image17](https://github.com/user-attachments/assets/cb528777-9354-4868-959c-b718eaed4393)


- Delete the installed ingress-nginx release:


      helm uninstall ingress-nginx -n ingress-nginx

        kubectl delete namespace ingress-nginx


- Add Helm Repository
- Add the ingress-nginx Helm repository (this will replace the --repo flag usage):


            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update

- ![Image18](https://github.com/user-attachments/assets/36879689-1898-4a4e-8ff6-051743017b7c)

- Reinstall ingress-nginx using the repository name instead of the --repo flag:


         helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
           --namespace ingress-nginx --create-namespace

- ![Image19](https://github.com/user-attachments/assets/144d01ae-5a22-4d5d-8321-ba4b5510a1b3)

- A few pods should start in the ingress-nginx namespace:

         kubectl get pods --namespace=ingress-nginx


- ![Image20](https://github.com/user-attachments/assets/e66ac627-1b72-4e39-9aec-69dfcc4d8529)


3. After a while, they should all be running. The following command will wait for the ingress controller pod to be up, running, and ready:


            kubectl wait --namespace ingress-nginx \
              --for=condition=ready pod \
              --selector=app.kubernetes.io/component=controller \
              --timeout=120s



4. Check to see the created load balancer in AWS.

         kubectl get service -n ingress-nginx
**Output**:

         NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP                                                                 PORT(S)                      AGE
         ingress-nginx-controller             LoadBalancer   172.20.52.252   aa2d0568b5b1e4a3db37301e6c82bc2f-294794177.eu-central-1.elb.amazonaws.com   80:31882/TCP,443:30956/TCP   36m
         ingress-nginx-controller-admission   ClusterIP      172.20.165.95   <none>                                                                      443/TCP       


- ![Image21](https://github.com/user-attachments/assets/b5f27f14-0348-409d-aa17-2276abace9e3)


The _ingress-nginx-controller service_ that was created is of the type _LoadBalancer_. That will be the load balancer to be used by all applications which require external access and are using this ingress controller.

5. Check the IngressClass that identifies this ingress controller.

         kubectl get ingressclass -n ingress-nginx
**Output**:

         NAME    CONTROLLER             PARAMETERS   AGE
         nginx   k8s.io/ingress-nginx   <none>       36m

- ![Image22](https://github.com/user-attachments/assets/04611d47-0036-4620-af4c-9ed15169d187)

## Deploy Artifactory Ingress
Now, it is time to configure the ingress so that we can route traffic to the Artifactory internal service, through the ingress controller’s load balancer.

Notice the _spec_ section with the configuration that selects the Nginx ingress controller using the **ingressClassName**, and we have scheme annotation set as _internet-facing_


             apiVersion: networking.k8s.io/v1
              kind: Ingress
              metadata:
                name: artifactory-ingress
                namespace: tools
                annotations:
                  service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
                  service.beta.kubernetes.io/aws-load-balancer-type: nlb
                  service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
                labels:
                  name: artifactory
              spec:
                ingressClassName: nginx
                rules:
                - host: tooling.artifactory.steghub.com
                  http:
                    paths:
                    - path: /
                      pathType: Prefix
                      backend:
                        service:
                          name: artifactory-artifactory-nginx
                          port:
                            number: 8082



- ![Image23](https://github.com/user-attachments/assets/c64d46d2-44ac-46c3-9d17-5edaadac7b08)


      kubectl apply -f <filename.yaml> -n tools

**Output**:

         NAME                 CLASS   HOSTS                                   ADDRESS                                                                  PORTS   AGE
         artifactory-ingress   nginx   tooling.artifactory.steghub.com   k8s-tools-artifact-0518ebb10a-2014569128.eu-central-1.elb.amazonaws.com   80      5s


- ![Image24](https://github.com/user-attachments/assets/52446b95-fc11-4e00-9f2e-2f81630537f4)


Now, take note of

CLASS – The Nginx controller class name nginx
HOSTS – The hostname to be used in the browser tooling.artifactory.steghub.com
ADDRESS – The load balancer address that was created by the ingress controller


## Configure DNS
If anyone were to visit the tool, it would be very inconvenient to share the long load balancer address. Ideally, you would create a DNS record that is human-readable and can direct requests to the balancer. This is exactly what has been configured in the ingress object - _host: "tooling.artifactory.steghub.com"_ but without a DNS record, there is no way that the host address can reach the load balancer.

The _sandbox.svc.steghub.com_ part of the domain is the configured **HOSTED ZONE** in AWS. So you will need to configure Hosted Zone in the AWS console or as part of your infrastructure as code using Terraform.

If you purchased the domain directly from AWS, the hosted zone will be automatically configured for you. But if your domain is registered with a different provider such as freenon or namechaep, you will have to create the hosted zone and update the name servers.

## Create Route53 record
Within the hosted zone is where all the necessary DNS records will be created. Since we are working on Artifactory, lets create the record to point to the ingress controller’s loadbalancer. There are 2 options. You can either use the CNAME or AWS Alias

## CNAME Method

1. Select the **HOSTED ZONE** you wish to use, and click on the create record button
2. Add the subdomain tooling.artifactory, and select the record type CNAME
3. Successfully created record
4. Confirm that the DNS record has been properly propagated. Visit https://dnschecker.org and check the record. Ensure to select CNAME. The search should return green ticks for each of the locations on the left.

## AWS Alias Method

1. In the create record section, type in the record name, and toggle the _alias_ button to enable an alias. An alias is of the A DNS record type which basically routes directly to the load balancer. In the _choose endpoint_ bar, select _Alias_ to _Application_ and _Classic Load Balancer_
2. Select the region and the load balancer required. You will not need to type in the load balancer, as it will already populate.
For detailed read on selecting between CNAME and Alias based records, read the [official documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html)

## Visiting the application from the browser

So far, we now have an application running in Kubernetes that is also accessible externally. That means if you navigate to https://tooling.artifactory.steghub.com/ _(replace the full URL with your domain), it should load up the Artifactory application._

Using Chrome browser will show something like the below. It shows that the site is indeed reachable, but insecure. It is insecure because it either does not have a trusted TLS/SSL certificate, or it doesn’t have any at all.

Nginx Ingress Controller does configure a default TLS/SSL certificate. However, it is not trusted because it is a self-signed certificate that browsers are not aware of.

To confirm this,

1. Click on the Not Secure part of the browser.
2. Select the Certificate is not valid menu
3. You will see the details of the certificate. There you can confirm that yes indeed there is encryption configured for the traffic, the browser is just not cool with it.
































































































































