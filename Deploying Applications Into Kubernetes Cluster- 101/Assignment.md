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
























































































































































































































































































































































































































