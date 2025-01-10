# Streamlining Tooling Application Deployments with Helm, Kustomize, and Vault 102
This project provides a practical guide to deploying and managing a "tooling application" (replace with your specific application context) in Kubernetes using a robust combination of Helm, Kustomize, and Vault.
Based on our experience on previous projects, you should already have a grasp of experience using Helm, but you should also know that there are multiple choices available when it comes to deploying applications into
Kubernetes, so Helm should not always be your default choice. It is important to be aware of the options available, know the pros and cons, and choose what works for your team or project.
Before we make a final decision on how we will realistically manage deployments into Kubernetes, let's see how Kustomize works.

# How Kustomize Works
**Kustomize** uses a file called _kustomization.yaml_ that contains declarative specifications as to what resources need to be imported from what manifest files and what changes need to be made. Once it has processed the
resources, it emits them to the standard output, which can be stored in a file or directly used with _kubectl_ to apply it to a particular cluster.

One of the excellent use cases of Kustomize is to manage Kubernetes resources for multiple environments. For Kustomize to work in that scenario, you would need a base directory that would contain all manifest files with all 
the common elements and an overlays directory that contains all the differences for a particular environment.

To understand better, let’s look at a hands-on example.

# Installing Kustomize
Kustomize comes pre-bundled with kubectl version >= 1.14. You can check your version using _kubectl version_. If the kubectl version is 1.14 or greater there's no need to take any steps.

