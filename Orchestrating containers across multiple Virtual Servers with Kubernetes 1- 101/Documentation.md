## Orchestrating containers across multiple Virtual Servers with Kubernetes 1- 101
This project is the first of a series Kubernetes-related practice projects, so get ready to become a professional containers' fleet "pilot"*.

*Kubernetes (κυβερνήτης, Greek for "helmsman" or "pilot" or "governor", and the etymological root of cybernetics)

- ![image-114-1024x681](https://github.com/user-attachments/assets/0d00ce77-9bd7-4d05-a43e-aed43a428257)

It is important to understand that, DevOps is about "Culture" and NOT "Tools" Therefore, it is not correct to say that one tool is better than another; different organizations have different needs and a good tool for 
one team may be bad for another just because their needs are not the same. In some teams, Docker Compose fit their needs perfectly, despite the perceived limitations. The major limitation of Docker Compose is that it 
can only be used to run workloads on a single computer host. Now, that is an obvious limitation because if our **Tooling Application** and its **MySQL Database** are all running on a single VM, like we did in Project 20, then
this host is considered as a **SPOF (i.e. - Single Point Of Failure)**.

So, could we say there is something wrong with Docker Compose? Not exactly, as a matter of fact, it is being used a lot in the industry. It fits well into some use cases that require speedy development and Proof of
Concepts. As you will soon see, Kubernetes is a lot more complex technology, and it may be an overkill for some use cases.


**Container orchestration** is a concept that allows to address these two scenarios, it provides automation of all the aspects of coordinating and managing containers. Container orchestration is focused on managing 
life cycle of containers and their dynamic environments.

It is about automating the entire lifecycle of containers running across multiple nodes:

- Configuring and scheduling of containers on nodes
- Ensuring the availability of containers, even when they die
- Scaling of containers to equally balance application workloads across infrastructure
- Allocation of resources between containers
- Load balancing, traffic routing and service discovery of containers
- Health monitoring of containers
- Securing the interactions between containers.
Kubernetes is a tool designed to do Container Orchestration and it does its job very well when correctly configured.

As mentioned earlier, there are other alternatives to Docker Compose. But, throughout the entire PBL program, we will not focus on Docker Swarm. We will rather spend more time with Kubernetes. 
Part of the reason for this is because Kubernetes has more functionalities and is widely in use in the industry.

## Kubernetes architecture
Kubernetes is a not a single package application that you can install with one command, it is comprised of several components, some of them can be deployed as services, some can be also deployed as separate containers.

Let us take a look at Kubernetes architecture diagram below:

- ![image-115-1024x600](https://github.com/user-attachments/assets/863ef095-03ed-474a-b238-fb1771396884)

      Docker Swarm is preferred in environments where simplicity and fast development is preferred, whereas Kubernetes is fit for the environments where medium to large clusters are running complex applications.

- [official documentation.](https://kubernetes.io/docs/concepts/overview/components/)

## "Kubernetes From-Ground-Up"
### K8s installation options
To successfully implement "K8s From-Ground-Up", the following and even more will be done by you as a K8s administrator
1. Install and configure master (also known as control plane) components and worker nodes (or just nodes).
2. Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
3. In transit encryption means encrypting communications over the network using HTTPS
4. At rest encryption means encrypting the data stored on a disk
5. Plan the capacity for the backend data store etcd
6. Configure network plugins for the containers to communicate
7. Manage periodical upgrade of the cluster
8. Configure observability and auditing

## Tools to be used and expected result of the Project 20
- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster
  
  We will create **6 EC2 Instances**, and in the end, we will have the following parts of the cluster properly configured:
- Three Kubernetes Master
- Three Kubernetes Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

## Step 0 Install client tools before bootstrapping the cluster.
First, you will need some client tools installed and configurations made on your client workstation:

- [awscli](https://aws.amazon.com/cli/) - is a unified tool to manage your AWS services
- [kubectl](https://kubernetes.io/docs/reference/kubectl/) - this command line utility will be your main control tool to manage your K8s cluster. You will use this tool so many times, so you will be able to type 'kubetcl' on your keyboard with a speed of light. You can always make a shortcut (alias) to just one character 'k'. Also, add this extremely useful official kubectl Cheat Sheet to your bookmarks, it has examples of the most used 'kubectl' commands.
- [cfssl](https://blog.cloudflare.com/introducing-cfssl/) - an open source toolkit for everything TLS/SSL from [Cloudflare](https://www.cloudflare.com/en-gb/)
- [cfssljson](https://github.com/cloudflare/cfssl) - a program, which takes the JSON output from the cfssl and writes certificates, keys, [CSRs](https://en.wikipedia.org/wiki/Certificate_signing_request), and bundles to disk.























































































































































































































































































































































































































































