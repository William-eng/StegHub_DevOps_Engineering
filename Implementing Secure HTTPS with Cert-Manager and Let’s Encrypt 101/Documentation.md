![svgviewer-output](https://github.com/user-attachments/assets/2ea9a7da-286c-413d-8c53-914cff0adf15)# Implementing Secure HTTPS with Cert-Manager and Let’s Encrypt 101
In this project, we'll enhance the security of our Artifactory deployment by implementing HTTPS using [Cert-Manager](https://cert-manager.io) to request and manage TLS certificates from Let's Encrypt automatically. 
Cert-manager creates TLS certificates for workloads in your Kubernetes cluster and renews the certificates before they expire. This will provide a trusted HTTPS URL for our application.

**Prerequisites**:

- EKS Kubernetes cluster with Nginx Ingress Controller installed
- Helm 3.x installed
- Kubectl configured to interact with your cluster
- Domain name configured with DNS pointing to your Ingress Controller's load balancer
- Nginx Ingress Controller installed (from the previous project)

## Step 1: Install Cert-Manager

1. Add the Jetstack Helm repository:

          helm repo add jetstack https://charts.jetstack.io
   
2. Update your local Helm chart repository cache:

          helm repo update


- ![Image1](https://github.com/user-attachments/assets/4afcb5ce-cdc0-4763-bea6-ec154e74b8d6)


3. Follow the URL below to set up an EKS IAM role for a service account for cert-manager:
https://cert-manager.io/docs/configuration/acme/dns01/route53/#eks-iam-role-for-service-accounts-irsa

Steps for _3_ above include :

A. Retrieve IAM OIDC Provider for Your EKS Cluster

    aws eks describe-cluster --name <EKS_CLUSTER_NAME> --query "cluster.identity.oidc.issuer" --output text

- ![Image2](https://github.com/user-attachments/assets/e9fad379-73e5-4efb-a2d7-9ac69a44550b)

B. Create the IAM policy required by cert-manager for managing Route53 records. Save the following policy document in a file called cert-manager-policy.json

            {
             "Version": "2012-10-17",
             "Statement": [
                 {
                     "Effect": "Allow",
                     "Action": [
                         "route53:GetChange",
                         "route53:ChangeResourceRecordSets",
                         "route53:ListHostedZones",
                         "route53:ListResourceRecordSets"
                     ],
                        "Resource": "*"
                  }
               ]
            }



      aws iam create-policy --policy-name CertManagerRoute53Policy --policy-document file://cert-manager-policy.json


- ![Image3](https://github.com/user-attachments/assets/b7d60181-3c60-4497-a5ee-cfad5989ae82)


C. Create an IAM Role for the cert-manager ServiceAccount

-  In the IAM console, go to Roles and click on Create role.
-  Select trust entity and select Web Identity and below it, select the oidc of your eks cluster
-  Set the audience and click next.
- Search for the policy you created in Step 2 (CertManagerRoute53Policy) and select it.
- Create role.
- Now edit the trust policy for your newly created role and add the following below

- ![Image4](https://github.com/user-attachments/assets/ea901f85-3932-4a6c-a964-d338450ad88f)
- ![Image5](https://github.com/user-attachments/assets/8088e822-3747-4fa9-b71f-5054899af1d1)
- ![Image6](https://github.com/user-attachments/assets/7d202990-dcc4-4ca0-9898-9db7c62ea1e3)
- ![Image7](https://github.com/user-attachments/assets/807f4efa-808a-420e-a222-3cd957512f81)
- ![Image8](https://github.com/user-attachments/assets/e5f1d870-433b-4901-ac3e-0c34e23adaaa)


D. Create namespace cert-manager

      kubectl create namespace cert-manager

- ![Image9](https://github.com/user-attachments/assets/7c5401dc-c952-4b3e-b453-1aa2dc63fd64)

E. Annotate the cert-manager ServiceAccount in Kubernetes


      touch cert-manager-sa.yaml


          apiVersion: v1
          automountServiceAccountToken: true
          kind: ServiceAccount
          metadata:
          annotations:
             eks.amazonaws.com/role-arn: arn:aws:iam::312973238800:role/cert_manager_role
          name: cert-manager
          namespace: cert-manager
          
 Modify Cert-manager Deployment with correct file system permissions. This will allow the pod to read the ServiceAccount token.

          spec:
            template:
              securityContext:
                fsGroup: 1001
 
Replace with your actual AWS account ID.

- ![Image10](https://github.com/user-attachments/assets/e8a62261-458f-4478-bde2-3f43f72c799c)


F. Apply the manifest:

      kubectl apply -f cert-manager-sa.yaml

- ![Image11](https://github.com/user-attachments/assets/ed127f30-9af5-4cb7-a052-eab4c40ec666)

G. Configure RBAC for Cert-Manager ServiceAccount
- Create a new file cert-manager-rbac.yaml

 - ![Image12](https://github.com/user-attachments/assets/c2426a4b-6974-48be-a4b4-0e554629bee7)

- Apply the configuration

          kubectl apply -f cert-manager-rbac.yaml

- ![Image12](https://github.com/user-attachments/assets/c5a31a66-3b04-45b8-8cbd-ee0d2e896eb3)

- Add role AWS to your Kubernetes Service Account annotation.

H. Verify the ServiceAccount Configuration

          kubectl describe serviceaccount cert-manager -n cert-manager




- ![Image13](https://github.com/user-attachments/assets/306ef935-382b-4947-bb9f-9d07708b86d2)

4. Install Cert-Manager using Helm:

         helm install cert-manager jetstack/cert-manager \
            --namespace cert-manager \
            --create-namespace \
            --version v1.15.3 \
            --set crds.enabled=true

- ![Image14](https://github.com/user-attachments/assets/d755f936-9470-4608-a87a-2b64ca015f92)

- Modify Cert-manager Deployment with correct file system permissions. This will allow the pod to read the ServiceAccount token.

                    spec:
                      template:
                        securityContext:
                          fsGroup: 1001

                    helm upgrade --install cert-manager jetstack/cert-manager \
                      --namespace cert-manager \
                      --create-namespace \
                      --version v1.15.3 \
                      --set installCRDs=true \
                      --set securityContext.fsGroup=1001


  - ![Image15](https://github.com/user-attachments/assets/e7959e28-516e-4db3-ac8b-67a1eb747e7e)


5. Verify the Cert-Manager installation:

          kubectl get pods --namespace cert-manager

- ![Image16](https://github.com/user-attachments/assets/e6d89575-a138-48df-aa6b-c05f39b2aa3c)

- ![Image17](https://github.com/user-attachments/assets/f472fe93-ec41-4840-a3a8-6db4f3ea1ad2)

This diagram illustrates the main components of Cert-Manager:

- Cert-Manager Controller: Manages the certificate lifecycle
- Webhook: Validates and mutates resources
- CA Injector: Injects CA bundles into resources
- Custom Resource Definitions (CRDs): Define custom resources like Certificate, Issuer, and ClusterIssuer

## Step 2: Configure Let's Encrypt Issuer
1. Create a file named _lets-encrypt-issuer.yaml_ and add the following content:

                    apiVersion: cert-manager.io/v1
                    kind: ClusterIssuer
                    metadata:
                      name: letsencrypt-prod
                    spec:
                      acme:
                        server: https://acme-v02.api.letsencrypt.org/directory
                        email: devops@steghub.com
                        privateKeySecretRef:
                          name: letsencrypt-prod
                        solvers:
                        - selector:
                            dnsZones:
                              - "steghub.com"
                          dns01:
                            route53:
                              region: us-east-1
                              role: "arn:aws:iam::123456789012:role/cert_manager_role" # This must be set so cert-manager what role to attempt to authenticate with
                              auth:
                                kubernetes:
                                  serviceAccountRef:
                                    name: "cert-manager" # The name of the service account created




Replace devops@steghub.com with your actual email addres and the role ARN to you one you created. Update serviceAccount name if necessary.

- ![Image18](https://github.com/user-attachments/assets/e433c9cc-e203-4335-a681-0747bf706d44)

This ClusterIssuer is set up to use Let's Encrypt for SSL/TLS certificates, solving ACME challenges using DNS-01 with Amazon Route 53, and authenticating via a Kubernetes service account.


2. Apply the ClusterIssuer:
   
          kubectl apply -f lets-encrypt-issuer.yaml


- ![Image19](https://github.com/user-attachments/assets/2ce535a0-05d2-464e-be04-4895f6f580af)

3. Verify the ClusterIssuer:

   
          kubectl get clusterissuer


- ![Image20](https://github.com/user-attachments/assets/0af3f6e1-ffe2-4720-a8fc-642391f2ab1a)

## Step 3: Update Ingress for Artifactory

1. Update your Artifactory Ingress to include TLS configuration:

                    apiVersion: networking.k8s.io/v1
                    kind: Ingress
                    metadata:
                      name: artifactory-ingress
                      namespace: tools
                      annotations:
                        nginx.ingress.kubernetes.io/proxy-body-size: 500m
                        service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
                        service.beta.kubernetes.io/aws-load-balancer-type: nlb
                        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: ssl
                        service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
                        cert-manager.io/cluster-issuer: letsencrypt-prod
                        cert-manager.io/private-key-rotation-policy: Always
                      labels:
                        name: artifactory
                    spec:
                      ingressClassName: nginx
                      tls:
                      - hosts:
                        - tooling.artifactory.steghub.com
                        secretName: tooling.artifactory.steghub.com
                      rules:
                      - host: tooling.artifactory.steghub.com
                        http:
                          paths:
                          - path: /
                            pathType: Prefix
                            backend:
                              service:
                                name: artifactory
                                port:
                                  number: 8082



Please make a note of the following information:

- metadata.annotations: We have a cert-manager cluster issuer annotation, which points to the ClusterIssuer we previously created. If Cert-Manager observes an Ingress with annotations described in the [Supported Annotations](https://cert-manager.io/docs/usage/ingress/#supported-annotations) section, it will ensure that a Certificate resource with the name provided in the _tls.secretName_ field and configured as described on the Ingress exists in the namespace.

- spec.tls: We added the tls block, which determines what ends up in the cert's configuration. The tls.hosts will be added to the cert's subjectAltNames.

- proxy-body-size: This is used to set the maximum size of a file we want to allow in our Artifactory. The initial value is lower, so you won't be able to upload a larger size artifact unless you add this annotation. A 413 error will be returned when the size in a request exceeds the maximum allowed size of the client request body.



2. Apply the updated Ingress:
   
          kubectl apply -f artifactory-ingress.yaml -n tools


- ![Image21](https://github.com/user-attachments/assets/df55b087-5f9e-42be-99a5-2f3c3e00925a)




















































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































