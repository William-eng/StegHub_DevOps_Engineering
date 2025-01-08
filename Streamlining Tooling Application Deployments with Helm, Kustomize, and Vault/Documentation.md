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









































































































































































































































































































































