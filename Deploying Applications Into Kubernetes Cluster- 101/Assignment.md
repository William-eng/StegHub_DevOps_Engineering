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


- Install the ECK operator:
  
                helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace


- ![Image29](https://github.com/user-attachments/assets/cd3dff01-8872-40e2-acc4-d8d0db15c95b)



- Open your terminal and run the above command, this will add all the repositories that we need for our Elastic stack, to confirm repos were added just search the repo we added

        helm search repo elastic

- ![Image30](https://github.com/user-attachments/assets/aeb7a4ac-3c49-4951-8036-f7fcd6c601f3)
- ![ImageX](https://github.com/user-attachments/assets/ac10e7ac-d4bf-47ab-8873-4ffc26b63e84)


We will overwrite the default helm value file to change some configs

**Filebeat**
Create a directory for Filebeat and run this command

                helm show values elastic/filebeat > values.yml 

This creates a file called values.yml
By default, Filebeat connects to Elasticsearch but we want to change this to point to Logstash.

In your value.yml file change the following config;

                filebeatConfig:
                    filebeat.yml: |
                      filebeat.inputs:
                      - type: container
                        paths:
                          - /var/log/containers/*.log
                        processors:
                        - add_kubernetes_metadata:
                            host: ${NODE_NAME}
                            matchers:
                            - logs_path:
                                logs_path: "/var/log/containers/"
                
                      output.elasticsearch:
                        host: '${NODE_NAME}'
                        hosts: '["https://${ELASTICSEARCH_HOSTS:elasticsearch-master:9200}"]'
                        username: '${ELASTICSEARCH_USERNAME}'
                        password: '${ELASTICSEARCH_PASSWORD}'
                        protocol: https
                        ssl.certificate_authorities: ["/usr/share/filebeat/certs/ca.crt"]



To

                filebeatConfig:
                    filebeat.yml: |
                      filebeat.inputs:
                      - type: container
                        paths:
                          - /var/log/containers/*.log
                        processors:
                        - add_kubernetes_metadata:
                            host: ${NODE_NAME}
                            matchers:
                            - logs_path:
                                logs_path: "/var/log/containers/"
                
                      output.logstash:
                        hosts: ["logstash-logstash:5044"]




- ![Image31](https://github.com/user-attachments/assets/46c8f52c-cc47-4ed0-ac3d-898e7692fadc)


**Logstash**
We will do the same Logstash to get its value file;

                helm show values elastic/logstash > values.yml 


We will be connecting Logstash with Elasticsearch and these are the default configs that we will need to add



                logstashPipeline:
                  logstash.conf: |
                    input {
                      beats {
                        port => 5044
                      }
                    }
                
                     output {
                      elasticsearch {
                        hosts => "https://elasticsearch-master:9200"
                        cacert => "/usr/share/logstash/config/elasticsearch-master-certs/ca.crt"
                        user => '${ELASTICSEARCH_USERNAME}'  # Elasticsearch username
                        password => '${ELASTICSEARCH_PASSWORD}' # Elasticsearch password
                      }
                    }



What we have done above is tell Logstash to expect its input from Filebeat on port 5044 and output it on Elasticsearch.

As we discussed earlier the latest version of Elastic search has by default enabled secure communication and we need to mount the certificate in our pod to be used for secure communication


                secretMounts:
                  - name: "elasticsearch-master-certs"
                    secretName: "elasticsearch-master-certs"
                    path: "/usr/share/logstash/config/elasticsearch-master-certs"




Elasticsearch will create secrets and one of the secrets will have ca.crt that we want in our Logstash pod that will be used to decrypt and encrypt communication between the two.

By default service is not enabled, enable it by adding;

                service:
                  annotations: {}
                  type: ClusterIP
                  loadBalancerIP: ""
                  ports:
                    - name: beats
                      port: 5044
                      protocol: TCP
                      targetPort: 5044
                    - name: http
                      port: 8080
                      protocol: TCP
                      targetPort: 8080

This should be it for Logstash there is more but this is enough.


- ![Image32](https://github.com/user-attachments/assets/8f4e7de2-5fca-4c28-b84e-d714704711ba)
- ![Image33](https://github.com/user-attachments/assets/81827d55-e28d-4f3c-8c40-8e8b060604ad)
- ![Image34](https://github.com/user-attachments/assets/b09305e0-38ee-4d74-a8ec-8fa6712ceff9)



**Elasticsearch**
We will get its value file

                helm show values elastic/elasticsearch > values.yml 
The current value works well with our setup, but it's good to have the value file in case you want to change values.


Kibana

        helm show values elastic/kibana > values.yml
        
We want to expose Kibana because that is how we will be able to log in to our dashboard.



                type: LoadBalancer
                  loadBalancerIP: ""
                  port: 5601
                  nodePort: ""
                  labels: {}
                  annotations: {}



**Installation**.
Now that we have all set it time we file the installation and I prefer running it, in this order;

- Elasticsearch

- Filebeat

- Logstash

- Kibana

helm install elasticsearch elastic/elasticsearch -f values.yml -n monitoring
helm install filebeat elastic/filebeat -f values.yml -n monitoring
helm install logstash elastic/logstash -f values.yml -n monitoring
helm install kibana elastic/kibana -f values.yml -n monitoring  

- ![Image35](https://github.com/user-attachments/assets/3574ddf9-9494-44e6-8ddc-f387be944750)

- ![Image36](https://github.com/user-attachments/assets/a8f9ab4b-0228-4ef3-92cb-33441d7cf30b)

- 

Please note to run the commands where the corresponding value file was saved.
If all goes well the pods should be up and running.

Kibana will generate a public URL for our case.




































































































































































































































































































