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


- ![Image18](https://github.com/user-attachments/assets/532ed1ef-20df-4aee-88ce-50308e47d557)



We use the command above instead of _kubectl apply -k overlays/dev_ due to a current limitation in Kustomize when working with Helm charts. The direct -k flag doesn't automatically enable Helm chart processing, which is required for our deployment. While this is a known limitation in Kustomize, this alternative command serves our purpose effectively by explicitly enabling Helm support using the _--enable-helm_ flag and piping the output to kubectl apply.

- Follow the next steps to initialize the Vault cluster.

- Run the command below to view all pods in vault namespace

              kubectl get po -n vault

  - ![Image19](https://github.com/user-attachments/assets/6ed1225e-a84d-44ad-a7d2-018f23dd6bdc)
 
pod/vault-0 is the pod in running status in this image

- Check the status of the vault cluster vault status, you should get an output similar to this:

            ---                      -----
            Recovery Seal Type       awskms
            Initialized              false
            Sealed                   true
            Total Recovery Shares    0
            Threshold                0
            Unseal Progress          0/0
            Unseal Nonce             n/a
            Version                  1.11.3
            Storage Type             raft
            HA Enabled               true


- ![Image20](https://github.com/user-attachments/assets/9a1b3de7-100d-4011-b056-ef9d76fe9f70)

- To initialize the Vault cluster you will run:

               vault operator init
You should get something similar to this after initializing the Vault cluster:

              Recovery Key 1: 4QY2/CbUeORpS......2Ek4jWk5HJF0sk/rb
              Recovery Key 2: EZrZw6BTsbD7m....../uqmRtPudLAWDWGfT
              Recovery Key 3: 4+wkqRbaJXosL......i4xENfUPlfm3lpr8t
              Recovery Key 4: GpbUXhbyt9TUm......Dy/rXhkOWC+2CcrQT
              Recovery Key 5: RzCAwLhMlWz1v......KV1I5inAU25S+gzhL
            
              Initial Root Token: hvs.JNGtNK.....Z8H7KqerwUNWL
            
              Success! Vault is initialized
            
              Recovery key initialized with 5 key shares and a key threshold of 3. Please
              securely distribute the key shares printed above.


| Copy the output into a file and save it.

- Check the status after when the vault cluster is unsealed vault status:
- ![Image22](https://github.com/user-attachments/assets/a0912105-8ba4-4715-824b-bb9663f86dcf)

From the vault status output, before you initialize the Vault cluster, you will see that the seal type is awskms, but after initializing the vault cluster you will get the recovery keys because some of the Vault operations still require Shamir keys. The Recovery keys generated after running vault operator init can be used to unseal the cluster when it is sealed manually or to regenerate a root token.

- Lastly, exit the vault-* pod.



**NB**: After initializing your vault cluster, some vault pods might not start running. This is because the current configuration uses pod affinity to spread pods across multiple nodes, but your cluster might have fewer nodes than required. You may need to either add more nodes or modify the pod affinity settings in the values.yaml file if you want all the pods to be running for High Availability.

The **awskms** key type is used for **auto unseal**, using the **awskms** key type, you don't have to manually unseal the pod if it gets recreated. Move to the next page to see how you can inject secrets from the Vault cluster into an application.

## Dynamically inject secrets into the tooling app container
In this session, we will see how we can securely inject the tooling application database credentials from the vault cluster into the tooling application. This method can be used to pass secret credentials like passwords, tokens, and other secret credentials into an application without the application being aware of the vault cluster.

To store the secrets we need to create a Vault secret of type **KV Version 2**, this is a versioned Key-Value store. You can exit out of the vault pod and install Vault on your local machine from [here](https://developer.hashicorp.com/vault/downloads) if you don't have Vault installed on your system. Export the vault address and login.

            export VAULT_ADDR="https://tooling.vault.steghub.com"
            
            vault login


- ![Image24](https://github.com/user-attachments/assets/4b5e7eed-2fd2-4e4d-8037-1c10c2cae7aa)

- Enable the kv-v2 secrets at the path app.

              vault secrets enable -path=app kv-v2

- ![Image25](https://github.com/user-attachments/assets/ee680101-2690-426c-a6d7-69d12e24126d)


- Create the tooling application database credentials at the path app/database/config/dev.

              vault kv put app/database/config/dev username=db password=password host=http://<aurora-db-endpoint>

- Verify that the secret is defined at the path app/database/config/dev.

              vault kv get app/database/config/dev

## Configure Kubernetes Authentication
Enable the Kubernetes auth method at the default path.

            vault auth enable kubernetes

- Configure Vault to talk to Kubernetes with the /config path. This will require the Kubernetes host address, use _kubectl cluster-info_ to get the Kubernetes host address and TCP port and replace it with the kubernetes_host in the command below.

              vault write auth/kubernetes/config \
              kubernetes_host=https://1758918D244B7C9FF1D1.gr7.eu-central-1.eks.amazonaws.com

You will get something like this:

            Success! Data written to: auth/kubernetes/config


- ![Image25](https://github.com/user-attachments/assets/b95cc65c-d753-4d52-a021-03b65a60275d)

- For the tooling application to read the database credentials, it needs the read capability to the path _app/data/database/config_. We can do this by creating a policy and attaching it to the Kubernetes authentication role we will create.


              vault policy write tooling-db - <<EOF
            path "app/data/database/config/*" {
              capabilities = ["read"]
            }
            EOF

- ![Image26](https://github.com/user-attachments/assets/733ee529-cceb-44e1-b308-845bb4d78c25)


- Create a Kubernetes authentication role named tooling-role

            vault write auth/kubernetes/role/tooling-role \
              ttl=6h \
              policies=tooling-db \
              bound_service_account_names=tooling-sa \
              bound_service_account_namespaces=dev-tooling
From the command above we are passing the service account name and namespace to _bound_service_account_names_ and _bound_service_account_namespaces_ respectively. You should get an output like this:

            Success! Data written to: auth/kubernetes/role/tooling-role


- ![Image27](https://github.com/user-attachments/assets/01123c8c-a191-483e-8caa-25af92578504)

## Inject Secrets into the Tooling Application
To inject secrets into the tooling application you will create a service account with the same name configured in the Kubernetes role and attach it to the tooling application pod. This will create a sidecar which is the Vault agent and it will do the authentication and inject the secrets into the application.

In the _tooling-app-kustomize/overlays_ directory where you have your Kubernetes manifest files, create a file _service-account.yaml_ and add:

### _tooling-app-kustomize/overlays/dev/service-account.yaml_

            apiVersion: v1
            kind: ServiceAccount
            metadata:
              name: tooling-sa


- ![Image26](https://github.com/user-attachments/assets/e5d55dd8-e1e5-4c8d-8692-07d34cb2c7c9)

### tooling-app-kustomize/overlays/dev/deployment.yaml
replace the content with:

            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: tooling-deployment
            spec:
              replicas: 3
              template:
                metadata:
                  annotations:
                    vault.hashicorp.com/agent-inject: 'true'
                    vault.hashicorp.com/role: 'tooling-role'
                    vault.hashicorp.com/agent-inject-status: 'update'
                    vault.hashicorp.com/agent-inject-secret-database-cred.txt: 'app/data/database/config/dev'
                    vault.hashicorp.com/agent-inject-template-database-cred.txt: |
                      {{- with secret "app/data/database/config/dev" -}}
                      export db-username={{ .Data.data.username }}
                      export db-password={{ .Data.data.password }}
                      export db-host={{ .Data.data.password }}
                      {{- end -}}
                spec:
                  serviceAccountName: tooling-sa

- ![Image27](https://github.com/user-attachments/assets/e1800ce8-1e90-4cc7-97d0-388b8be8fbc1)

- Add the _service-account.yaml_ file under the **resources** field of your Kustomization file in the dev directory. The Kustomization file should look like this:

            apiVersion: kustomize.config.k8s.io/v1beta1
            kind: Kustomization
            namespace: dev-tooling
            resources:
              - ../../base
              - namespace.yaml
              - service-account.yaml
            
            labels:
              - pairs:
                  env: dev-tooling
            
            patches:
              - path: deployment.yaml

- Now lets apply the configuration.

              kubectl apply -k overlays/dev

  - ![Image28](https://github.com/user-attachments/assets/d11f1d43-a5d9-42de-a4bc-28fa85459e87)


You can inspect the tooling application pod, you will find out that the new pod now launches two containers. The application container, named tooling, and the Vault Agent container, named vault-agent.

- You can inspect the tooling application pod, you will find out that the new pod now launches two containers. The application container, named tooling, and the Vault Agent container, named vault-agent.

  
Vault Agent manages the token lifecycle and the secret retrieval. The database credentials will be saved at the path _/vault/secrets/database-cred.txt_ in the tooling application container. Run the command below to check the content of the file.


            kubectl exec -it deployment/tooling-deployment -n dev-tooling 
              -c tooling -- cat /vault/secrets/database-cred.txt

- To export the Database credentials in the tooling application you can run the command below:

              kubectl exec -it deployment/tooling-deployment -n dev-tooling -c tooling -- cat /vault/secrets/database-cred.txt


- ![Image28](https://github.com/user-attachments/assets/ae982945-6ccd-404c-965e-bf8a671fee8a)

You can work on passing the credentials file as an environment variable so that your tooling application can automatically ingest the credentials without having to manually run the command above. You can also change the credential path to the dotenv file in the application tooling workspace and change the file format.


## Working with the Vault UI
We have been using the **Vault CLI** for our vault configurations but you can also use the **vault UI** to do some of the configurations. To view the vault UI, copy and paste the vault address on your browser, and then you will see the login page.


- ![Image29](https://github.com/user-attachments/assets/0e3799bb-a7f5-4a19-8457-5731c9fe997b)

- You can log in using the token you got after initializing the vault cluster. Check the database KV secret created before which is at the path app/database/config/dev.

- ![Image30](https://github.com/user-attachments/assets/4c9f7f14-9366-41d6-809c-332cbce07cd3)

- ![Image31](https://github.com/user-attachments/assets/8098f552-faaf-460b-9567-26ec4984afec)

Check the tooling-role Kubernetes auth method role.

- ![Image32](https://github.com/user-attachments/assets/0996d0ba-01f5-4f5c-ac7d-4381277296be)
- ![Image33](https://github.com/user-attachments/assets/3966f8db-67d7-4295-a65e-1660520ff5b9)
- ![Image34](https://github.com/user-attachments/assets/8f829f31-33e9-4142-9d75-0eb73517a082)



- Navigate to the vault policy. attached to the tooling-role Kubernetes auth method.

- ![Image35](https://github.com/user-attachments/assets/615da71a-018a-44e1-8527-5762e213de62)



Congratulations! You have completed this project. By completing this project, you've gained valuable experience in implementing a robust and secure Kubernetes deployment strategy using a powerful combination of Helm, Kustomize, and Vault.

Here's a recap of the key skills you've developed:
1. Efficient Environment Management: You learned how to leverage Kustomize to create a base configuration for your application and then tailor it for different environments (dev, sit, prod) using overlays. This promotes code reusability and reduces the risk of configuration drift.

2. Streamlined Complex Deployments: You used Helm to simplify the installation and management of Vault, a complex application with its own set of configurations. This demonstrates the power of Helm for handling pre-packaged applications.

3. Secrets Management Best Practices: You successfully integrated Vault into your Kubernetes workflow to securely store and dynamically inject database credentials into your application. This eliminates the need for hardcoding secrets and significantly enhances the security posture of your deployments.

4. Kubernetes Authentication and Authorization: You configured Vault to authenticate with your Kubernetes cluster and defined fine-grained access control policies to restrict which applications and services can access sensitive secrets.

























































