For a stand-alone Kustomize installation (aka Kustomize CLI), go through the official documentation to install [Kustomize - Here](https://kubectl.docs.kubernetes.io/installation/kustomize/)

# Working with Kustomize
Kustomize is a command line tool supporting template-free, structured customization of declarative configuration targeted to k8s-style objects.

Targeted to k8s means that _kustomize_ has some understanding of API resources, k8s concepts like names, labels, namespaces, etc., and the semantics of resource patching.

Kustomize is an implementation of [Declarative Application Management (DAM)](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#declarative-application-management). A set of best practices around managing k8s clusters.
It is a configuration management solution that leverages layering to preserve the base settings of your applications and components by overlaying declarative YAML artifacts (called patches) that selectively override default settings without actually changing the original files (base files).

Kustomize relies on the following system of configuration management layering to achieve reusability:

- **Base Layer** - Specifies the most common resources
- **Patch Layers** - Specifies use of case-specific resources

Let’s step through how Kustomize works using a deployment scenario involving 3 different environments: **dev**, **sit**, and **prod**. In this example, we’ll use Service, Deployment, and Namespace resources. For the 
dev environment, there won't be any specific changes as it will use the same configuration from the base setting. In sit and prod environments, the replica settings will be different.

Using the tooling app for this example, create a folder structure as below.

## Project Structure


            └── tooling-app-kustomize
                ├── base
                │   ├── deployment.yaml
                │   ├── kustomization.yaml
                │   └── service.yaml
                └── overlays
                    ├── dev
                    │   ├── deployment.yaml
                    │   ├── kustomization.yaml
                    │   └── namespace.yaml
                    ├── prod
                    │   ├── deployment.yaml
                    │   ├── kustomization.yaml
                    │   └── namespace.yaml
                    └── sit
                        ├── deployment.yaml
                        ├── kustomization.yaml
                        └── namespace.yaml

- ![Image00](https://github.com/user-attachments/assets/4f54639b-5dba-40c0-af14-35184d6a2db2)


Now, let's walk through the content of each file.

# Base directory

## deployment.yaml

          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: tooling-deployment
            labels:
              app: tooling
          spec:
            replicas: 1
            selector:
              matchLabels:
                app: tooling
            template:
              metadata:
                labels:
                  app: tooling
              spec:
                containers:
                - name: tooling
                  image: steghub/tooling-app:1.0.2
                  ports:
                  - containerPort: 80
                  resources:
                    requests:
                      memory: "64Mi"
                      cpu: "250m"
                    limits:
                      memory: "128Mi"
                      cpu: "500m"


- ![Image01](https://github.com/user-attachments/assets/cf78e038-997c-403a-b2c2-f3ada77b441b)


### service.yaml

            apiVersion: v1
            kind: Service
            metadata:
              name: tooling-service
              labels:
                app: tooling
            spec:
              ports:
              - port: 80
                protocol: TCP
                targetPort: 80
                name: http
              type: ClusterIP
              selector:
                app: tooling


- ![Image03](https://github.com/user-attachments/assets/c722dc06-c02c-42dd-90a9-ca6e9f1a7e9a)

### kustomization.yaml
        
          apiVersion: kustomize.config.k8s.io/v1beta1
          kind: Kustomization
          resources:
          - deployment.yaml
          - service.yaml


- ![Image04](https://github.com/user-attachments/assets/6d5478f6-b7a8-45b3-9321-f37eabdc77c9)

The resources being monitored here are **deployment** and **services**. You can simply add more to the list as you wish.

It is assumed that we will need to deploy Kubernetes resources across multiple environments, as a standard practice in most cases. Hence, to deploy resources into the Dev environment, let's see what the layout and file contents will look like.

# DEV Environment
In the **overlays** folder - This is where you manage multiple environments. In each environment, there is a Kustomize file that tells Kustomize where to find the base setting, and how to **patch** the environment using the **base** as the starting point.


In the **dev** environment for example, the namespace for dev is created, and the deployment is patched to use a replica count of "**3**" different from the base setting of "**1**". So Kustomize will simply create all the resources in the base, in addition to whatever is specified in the dev directory. We will discuss patching a little further in the following section.

Let's have a look at what each file contains.

### tooling-app-kustomize/overlays/dev/namespace.yaml

      apiVersion: v1
      kind: Namespace
      metadata:
        name: dev-tooling

- ![Image05](https://github.com/user-attachments/assets/bcfed32d-5bf3-4c58-bf5e-8ed064092905)

### tooling-app-kustomize/overlays/dev/deployment.yaml


        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: tooling-deployment
        spec:
          replicas: 3


- ![Image06](https://github.com/user-attachments/assets/7e37b327-7a5e-4bf2-91bd-08fd39f0743c)

### tooling-app-kustomize/overlays/dev/kustomization.yaml

      apiVersion: kustomize.config.k8s.io/v1beta1
      kind: Kustomization
      namespace: dev-tooling
      labels:
      - pairs:
          env: dev-tooling
      resources:
      - ../../base
      - namespace.yaml
The Kustomization file for dev here specifies that the base configuration should be applied, and include the YAML file(s) specified in the resources section. It also indicates what namespace the configuration should be applied to.


In summary, it specifies the following:

- The apiVersion
- The Kind of resource (Kustomization)
- The namespace where this Kustomizaton will create or patch resources
- The location of the base folder, where the base configuration can be found.
- The resource(s) to be created - Such as a namespace or deployment
- The labels field ensures that Kubernetes labels and selectors are automatically injected into the resources being created. such as below;
Generally, A Kustomization file contains fields falling into four categories (although not all have been used in the example above):

**resources** - what existing resources are to be customized. Example fields: k8s resources, crds.
**generators** - what new resources should be created. Example fields: configMapGenerator (legacy), secretGenerator (legacy), generators (v2.1).
**transformers** - what to do to the aforementioned resources. Example fields: namePrefix, nameSuffix, images, labels, etc., and the more general transformers (v2.1) field.
**meta** - fields which may influence all or some of the above. Example fields: vars, namespace, apiVersion, kind, etc.

# Patching configuration with Kustomize

With Kustomize, you can now begin to patch your environments with extra configurations that overwrite the base setting either by

- creating new resources, or
- patching existing resources.

This is all achieved through the overlays configuration. The overlays/dev/kustomization.yaml example above only creates a new resource. What if we wanted to patch an existing resource? For example, increase the pod replica from the default 1 to 3 as shown in the overlays/dev/deployment.yaml file

## Applying Configurations
To apply the configuration:

            kubectl apply -k overlays/dev

**Output**:

            namespace/dev-tooling created
            service/tooling-service created
            deployment.apps/tooling-deployment created


- ![Image07](https://github.com/user-attachments/assets/9afcd15c-94a9-4c4c-8970-b0f131ba480e)

# Side Tasks
With your understanding of how kustomize can patch resources per environment, now configure both SIT and PROD environments with their respective overlays and set different configuration values for

- Pod replica
- Resource limit and requests
- Image tag

### prod/kustomization.yaml

            apiVersion: kustomize.config.k8s.io/v1beta1
            kind: Kustomization
            namespace: prod-tooling
            commonLabels:
                  env: prod-tooling
            resources:
              - ../../base
              - namespace.yaml


### prod/deployment.yaml

              selector:
                matchLabels:
                  app: tooling
              template:
                metadata:
                  labels:
                    app: tooling
                spec:
                  containers:
                  - name: tooling
                    image: willywan/toolingapp:test-0.0.4
                    ports:
                    - containerPort: 80
                    resources:
                      requests:
                        memory: "64Mi"
                        cpu: "250m"
                      limits:
                        memory: "128Mi"
                        cpu: "500m"

### prod/namespace.yaml


            apiVersion: v1
            kind: Namespace
            metadata:
              name: prod-tooling



### sit/deployment.yaml

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: tooling-deployment  
            spec:
              replicas: 3
              selector:
                matchLabels:
                  app: tooling
              template:
                metadata:
                  labels:
                    app: tooling
                spec:
                  containers:
                  - name: tooling
                    image: willywan/toolingapp:test-0.0.4
                    ports:
                    - containerPort: 80
                    resources:
                      requests:
                        memory: "64Mi"
                        cpu: "250m"
                      limits:
                        memory: "128Mi"
                        cpu: "500m"



### sit/kustomization.yaml

            apiVersion: kustomize.config.k8s.io/v1beta1
            kind: Kustomization
            namespace: sit-tooling
            commonLabels:
                  env: sit-tooling
            resources:
              - ../../base
              - namespace.yaml

### sit/namespaces.yaml

            apiVersion: v1
            kind: Namespace
            metadata:
              name: sit-tooling






## Applying Configurations

To apply the configuration:

            kubectl apply -k overlays/prod
            kubectl apply -k overlays/sit



- ![Image08](https://github.com/user-attachments/assets/f655084a-7d5f-4074-a9fe-545f1f7df5c1)


# Helm Template Engine vs. Kustomize Overlays
Both technologies have good reasons why they are designed the way they are. But most of the experienced engineers in the industry would rather get the best of both worlds.


With helm, you can simply install already packaged applications from [artifacthub.io](https://www.artifacthub.io/), and then use Kustomize to patch its values files.

For business applications, you can choose to package your applications in Helm and simply patch the values files with Kustomize as well. But you might also just use Helm only for applications you wish to download from the public and use Kustomize directly for business applications.

## Integrate the tooling app with Amazon Aurora for Dev, SIT, and PROD environments

- Configure Terraform to deploy an Amazon Aurora instance.
- Use the tooling.sql file in the tooling repo to load the database schema.
- Add K8s Secret resource to store database credentials.
- Configure environment variables for database connectivity in the deployment file and patch each environment for the appropriate values.

## Steps Involved
1. create a new terraform directory _terraform_

## Project Structure

            terraform
            ├── main.tf
            ├── providers.tf
            ├── variables.tf
            ├── output.tf


## terraform/main.tf

            resource "aws_db_subnet_group" "aurora_subnet_group" {
              name       = "aurora-subnet-group"
              subnet_ids = var.subnet_ids  
              description = "Subnet group for Aurora cluster"
            }
            
            resource "aws_security_group" "aurora_sg" {
              name        = "aurora-sg"
              description = "Allow access to Aurora from EKS cluster"
              vpc_id      = var.vpc_id 
            
              
              ingress {
                from_port   = 3306
                to_port     = 3306
                protocol    = "tcp"
                cidr_blocks = ["0.0.0.0/0"]  
              }
            
            
              egress {
                from_port   = 0
                to_port     = 0
                protocol    = "-1"
                cidr_blocks = ["0.0.0.0/0"]
              }
            }
            
            resource "aws_rds_cluster" "aurora_cluster" {
              cluster_identifier   = "tooling-aurora-cluster"
              engine               = "aurora-mysql"   
              engine_version       = "8.0.mysql_aurora.3.05.2"
              master_password      = var.db_password
              master_username      = var.db_username
              database_name        = var.db_name
              skip_final_snapshot  = true
            
            
              storage_encrypted    = true
            
              db_subnet_group_name     = aws_db_subnet_group.aurora_subnet_group.name
              vpc_security_group_ids   = [aws_security_group.aurora_sg.id]
            }
            
            resource "aws_rds_cluster_instance" "aurora_instance" {
              count                = 2                  
              cluster_identifier   = aws_rds_cluster.aurora_cluster.id
              instance_class       = var.db_instance_class
              engine               = "aurora-mysql"    
              publicly_accessible  = true
            }



## terraform/providers.tf

            terraform {
              required_providers {
                aws = {
                  source  = "hashicorp/aws"
                  version = "5.74.0"
                }
              }
            }
            
            provider "aws" {
              region = "us-west-1"
            }


## terraform/output.tf

            output "aurora_endpoint" {
              value = aws_rds_cluster.aurora_cluster.endpoint
            }


## terraform/variables.tf

            variable "db_instance_class" {
              default = "db.r7g.large"
            }
            
            variable "db_name" {
              default = "tooling_db"
            }
            
            variable "db_username" {
              default = "admin"
            }
            
            variable "db_password" {
              default = "<db-password>"
              sensitive = true
            }
            
            variable "vpc_id" {
              default = "vpc-0e65f071d86a07a0c"
              type        = string
            }
            
            variable "subnet_ids" {
              description = "List of subnet IDs for Aurora cluster"
              type        = list(string)
              default     = ["subnet-0f78d1558d12f5798", "subnet-08795fae9110802f6"]  
            }





Go ahead and initialize the terraform project, as well as run terraform apply.

            terraform init
            terraform validate
            terraform plan 
            terraform apply -auto-approve

- ![Image09](https://github.com/user-attachments/assets/8b62ff7b-5b45-4c0f-b0a0-1f2f61b5e907)

Confirm from AWS console

- ![Image10](https://github.com/user-attachments/assets/e0cf75b4-c15e-49ef-b223-13e4d662271d)


## Step 2: Create a bash script to automate loading of the database schema.

            touch load_tooling_db.sh
            chmod +x load_tooling_db.sh



### Run The Script

     ./load_tooling_db.sh
     

- ![Image11](https://github.com/user-attachments/assets/83f199c4-e561-4eed-a179-69387665bba8)


confirm that the database has been imported

            sudo mysql -h <aurora-db-endpoint> -u <db-username> -p
            SHOW DATABASES;


- ![Image12](https://github.com/user-attachments/assets/7a8ede80-efef-4883-9cfc-7e7ec4b1b43b)

- ![Imae13](https://github.com/user-attachments/assets/4e068021-1e60-429e-b95b-68fadd0f90c0)

3. Create a Kubernetes secret in each environment (e.g., Dev, SIT, PROD) to store the database credentials.
   
- first, encode your db username and password and copy the encoded output.

              echo -n "your-username" | base64
              echo -n "your-password" | base64


### overlays/dev/database-secret.yaml

            apiVersion: v1
            kind: Secret
            metadata:
              name: aurora-db-credentials
              namespace: dev-tooling
            type: Opaque
            data:
              DB_USERNAME: <base64-encoded-username>
              DB_PASSWORD: <base64-encoded-password>


- Repeat same thing for SIT and PROD env.

- For each environment, deploy the secret to Kubernetes using:

            kubectl apply -f overlays/dev/database-secret.yaml
            kubectl apply -f overlays/sit/database-secret.yaml
            kubectl apply -f overlays/prod/database-secret.yaml


- ![Image14](https://github.com/user-attachments/assets/eb4ed89a-f4b3-4597-9876-2428d7e3cd48)


4. patch the different env deployment.yaml to include environment variables for the database connection, using the values stored in the Kubernetes Secret.

### base/deployment.yaml

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: tooling-deployment
              labels:
                app: tooling
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: tooling
              template:
                metadata:
                  labels:
                    app: tooling
                spec:
                  containers:
                    - name: tooling
                      image: steghub/tooling-app:1.0.2
                      ports:
                        - containerPort: 80
                      resources:
                        requests:
                          memory: "64Mi"
                          cpu: "250m"
                        limits:
                          memory: "128Mi"
                          cpu: "500m"
                      env:
                        - name: DB_HOST
                          value: "db-endpoint"  
                        - name: DB_PORT
                          value: "3306"
                        - name: DB_NAME
                          value: "your-database-name"        
                        - name: DB_USERNAME
                          valueFrom:
                            secretKeyRef:
                              name: aurora-db-credentials    
                              key: DB_USERNAME              
                        - name: DB_PASSWORD
                          valueFrom:
                            secretKeyRef:
                              name: aurora-db-credentials
                              key: DB_PASSWORD  
            



- Repeat same thing for overlay deployment.yaml

**Apply the deployment**

            kubectl apply -k base
            kubectl apply -k overlays/dev
            kubectl apply -k overlays/prod
            kubectl apply -k overlays/sit


- ![Image15](https://github.com/user-attachments/assets/936eafc3-fe3d-4727-92e6-d6ad770f1f1b)

  

## Integrate Vault with Kubernetes
Before we integrate Vault with our Kubernetes cluster, we will have will use Helm and Kustomize for the installation. The [vault helm chart](https://artifacthub.io/packages/helm/hashicorp/vault) is the recommended way to install Vault in a Kubernetes cluster and configure it. In this project, we will configure Vault to use [High Availability Mode](https://www.vaultproject.io/docs/concepts/ha) with **Integrated storage (Raft)**, This is recommended for production-ready deployment. This installs a StatefulSet of Vault server Pods with either Integrated Storage or a Consul storage backend. The Vault Helm chart can also configure Vault to run in standalone mode or [dev](https://www.vaultproject.io/docs/concepts/dev-server).

Create the folder structure as below:

            vault
            ├── base
            │   ├── kustomization.yaml
            │   └── namespace.yaml
            └── overlays
                ├── dev
                │   ├── .env
                │   ├── kustomization.yaml
                │   ├── namespace.yaml
                │   └── values.yaml
                ├── sit
                │   ├── .env
                │   ├── kustomization.yaml
                │   ├── namespace.yaml
                │   └── values.yaml
                └── prod
                    ├── .env
                    ├── kustomization.yaml
                    ├── namespace.yaml
                    └── values.yaml

## Base Files

### namespace.yaml

            apiVersion: v1
            kind: Namespace
            metadata:
              name: vault


### kustomization.yaml

            apiVersion: kustomize.config.k8s.io/v1beta1
            kind: Kustomization
            resources:
            - "namespace.yaml"


## Dev Environment Files

### namespace.yaml

            apiVersion: v1
            kind: Namespace
            metadata:
              name: vault
              labels:
                env: vault-dev


### kustomization.yaml

            apiVersion: kustomize.config.k8s.io/v1beta1
            kind: Kustomization
            namespace: vault
            resources:
            - ../../base
            patches:
            - path: namespace.yaml
            helmCharts:
            - name: vault
              namespace: vault
              repo: https://helm.releases.hashicorp.com
              releaseName: vault
              version: 0.28.1
              valuesFile: values.yaml
            secretGenerator:
            - name: vault-kms
              env: .env
            generatorOptions:
              disableNameSuffixHash: true



### .env

            VAULT_SEAL_TYPE=awskms
            VAULT_AWSKMS_SEAL_KEY_ID=<arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab>
            

**Note**: Replace the kms key arn with the one created by terraform


## Terraform Configuration

### file Structure

            terraform
            ├── main.tf
            ├── providers.tf
            ├── variables.tf



### providers.tf

            terraform {
              required_providers {
                aws = {
                  source  = "hashicorp/aws"
                  version = "5.74.0"
                }
              }
            }
            
            provider "aws" {
              region = "us-west-1"
            }


### main.tf

            module "vault_iam_role" {
              source      = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
              version     = "5.47.1"
              role_name   = "vaultKMS"
              create_role = true
              role_policy_arns = {
                AWSKeyManagementServicePowerUser = "arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser"
              }
            
              oidc_providers = {
                main = {
                  provider_arn               = "arn:aws:iam::012345678901:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/5C54DDF35ER19312844C7333374CC09D"
                  namespace_service_accounts = ["vault:vault-kms"]
                }
              }
            
              tags = var.tags
            }
            
            module "vault_kms_key" {
              source                  = "terraform-aws-modules/kms/aws"
              version                 = "3.1.1"
              description             = "Vault Cluster KMS Key"
              deletion_window_in_days = 7
              enable_key_rotation     = true
              is_enabled              = true
              key_usage               = "ENCRYPT_DECRYPT"
              multi_region           = false
            
              enable_default_policy = true
              key_owners           = [module.vault_iam_role.iam_role_arn]
              key_administrators   = [module.vault_iam_role.iam_role_arn]
            
              aliases                 = ["dev-vault-kms"]
              aliases_use_name_prefix = true
            
              grants = {}
            
              tags = var.tags
            }



### variables.tf

            variable "tags" {
              type = map(any)
              default = {
                Terraform   = "true"
                Environment = "dev"
              }
            }



To initialize and apply Terraform configuration:

            cd terraform
            terraform init
            terraform plan -out tfplan
            terraform apply tfplan

- ![Imag16](https://github.com/user-attachments/assets/569a0a7d-9cd4-4f9b-bef1-c8ea509277d2)

Before we add the content of the values file, we need to install the Ingress Controller and Cert-Manager. If you don't have those tools installed in your cluster, you can reference the last two projects for this.

- Ingress Controller: For this ingress controller you can use the [Nginx ingress controller](https://kubernetes.github.io/ingress-nginx/deploy/) helm chart for the installation. You can deploy the ingress controller with the following command:

  
            helm upgrade --install ingress-nginx ingress-nginx \
              --repo https://kubernetes.github.io/ingress-nginx \
              --namespace ingress-nginx --create-namespace


  This will install the controller in the ingress-nginx namespace, creating that namespace if it doesn't already exist.

- Cert-Manager: This is a Kubernetes addon to automate the management and issuance of TLS certificates from various issuing sources. It will ensure certificates are valid and up to date periodically, and attempt to renew certificates at an appropriate time before expiry. Visit the [last project documentation](https://github.com/William-eng/StegHub_DevOps_Engineering/blob/main/Implementing%20Secure%20HTTPS%20with%20Cert-Manager%20and%20Let%E2%80%99s%20Encrypt%20101/Documentation.md) for the installation.

- ![Image17](https://github.com/user-attachments/assets/ffeede11-b587-4632-a88c-c0640f7b7af6)

After the installation of the Cert-manager and Ingress controller, the next step is to configure the Vault cluster from the values file and then deploy it.

_vault/overlays/dev/values.yaml_


            server:
              enabled: "-"
              image:
                repository: "hashicorp/vault"
                tag: "1.17.2"
                # Overrides the default Image Pull Policy
                pullPolicy: IfNotPresent
            
              # Configure the Update Strategy Type for the StatefulSet
              updateStrategyType: "OnDelete"
            
              # Ingress allows ingress services to be created to allow external access
              # from Kubernetes to access Vault pods.
              ingress:
                enabled: true
                labels: {}
                annotations:   
                  nginx.ingress.kubernetes.io/proxy-body-size: 500m
                  service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
                  service.beta.kubernetes.io/aws-load-balancer-type: nlb
                  service.beta.kubernetes.io/aws-load-balancer-backend-protocol: ssl
                  service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
                  cert-manager.io/cluster-issuer: letsencrypt-prod
                  cert-manager.io/private-key-rotation-policy: Always
                  
                ingressClassName: "nginx"
                pathType: Prefix
            
                activeService: true
                hosts:
                  - host: "tooling.vault.steghub.com"
                    paths: [/]
                tls:
                 - secretName: tooling.vault.steghub.com
                   hosts:
                     - tooling.vault.steghub.com
            
              terminationGracePeriodSeconds: 10
            
              # extraSecretEnvironmentVars is a list of extra environment variables to set with the stateful set.
              # These variables take value from existing Secret objects.
              extraSecretEnvironmentVars:
                - envName: VAULT_SEAL_TYPE
                  secretName: vault-kms
                  secretKey: VAULT_SEAL_TYPE
                - envName: VAULT_AWSKMS_SEAL_KEY_ID
                  secretName: vault-kms
                  secretKey: VAULT_AWSKMS_SEAL_KEY_ID
            
              # Enables a headless service to be used by the Vault Statefulset
              service:
                enabled: true
            
                # Port on which Vault server is listening
                port: 8200
                # Target port to which the service should be mapped to
                targetPort: 8200
                # Extra annotations for the service definition.
                annotations: {}
            
              # This configures the Vault Statefulset to create a PVC for data
              # storage when using the file or raft backend storage engines.
              dataStorage:
                enabled: true
                # Size of the PVC created
                size: 2Gi
                # Location where the PVC will be mounted.
                mountPath: "/vault/data"
                # Name of the storage class to use.  If null it will use the
                # configured default Storage Class.
                storageClass: null
                # Access Mode of the storage device being used for the PVC
                accessMode: ReadWriteOnce
                annotations: {}
            
              # Run Vault in "HA" mode. There are no storage requirements unless the audit log
              # persistence is required.  In HA mode Vault will configure itself to use Consul
              # for its storage backend.
              ha:
                enabled: true
                replicas: 3
            
                # If set to null, this will be set to the Pod IP Address
                apiAddr: null
                clusterAddr: null
            
                # Enables Vault's integrated Raft storage.
                raft:
                  # Enables Raft integrated storage
                  enabled: true
                  # Set the Node Raft ID to the name of the pod
                  setNodeId: true
            
                  config: |
                    ui = true
            
                    listener "tcp" {
                      tls_disable = 1
                      address = "[::]:8200"
                      cluster_address = "[::]:8201"
                    }
            
                    storage "raft" {
                      path = "/vault/data"
            
                      retry_join {
                        leader_api_addr = "http://vault-0.vault-internal:8200"
                      }
            
                      retry_join {
                        leader_api_addr = "http://vault-1.vault-internal:8200"
                      } 
            
                      retry_join {
                        leader_api_addr = "http://vault-2.vault-internal:8200"
                      }
            
                      autopilot {
                        cleanup_dead_servers = "true"
                        last_contact_threshold = "200ms"
                        last_contact_failure_threshold = "10m"
                        max_trailing_logs = 250000
                        min_quorum = 5
                        server_stabilization_time = "10s"
                      }
                    }
            
                    # cluster_addr = "http://vault:8200"
            
                    service_registration "kubernetes" {}
            
            
                # config is a raw string of default configuration when using a Stateful
                # deployment. Default is to use a Consul for its HA storage backend.
                # This should be HCL.
                config: |
                  ui = true
            
                  listener "tcp" {
                    tls_disable = 1
                    address = "[::]:8200"
                    cluster_address = "[::]:8201"
                  }
            
                  seal "awskms" {
                  }
            
                  service_registration "kubernetes" {}
            
                  log_requests_level = "trace"
            
              # Definition of the serviceAccount used to run Vault.
              # These options are also used when using an external Vault server to validate
              # Kubernetes tokens.
              serviceAccount:
                # Specifies whether a service account should be created
                create: true
                # The name of the service account to use.
                name: "vault-kms"
                # Extra annotations for the serviceAccount definition. This can either be
                # YAML or a YAML-formatted multi-line templated string map of the
                # annotations to apply to the serviceAccount.
                annotations: 
                  eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/vaultKMS ## Update role for new AWS account
            
            
            # Vault UI
            ui:
              # True if you want to create a Service entry for the Vault UI.
              #
              # serviceType can be used to control the type of service created. For
              # example, setting this to "LoadBalancer" will create an external load
              # balancer to access the UI.
              enabled: true
              publishNotReadyAddresses: true
              # The service should only contain selectors for active Vault pod
              activeVaultPodOnly: false
              serviceType: "ClusterIP"
              serviceNodePort: null
              externalPort: 8200
              targetPort: 8200

**Install Vault**: Update the ServiceAccount annotations with _jsonpath="{server.serviceAccount.annotations}_" with your account ID. Change your directory to the vault directory and run the comand below to install the vault in your cluster. Replace the URL _tooling.vault.steghub.com_ with your domain name and update the record on your hosted zone.

From the values file above, we are using **ingress annotation** under the **server** field to configure the ingress. The ingress is configured with a TLS certificate, and the certificate is managed by **Cert-manager**, as you can see below.

            ingress:
              annotations: 
                cert-manager.io/cluster-issuer: letsencrypt-prod
                cert-manager.io/private-key-rotation-policy: Always


Apply the changes to your cluster

            kubectl kustomize --enable-helm overlays/dev | kubectl apply -f -





































































































































































