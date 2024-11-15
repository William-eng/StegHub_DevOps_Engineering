# Deploying Applications Into Kubernetes Cluster- 101

In this project, we will build upon your knowledge of Kubernetes architecture, and begin to deploy applications on a K8s cluster. Kubernetes has a lot of moving parts; it operates with several layers of abstraction between your 
application and host machines where it runs. So many terms, and capabilities that is not realistic to learn it all at once. Hence, you will be introduced to as many concepts as possible, but gradually.

## Choosing the right Kubernetes cluster set up
When it comes to using a Kubernetes cluster, there is a number of options available depending on the ultimate use of it. For example, if you just need a cluster for development or learning, you can use lightweight tools like 
[Minikube](https://minikube.sigs.kubernetes.io/docs/start/), or [k3s](https://k3s.io/).

or [Kind](https://kind.sigs.k8s.io/docs/user/quick-start)

        #Download the Kind binary
        
        curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
        
        #Make the Kind binary executable:
        
        chmod +x ./kind
        
        #Move the binary to your system's PATH:
        
        sudo mv ./kind /usr/local/bin/kind
        
        #After installing Kind, verify that it's properly installed by checking the version:
        
        kind --version
        
        #Once Kind is installed, you can create your cluster with:
        
        sudo kind create cluster
        
        #Verify that the cluster is created:
        
        sudo kubectl get nodes
- ![Kind](https://github.com/user-attachments/assets/3ef5d599-b1fe-4c1d-9040-de84cc7ef20b)


Other options will be to leverage a [Managed Service](https://www.adept.co.uk/the-benefits-of-cloud-managed-services-for-business/) Kubernetes cluster from public cloud providers such as: [AWS EKS](https://aws.amazon.com/eks),
[Microsoft AKS](https://azure.microsoft.com/en-gb/services/kubernetes-service), or [Google Cloud Platform GKE](https://cloud.google.com/kubernetes-engine). There are so many more options out there. Regardless of whichever 
one you choose, the experience is usually very similar.

Using [eksctl](https://learn.arm.com/install-guides/eksctl/) to setup a cluster on EKS

- Download the eksctl package using curl:
  
        curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_arm64.tar.gz"

- Install eksctl with:
  
       tar -xzf eksctl_Linux_arm64.tar.gz -C /tmp && rm eksctl_Linux_arm64.tar.gz

- sudo mv /tmp/eksctl /usr/local/bin

- Confirm eksctl is installed:

      eksctl version

- The output will be similar to:

       0.160

- With your AWS account configured, run eksctl to create a cluster with 2 nodes with AWS Graviton processors:

        eksctl create cluster  \
        --name cluster-1  \
        --region us-east-1 \
        --node-type t4g.small \
        --nodes 2 \
        --nodegroup-name node-group-1


- ![Creating-Cluster](https://github.com/user-attachments/assets/1e8cfced-83fc-4ef8-b63b-38d8ed2b1c74)

- ![cluster-created](https://github.com/user-attachments/assets/efa7e5ae-df19-4f0e-a017-3fefb9f4fb79)

- ![noderunning](https://github.com/user-attachments/assets/7732476d-13b8-4e25-854a-f80a0203eddf)


- When the cluster is created, use kubectl to get the status of the nodes in the cluster.

          kubectl get nodes -o wide

- ![runningowide](https://github.com/user-attachments/assets/8d7efb1f-eb81-40ac-b904-1e3705290eaa)

Most organisations choose Managed Service options for obvious reasons such as:

1. Less administrative overheads
2. Reduced cost of ownership
3. Improved Security
4. Seamless support
5. Periodical updates to a stable and well-tested version
6. Faster cluster spin up
... and many more

However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. Most of the companies that go this route will mostly use on-premises data 
centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which 
their data is being persisted. Banks and Governments are typical examples of this.

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run [stateful](https://whatis.techtarget.com/definition/stateful-app) applications can be configured in private datacentres, while worker nodes 
that require heavy computations and [stateless](https://www.redhat.com/en/topics/cloud-native-apps/stateful-vs-stateless) applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.

- ![Hybrid-Managed-Architecture](https://github.com/user-attachments/assets/cbbb6525-4113-4165-9f6c-94d6718cb78f)


## Deploying the Tooling app using Kubernetes objects
In this section, you will begin to write configuration files for Kubernetes objects (they are usually referred as _manifests_) in the form of files with _yaml_ syntax and deploy them using _kubectl_ console. But first, let us understand what a Kubernetes object is.

**Kubernetes objects** are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

- What containerized applications are running (and on which nodes)
- The resources available to those applications
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance
These objects are "**_record of intent_**" - once you create the object, the Kubernetes system will constantly work to ensure that the object exists. By creating an object, you are effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.

To work with Kubernetes objects - whether to create, modify, or delete them - you will need to use the Kubernetes API. When you use the _kubectl_ command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. It is also possible to use _curl_ to directly interact with the Kubernetes API, or it can be as part of developing a program in different programming languages. That will require some advance knowledge. You can read more about client libraries to get an idea on how that works.


## Common Kubernetes objects
- Pod
- Namespace
- ResplicaSet (Manages Pods)
- DeploymentController (Manages Pods)
- StatefulSet
- DaemonSet
- Service
- ConfigMap
- Volume
- Job/Cronjob
The very first concept to understand is the difference between how **Docker** and **Kubernetes** run containers - with Docker, every _docker run_ command will run an image (representing an application) as a container. The running container is a Docker's smallest entity, it is the most basic deployable object. Kubernetes on the other hand operates with _pods_ instead of containers, a _pods_ encapsulates a container. Kubernetes uses pods as its smallest, and most basic deployable object with a unique feature that allows it to run multiple containers within a single Pod. It is not the most common pattern - to have more than one container in a Pod, but there are cases when this capability comes in handy.

In the world of docker, or docker compose, to run the Tooling app, you must deploy separate containers for the application and the database. But in the world of Kubernetes, you can run both: application and database containers in the same Pod. When multiple containers run within the same Pod, they can directly communicate with each other as if they were running on the same localhost. Although running both the application and database in the same Pod is NOT a recommended approach.

A **Pod** that contains one container is called _single container_ pod and it is the most common Kubernetes use case. A Pod that contains multiple co-related containers is called _multi-container pod_. There are few patterns for multi-container Pods; one of them is the [**sidecar**](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d) container pattern - it means that in the same Pod there is a main container and an auxiliary one that extends and enhances the functionality of the main one without changing it.

There are other patterns, such as: init container, adapter container, ambassador container. These are more advanced topics that you can study on your own, let us continue with the other objects.

We will not go into the theoretical details of all the objects, rather we will begin to experience them in action.


## Understanding the common YAML fields for every Kubernetes object
Every Kubernetes object includes object fields that govern the object's configuration:

- kind: Represents the type of kubernetes object created. It can be a Pod, DaemonSet, Deployments or Service.
- version: Kubernetes api version used to create the resource, it can be v1, v1beta and v2. Some of the kubernetes features can be released under beta and available for general public usage.
- metadata: provides information about the resource like name of the Pod, namespace under which the Pod will be running, labels and annotations.
- spec: consists of the core information about Pod. Here we will tell kubernetes what would be the expected state of resource, Like container image, number of replicas, environment variables and volumes.
- status: consists of information about the running object, status of each container. Status field is supplied and updated by Kubernetes after creation. This is not something you will have to put in the YAML manifest.


## Deploying a random Pod
Lets see what it looks like to have a Pod running in a k8s cluster. This section is just to illustrate and get you to familiarise with how the object's fields work. Lets deploy a basic Nginx container to run inside a Pod.

- apiVersion is v1
- kind is Pod
- metatdata has a name which is set to nginx-pod
- The spec section has further information about the Pod. Where to find the image to run the container - (This defaults to Docker Hub), the port and protocol.

The structure is similar for any Kubernetes objects, and you will get to see them all as we progress.

1. Create [a Pod](https://kubernetes.io/docs/concepts/workloads/pods/) _yaml_ manifest on your master node

                sudo cat <<EOF | sudo tee ./nginx-pod.yaml
                apiVersion: v1
                kind: Pod
                metadata:
                  name: nginx-pod
                spec:
                  containers:
                  - image: nginx:latest
                    name: nginx-pod
                    ports:
                    - containerPort: 80
                      protocol: TCP
                EOF

- ![Image1](https://github.com/user-attachments/assets/772b53f3-626e-4316-9278-8f0dd11b5d0d)

2. Apply the manifest with the help of kubectl

                kubectl apply -f nginx-pod.yaml

**Output**:

                pod/nginx-pod created

3. Get an output of the pods running in the cluster

                kubectl get pods
**Output**:

                NAME        READY   STATUS    RESTARTS   AGE
                nginx-pod   1/1     Running   0          19m

If the Pods were not ready for any reason, for example if there are no worker nodes, you will see something like the below output.

                NAME        READY   STATUS    RESTARTS   AGE
                nginx-pod   0/1     Pending   0          111s

5. To see other fields introduced by kubernetes after you have deployed the resource, simply run below command, and examine the output. You will see other fields that kubernetes updates from time to time to represent the state of the resource within the cluster. -o simply means the output format.

                kubectl get pod nginx-pod -o yaml 
or

                kubectl describe pod nginx-pod

- ![Image2](https://github.com/user-attachments/assets/412affae-8e44-42af-ad9c-696ab1843e08)

- ![Image3](https://github.com/user-attachments/assets/db3c758d-da43-46f0-95ae-f5c3e99f9005)
- ![Image4](https://github.com/user-attachments/assets/2d669a02-ca23-4162-83ab-5c0330f896da)


























































































































































































































































