![image](https://github.com/user-attachments/assets/f713bc2f-bebf-4c2b-8eec-6ed6f9c60628)# Persisting Data in Kubernetes- 101
NOTE: Create EKS cluster first before the below section

### 1. Set up an Amazon Elastic Kubernetes Service (EKS) cluster.
Ensure AWS CLI is already installed and access key credentials have been configured.


- Install eksctl on Linux
  
          # for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
          ARCH=amd64
          PLATFORM=$(uname -s)_$ARCH
          curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
          
          # (Optional) Verify checksum
          curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
          
          tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
          
          sudo mv /tmp/eksctl /usr/local/bin


- Create EKS cluster using eksctl or from AWS console

      eksctl create cluster --name fnc-eks-cluster --region us-east-1 --nodegroup-name ktrontech-node-group --node-type t2.micro --nodes 1

- Enable OIDC to allow the cluster to use IAM roles for service accounts and automatically configure IAM permissions for addons.
  
        eksctl utils associate-iam-oidc-provider --cluster fnc-eks-cluster --approve

- ![Image01](https://github.com/user-attachments/assets/2a0eed9f-2d9f-40d0-b1de-2afe2f73a185)

- ![Image02](https://github.com/user-attachments/assets/a068712a-8647-4900-abbc-d00c92fad2b9)


- Check the cluster status and verify API connection

- ![image03](https://github.com/user-attachments/assets/2d2c8b99-990e-42cf-b7cb-2b96cd648333)
- ![Image04](https://github.com/user-attachments/assets/ab59b292-5e57-444e-9dee-5918416f7b62)

Now we know that containers are stateless by design, which means that data does not persist in the containers. Even when you run the containers in kubernetes pods, they still remain stateless unless you ensure that your configuration supports statefulness.

To achieve statefuleness in kubernetes, we must understand how _volumes_, _persistent volumes_, and _persistent volume claims_ work.

### Volumes
On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers. One problem is the loss of files when a container crashes. The kubelet restarts the container but with a clean state. A second problem occurs when sharing files between containers running together in a Pod. The Kubernetes volume abstraction solves both of these problems

Docker has a concept of volumes, though it is somewhat looser and less managed. A Docker volume is a directory on disk or in another container. Docker provides volume drivers, but the functionality is somewhat limited.

Kubernetes supports many types of volumes. A Pod can use any number of volume types simultaneously. **Ephemeral volume** types have a lifetime of a pod, but **persistent volumes** exist beyond the lifetime of a pod. When a pod ceases to exist, Kubernetes destroys ephemeral volumes; however, Kubernetes does not destroy persistent volumes. For any kind of volume in a given pod, data is preserved across container restarts.

At its core, a volume is a directory, possibly with some data in it, which is accessible to the containers in a pod. How that directory comes to be, the medium that backs it, and the contents of it are all determined by the particular **volume type** used. This means, you must know some of the different types of volumes available in kubernetes before choosing what is ideal for your particular use case.

Lets have a look at a few of them.


### awsElasticBlockStore
An awsElasticBlockStore volume mounts an Amazon Web Services (AWS) EBS volume into your pod. The contents of an EBS volume are persisted and the volume is only unmmounted when the pod crashes, or terminates. This means that an EBS volume can be pre-populated with data, and that data can be shared between pods.

Lets see what it looks like for our Nginx pod to persist data using awsElasticBlockStore volume

            sudo cat <<EOF | sudo tee ./nginx-pod.yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: nginx-deployment
              labels:
                tier: frontend
            spec:
              replicas: 1
              selector:
                matchLabels:
                  tier: frontend
              template:
                metadata:
                  labels:
                    tier: frontend
                spec:
                  containers:
                  - name: nginx
                    image: nginx:latest
                    ports:
                    - containerPort: 80
                  volumes:
                  - name: nginx-volume
                    # This AWS EBS volume must already exist.
                    awsElasticBlockStore:
                      volumeID: "vol-0467b24acb34419d5"
                      fsType: ext4
            EOF

The **volume** section indicates the type of volume to be used to ensure persistence.

If you notice the config above carefully, you will realise that there is need to provide a **volumeID** before the deployment will work. Therefore, You must create an **EBS** volume by using aws ec2 create-volume command or the AWS console.

Before you create a volume, lets run the nginx deployment into kubernetes without a volume.

        sudo cat <<EOF | sudo tee ./nginx-pod.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: nginx-deployment
          labels:
            tier: frontend
        spec:
          replicas: 3
          selector:
            matchLabels:
              tier: frontend
          template:
            metadata:
              labels:
                tier: frontend
            spec:
              containers:
              - name: nginx
                image: nginx:latest
                ports:
                - containerPort: 80
        EOF

      kubectl apply -f nginx-pod.yaml

- ![Image05](https://github.com/user-attachments/assets/06844f1d-fc29-4bbd-81f5-82c082f53cd0)



- **Tasks**

- Verify that the pod is running

      kubectl get pods


- ![Image06](https://github.com/user-attachments/assets/7377ef6c-64e8-4e58-8542-ee1b11da840d)

- Check the logs of the pod

        kubectl logs -l tier=frontend

- ![Image07](https://github.com/user-attachments/assets/b67ab313-9ee1-4529-85dc-fb5178d19f07)


- Exec into the pod and navigate to the nginx configuration file /etc/nginx/conf.d

          POD_NAME=$(kubectl get pod -l tier=frontend -o jsonpath="{.items[0].metadata.name}")

          kubectl exec -it $POD_NAME -- /bin/bash

          cd /etc/nginx/conf.d


- Open the config files to see the default configuration.

- ![Image08](https://github.com/user-attachments/assets/a602725b-ab23-4d37-8e87-a244ac2d259c)

**NOTE**: There are some restrictions when using an awsElasticBlockStore volume:

- The nodes on which pods are running must be AWS EC2 instances
- Those instances need to be in the same region and availability zone as the EBS volume
- EBS only supports a single EC2 instance mounting a volume
  
Now that we have the pod running without a volume, Lets now create a volume from the AWS console.

- In your AWS console, head over to the EC2 section and scroll down to the Elastic Block Storage menu.
- Click on Volumes
- At the top right, click on Create Volume
Part of the requirements is to ensure that the volume exists in the same region and availability zone as the EC2 instance running the pod. Hence, we need to find out

- Which node is running the pod (replace the pod name with yours)


              kubectl get po nginx-deployment-75b7745567-9gxvz -o wide

**Output**:

      NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE                                       NOMINATED NODE   READINESS GATES
      nginx-deployment-6fdcffd8fc-thcfp   1/1     Running   0          64m   10.0.3.159   ip-10-0-3-233.eu-west-2.compute.internal   <none>           <none>


- ![Image09](https://github.com/user-attachments/assets/e926f856-8f8c-4be6-8817-59f6020f1345)

The NODE column shows the node the pode is running on

- In which Availability Zone the node is running.

        kubectl describe node ip-10-0-3-233.eu-west-2.compute.internal 

The information is written in the labels section of the descibe command. 

- ![Image10](https://github.com/user-attachments/assets/1ffa6bc0-f9b5-4b9c-bedb-bd1ee1ef0e85)

- So, in the case above, we know the AZ for the node is in _us-east-1f_ hence, the volume must be created in the same AZ. Choose the size of the required volume.
  
The **create volume** selection should be like:

- ![Image11](https://github.com/user-attachments/assets/20c1c876-4425-49ff-8da8-436e0eb43a5b)

- Copy the VolumeID

- Update the deployment configuration with the volume spec.


              sudo cat <<EOF | sudo tee ./nginx-pod.yaml
              apiVersion: apps/v1
              kind: Deployment
              metadata:
                name: nginx-deployment
                labels:
                  tier: frontend
              spec:
                replicas: 1
                selector:
                  matchLabels:
                    tier: frontend
                template:
                  metadata:
                    labels:
                      tier: frontend
                  spec:
                    containers:
                    - name: nginx
                      image: nginx:latest
                      ports:
                      - containerPort: 80
                    volumes:
                    - name: nginx-volume
                      # This AWS EBS volume must already exist.
                      awsElasticBlockStore:
                        volumeID: "vol-01d59773bbb64f8c5"
                        fsType: ext4
              EOF

Apply the new configuration and check the pod. As you can see, the old pod is being terminated while the updated one is up and running.

        kubectl apply -f nginx-pod.yaml

- ![Image12](https://github.com/user-attachments/assets/85a59315-5852-4cb3-8c79-74e196fd39e8)

Now, the new pod has a volume attached to it, and can be used to run a container for statefuleness. Go ahead and explore the running pod. Run describe on both the pod and deployment

        kubectl describe pod nginx-deployment-6fbdb6d65f-jslb9

- ![Image13](https://github.com/user-attachments/assets/7c5ae295-84d0-4ddc-aa39-9047ca93d36f)
![Image14](https://github.com/user-attachments/assets/8a60d873-611c-45ea-9546-24d6df2be01b)

At this point, even though the pod can be used for a stateful application, the configuration is not yet complete. This is because, the volume is not yet mounted onto any specific filesystem inside the container. The directory _/usr/share/nginx/html_ which holds the software/website code is still ephemeral, and if there is any kind of update to the _index.html_ file, the new changes will only be there for as long as the pod is still running. If the pod dies after, all previously written data will be erased.

To complete the configuration, we will need to add another section to the deployment yaml manifest. The **volumeMounts** which basically answers the question "Where should this Volume be mounted inside the container?" Mounting a volume to a directory means that all data written to the directory will be stored on that volume.

Lets do that now.



              cat <<EOF | tee ./nginx-pod.yaml
              apiVersion: apps/v1
              kind: Deployment
              metadata:
                name: nginx-deployment
                labels:
                  tier: frontend
              spec:
                replicas: 1
                selector:
                  matchLabels:
                    tier: frontend
                template:
                  metadata:
                    labels:
                      tier: frontend
                  spec:
                    containers:
                    - name: nginx
                      image: nginx:latest
                      ports:
                      - containerPort: 80
                      volumeMounts:
                      - name: nginx-volume
                        mountPath: /usr/share/nginx/
                    volumes:
                    - name: nginx-volume
                      # This AWS EBS volume must already exist.
                      awsElasticBlockStore:
                        volumeID: "vol-01d59773bbb64f8c5"
                        fsType: ext4
              EOF
              
Notice the newly added section:

        volumeMounts:
        - name: nginx-volume
          mountPath: /usr/share/nginx/


- The value provided to _name_ in _volumeMounts_ must be the same value used in the volumes section. It basically means mount the volume with the name provided, to the provided mountpath
In as much as we now have a way to persist data, we also have new problems.

If you port forward the service and try to reach the endpoint, you will get a 403 error. This is because mounting a volume on a filesystem that already contains data will automatically erase all the existing data. This strategy for statefulness is preferred if the mounted volume already contains the data which you want to be made available to the container.

- ![Image15](https://github.com/user-attachments/assets/ab08e53b-bcf2-4dda-be59-ebe75c90509a)

- It is still a manual process to create a volume, manually ensure that the volume created is in the same Avaioability zone in which the pod is running, and then update the manifest file to use the volume ID. All of these is against DevOps principles because it will mean having a lot of road blocks to getting a simple thing done.
The more elegant way to achieve this is through Persistent Volume and Persistent Volume claims.

In kubernetes, there are many elegant ways of persisting data. Each of which is used to satisfy different use cases. Lets take a look at the different options available.

- **Persistent Volume (PV)** and **Persistent Volume Claim (PVC)**
- **configMap**

### Managing Volumes Dynamically with PVs and PVCs
Kubernetes provides API objects for storage management such that, the lower level details of volume provisioning, storage allocation, access management etc are all abstracted away from the user, and all you have to do is present manifest files that describes what you want to get done.

PVs are volume plugins that have a lifecycle completely independent of any individual Pod that uses the PV. This means that even when a pod dies, the PV remains. A PV is a piece of storage in the cluster that is either provisioned by an administrator through a manifest file, or it can be dynamically created if a storage class has been pre-configured.

Creating a PV manually is like what we have done previously where with creating the volume from the console. As much as possible, we should allow PVs to be created automatically just be adding it to the container spec iin deployments. But without a storageclass present in the cluster, PVs cannot be automatically created.

If your infrastructure relies on a storage system such as NFS, iSCSI or a cloud provider-specific storage system such as EBS on AWS, then you can dynamically create a PV which will create a volume that a Pod can then use. This means that there must be a storageClass resource in the cluster before a PV can be provisioned.

By default, in EKS, there is a default storageClass configured as part of EKS installation. This storageclass is based on gp2 which is Amazon's default type of volume for Elastic block storage.gp2 is backled by solid-state drives (SSDs) which means they are suitable for a broad range of transactional workloads.

Run the command below to check if you already have a storageclass in your cluster kubectl get storageclass




















































































































































































































