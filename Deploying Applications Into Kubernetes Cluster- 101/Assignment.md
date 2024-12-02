# QUICK TASK 
## Now We will setup the following tools using Helm
This section will be quite challenging for you because you will need to spend some time to research the charts, read their documentations and understand how to get an application running in your cluster by simply running a helm install command.

1. Artifactory
2. Hashicorp Vault
3. Prometheus
4. Grafana
5. Elasticsearch ELK using ECK

## 1. Artifactory
The following steps were taking to install JFrog Artifactory in the EKS (Elastic Kubernetes Service) cluster using Helm:

- ![Image01](https://github.com/user-attachments/assets/c23faf93-0fe3-455e-806a-f266b8925cd1)


1. Add JFrog Helm repository
Before installing JFrog helm charts, you need to add them to your helm client

        helm repo add jfrog https://charts.jfrog.io
        helm repo update
   
 - ![Image02](https://github.com/user-attachments/assets/536ab7dd-0903-4af2-b59e-810949481f9d)


2. Create a Namespace for Artifactory. It’s a good practice to create a dedicated namespace for Artifactory

        kubectl create namespace artifactory


3. Install Artifactory Using Helm
   
                helm install artifactory jfrog/artifactory \
                  --namespace artifactory \
                  --set artifactory.service.type=LoadBalancer \
                  --set postgresql.enabled=true \
                  --set artifactory.admin.password=admin
   
- artifactory: Name of the release.
- jfrog/artifactory: Helm chart to install.
- --namespace artifactory: Namespace where Artifactory will be installed.
- --set artifactory.service.type=LoadBalancer: Exposes Artifactory using a LoadBalancer service.
- --set postgresql.enabled=true: Enables PostgreSQL as the database for Artifactory.
- --set artifactory.admin.password=: Set the admin password.
  
- ![Image03](https://github.com/user-attachments/assets/0f0e3f8c-d652-4ab3-8d5b-1e95624719b5)


4. Check the status of the Helm release
   
                helm status artifactory -n artifactory

- ![Image04](https://github.com/user-attachments/assets/1830721d-387c-449a-aad5-0838d7d56632)


Check status of nginx service

                kubectl get svc --namespace artifactory -w artifactory-artifactory-nginx

- ![Image05](https://github.com/user-attachments/assets/65533fde-b175-478e-ae81-23287b736529)

Check status of the service

                kubectl get svc -n artifactory

- ![Image06](https://github.com/user-attachments/assets/455ae078-26c3-498f-9802-ed2a4a51bd75)

5. Access the Artifactory via a browser by port forwarding

   
                kubectl port-forward svc/artifactory  8082:8082 -n artifactory

- ![Image](https://github.com/user-attachments/assets/baa7e59e-673f-487e-bdd5-70da0103c207)
![Image](https://github.com/user-attachments/assets/2ca8e855-84c4-4bdb-aa1c-4b041538f6f1)




## 2. Hashicorp Vault
- Install HashiCorp Vault in EKS cluster using Helm
Add HashiCorp Helm repository to get access to the Vault Helm chart

        helm repo add hashicorp https://helm.releases.hashicorp.com
        helm repo update
- ![Image07](https://github.com/user-attachments/assets/413cd18a-f27b-4893-8a11-069dd9b036fc)



- Install Vault Using Helm
  
        helm install vault hashicorp/vault --namespace vault --create-namespace
  
  - ![Image08](https://github.com/user-attachments/assets/a5ddecdc-0fad-4c20-9cae-655574bf1c2e)


- Check if Vault pods are running correctly in the EKS cluster.

                kubectl get pods -n vault


- ![Image09](https://github.com/user-attachments/assets/357cbcb6-1c7b-45f8-8758-65eaf4b26543)

- Initialize vault-0 with one key share and one key threshold.

                         kubectl exec vault-0 -n vault -- vault operator init \
                            -key-shares=1 \
                            -key-threshold=1 \
                            -format=json > cluster-keys.json
                        cat cluster-keys.json

  operator init command generates a root key that it disassembles into key shares -key-shares=1

-key-threshold=1then sets the number of key shares required to unseal Vault

- ![Image10](https://github.com/user-attachments/assets/f38ba247-911f-4211-8807-4c5deba3bd6a)


- Create a variable named VAULT_UNSEAL_KEY to capture the Vault unseal key

        VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" cluster-keys.json)


- Unseal Vault running on the vault-0 pod

                 kubectl exec vault-0 -n vault -- vault operator unseal   VAULT_UNSEAL_KEY

- ![Image11](https://github.com/user-attachments/assets/9a53bfe7-4b52-46fe-a626-863a103fd258)

- Join the vault-1 and vault-2pods to the Raft cluster
  
                $ kubectl exec -ti vault-1 -n vault -- vault operator raft join http://vault-0.vault-internal:8200
                $ kubectl exec -ti vault-2 -n vault -- vault operator raft join http://vault-0.vault-internal:8200


- 

- Use the unseal key from above to unseal vault-1 and vault-2
Unsealing is the process of constructing the root key necessary to read the decryption key to decrypt the data, allowing access to the Vault.

                        $ kubectl exec -ti vault-1 -n vault -- vault operator unseal VAULT_UNSEAL_KEY
                        $ kubectl exec -ti vault-2 -n vault -- vault operator unseal VAULT_UNSEAL_KEY

- After this unsealing process all vault pods are now in running (1/1 ready ) state
  
                        $ kubectl get po -n vault


- ![Image12](https://github.com/user-attachments/assets/e6b325b3-2f78-429e-8d55-84762e4a3910)



Vault service is of ClusterIP type which means we can access Vault console from browser so to access this we need to use port-forward command

        $ kubectl port-forward service/vault -n vault 8200:8200



- ![Image13](https://github.com/user-attachments/assets/cfa73427-5e7a-4bb0-a9bc-390bf5706146)

- type http://localhost:8200 in browser and enter root token in cluster.json

- ![Image14](https://github.com/user-attachments/assets/15f0466d-978c-4c5b-b24f-4734ff5060e8)
- ![Image15](https://github.com/user-attachments/assets/01b5fdf9-afbc-42b2-b1e9-86c288c8d656)
- ![Image16](https://github.com/user-attachments/assets/4c5910a0-900c-4a24-a1a9-aea1928789a9)



## 3. Prometheus
- Add the Prometheus Helm repo

        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        helm repo update

- ![Image17](https://github.com/user-attachments/assets/657cf9a5-341a-41e0-9f6e-0fd809647f3d)

- Create a namespace
  
                kubectl create namespace monitoring

- Install Prometheus
  
        helm install prometheus prometheus-community/prometheus --namespace monitoring

- ![Image18](https://github.com/user-attachments/assets/3cf65a00-3ed8-4395-a0ce-55af26e850db)

- Accessing Prometheus UI
Port-forward Prometheus to local machine to access the UI

        kubectl port-forward -n monitoring svc/prometheus-server 9090:80



- ![Image19](https://github.com/user-attachments/assets/2c945b50-0bdf-4f14-9493-aff457378397)
- ![Image20](https://github.com/user-attachments/assets/9ff6e01c-00a3-46f1-8c32-66b5eadc4b19)

- ![Image21](https://github.com/user-attachments/assets/c6f5ddf6-9408-410c-bcf9-8e06ee3f994d)



## 4. Grafana
- Add the official Grafana Helm repository to Helm client

                helm repo add grafana https://grafana.github.io/helm-charts
                helm repo update

- ![Image22](https://github.com/user-attachments/assets/76001a7d-91fa-4b6c-a704-1d77822bc882)

- Install Grafana using Helm
  
                helm install grafana grafana/grafana --namespace monitoring

- ![Image23](https://github.com/user-attachments/assets/363551fb-a947-4bbb-8823-20fb4c0dad97)


- Get the Grafana admin password

        kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo



### - Access Grafana UI

- Get the pod and service

                kubectl get svc -o wide -n monitoring


- ![Image24](https://github.com/user-attachments/assets/1733ac36-1638-4f70-90e9-7428869a1cfd)

- port-forward to the local machine

                kubectl port-forward --namespace monitoring svc/grafana 3000:80


- ![Image25](https://github.com/user-attachments/assets/c6cdfa1c-855e-462d-8aa3-fceb64270da0)
- ![Image26](https://github.com/user-attachments/assets/fa3b7de6-f35e-4cd3-8951-c2b8a4ffee48)
- ![Image27](https://github.com/user-attachments/assets/44987610-7161-48c7-9851-97056af3f64b)


## 5. Elasticsearch ELK using [ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-install-helm.html)

### Step 1. Install ECK Operator
ECK (Elastic Cloud on Kubernetes) is the official way to deploy and manage Elasticsearch on Kubernetes. This will handle the deployment of Elasticsearch, Kibana, and other components.

Add the Elastic Helm repository

                helm repo add elastic https://helm.elastic.co
                helm repo update



- ![Image28](https://github.com/user-attachments/assets/42664a9c-662d-4cf2-ad58-679d086fbdd5)


- Now, use the curl command to download the values.yaml file containing configuration information:

                curl -O https://raw.githubusercontent.com/elastic/helm-charts/master/elasticsearch/examples/minikube/values.yaml

- Use the helm install command and the values.yaml file to install the Elasticsearch helm chart:

                helm install elasticsearch elastic/elasticsearch -f ./values.yaml

The -f option allows specifying the yaml file with the template. If you wish to install Elasticsearch in a specific namespace, add the -n option followed by the name of the namespace.

                helm install elasticsearch elastic/elasticsearch -n [namespace] -f ./values.yaml

The output confirms the status of the app as deployed and offers additional options to test the installation:

-  The first option is to use the get pods command to check if the cluster members are up:

                kubectl get pods --namespace=default -l app=elasticsearch-master -w

Once the READY column in the output is entirely populated with 1/1 entries, all the cluster members are up:

- 4. The first option is to use the get pods command to check if the cluster members are up:

                kubectl get pods --namespace=default -l app=elasticsearch-master -w

Once the READY column in the output is entirely populated with 1/1 entries, all the cluster members are up:


                kubectl port-forward svc/elasticsearch-master 9200


- ![Image](https://github.com/user-attachments/assets/761e2464-2406-4b8b-87f8-5be43ae48964)
                



### Install Kibana
1. To install Kibana on top of Elasticsearch, type the following command:

                helm install kibana elastic/kibana

The output confirms the deployment of Kibana:

2. Check if all the pods are ready:

                kubectl get pods

3. Forward Kibana to port 5601 using kubectl:


                kubectl port-forward deployment/kibana-kibana 5601

- ![Image](https://github.com/user-attachments/assets/ba4e964a-6d24-4458-beca-989091d0f45f)
   

5. After you set up port-forwarding, access Elasticsearch, and the Kibana GUI by typing http://localhost:5601 in your browser:

- ![Image](https://github.com/user-attachments/assets/14d70301-6060-4aed-9840-58f06c5b748f)














































































































































































































































































