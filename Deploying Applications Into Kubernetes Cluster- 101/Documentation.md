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

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run stateful applications can be configured in private datacentres, while worker nodes 
that require heavy computations and stateless applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.





























































































































































































































































































