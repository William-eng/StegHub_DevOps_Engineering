# Enterprise Logging for Kubernetes Applications using ELK 101
In this project, you'll architect and implement a production-grade logging infrastructure using the Elastic Stack (formerly ELK Stack) for applications running on Kubernetes.

## Key Goals
- Design and implement a scalable logging architecture for Kubernetes
- Make informed decisions about resource allocation
- Implement security best practices for logging infrastructure
- Create meaningful visualizations of your application using Kibana dashboards
- Handle high-volume log ingestion efficiently

- ![Image1](https://github.com/user-attachments/assets/d1f79540-e141-459f-9df7-e771eaeb8e86)

## ELK Infrastructure Setup Resource Planning
Deploy the application you want to log, the tooling-app :

- ![Image2](https://github.com/user-attachments/assets/ec8a4381-08cc-47b0-8fde-e706b2f55a7e)

- ![Image3](https://github.com/user-attachments/assets/609bf7e2-02a0-4dca-a8c9-ac42d52316b6)

- ![Image4](https://github.com/user-attachments/assets/f2f44c16-f066-4022-be43-898ef1f1df9c)



using the below script, calculate the log per component on each pod:

      #!/bin/bash
      for pod in $(kubectl get pods  -o jsonpath='{.items[*].metadata.name}'); do
          echo "Logs for $pod:"
          kubectl logs $pod | wc -c | awk '{print $1/1024/1024 " MB"}'
      done


run the script 

- ![Image5](https://github.com/user-attachments/assets/cd653b53-56df-47b9-92a3-18207a996649)

### Base Log Size per Component:
Each component is generating ~0.0002-0.0007 MB of logs as shown in the above image
Average log size appears to be around 0.0005 MB per component

To see the resource consumption of pods. Due to the metrics pipeline delay, they may be unavailable for a few minutes since pod creation. 

      kubectl top pod [NAME | -l label]

- ![Image6](https://github.com/user-attachments/assets/e496d5ba-e7f9-4d7e-bce1-bdd51e982a22)

Run the command below to create the metrics 

    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

- ![Image7](https://github.com/user-attachments/assets/08607294-dc41-4a56-bcba-d3b820e128e8)

the run the kubectl top command again:

- ![Image8](https://github.com/user-attachments/assets/d3bd72c8-7aab-42c6-86a6-c0863f6fb49d)

Based on these inferences we can deduce:

1. **Daily raw log volume**

         Daily Log Volume = (Number of Components × Average Log Size × Log Events per Day)

Daily raw log volume ≈ (6 components × 0.0005 MB × 1440 minutes)
≈ 4.32 MB per day of raw logs


2. **Total Storage Needed**

         Total Storage Needed = Daily Log Volume × Retention Days × Peak Factor × Growth Buffer

- Peak logging periods (usually 2-3× normal volume)
- Log retention period (e.g., 30 days)
- Growth factor (20% buffer)

**Total Storage Needed**  = 4.32 MB × 30 × 2 × 1.2
≈ 311 MB


**Recommendations**:

we will plan for at least 500 MB of storage to account for:

- Unexpected spikes in logging
- Additional metadata and indexes
- Future component scaling

## Elastic Stack Deployment

The ELK stack is composed of Elastic Search, Kibana, and Logstash, and its primary role is log aggregation. As microservices architecture becomes increasingly popular, the need for an efficient method to collect and search logs for debugging has grown. The ELK stack addresses this need by gathering logs and enabling their exploration. The key components of the ELK stack include:

- Elastic Search: A database designed to store and index logs.
- Kibana: A visualization tool that allows users to create queries and analyze data stored in Elastic Search.
- Logstash: A data processing pipeline that collects logs from various sources, processes them, and sends them to Elastic Search.
- Filebeat: A lightweight log forwarding agent that exports logs from their source and delivers them to Logstash.

The directory for the default yaml file for the Elastic-search is [here](https://github.com/elastic/helm-charts)

Now create a directory similar to 

            ├── elasticsearch.yaml
            ├── kibana.yaml
            └── filebeat.yaml
            └── logstash.yaml


**Deploy ElasticSearch:**

Now, we will create a custom values file for Kibana helm chart. Create a file values-2.yaml with the following content:
edit the default config file 

Fine-tune the resource settings to ensure the DaemonSet operates efficiently

            resources:
              requests:
                cpu: "500m"
                memory: "1Gi"
              limits:
                cpu: "1"
                memory: "512Mi"
            antiAffinity: "soft"

_antiAffinity: "soft"_: Configures soft anti-affinity, allowing pods to be scheduled on the same node if necessary, but preferring to spread them across nodes when possible.



Now execute the following commands to add the Elastic Search helm repo:

            helm repo add elastic https://helm.elastic.co
            helm repo update
- ![Image9](https://github.com/user-attachments/assets/1751d84c-e04d-44d6-a693-166ac27acb80)


Now to deploy the elastic search, execute the command:

            helm install elk-elasticsearch elastic/elasticsearch -f elasticsearch.yaml --namespace logging --create-namespace

- ![Image10](https://github.com/user-attachments/assets/65ba6acc-467b-43c4-a4a3-619ef4524dcc)

**Deploy Kibana**:

Now, we will create a custom values file for Kibana helm chart. In the kibana.yaml, edit the default yaml with the following content:

            service:
              type: NodePort
              port: 5601
            
            resources:
              requests:
                cpu: "200m"
                memory: "200Mi"
              limits:
                cpu: "1000m"
                memory: "2Gi"

- service.type: NodePort: Exposes Kibana on a specific port on all nodes in the Kubernetes cluster. This makes it accessible from outside the cluster for development and testing purposes.
- port: 5601: The default port for Kibana, which is exposed for accessing the Kibana web interface.


Now, to deploy the helm chart use the command:

      helm install elk-kibana elastic/kibana -f kibana.yaml

- ![Image10](https://github.com/user-attachments/assets/7ba0bf8b-f19e-4d55-81ba-ae1f435a8b8b)

**Deploy the logstash:**

Logstash processes and transforms logs before indexing them in Elasticsearch. We’ll set up Logstash to receive logs from Filebeat and send them to Elasticsearch.
In the C logstash.yaml file with the following content


                  extraEnvs:
                    - name: "ELASTICSEARCH_USERNAME"
                      valueFrom:
                        secretKeyRef:
                          name: elasticsearch-master-credentials
                          key: username
                    - name: "ELASTICSEARCH_PASSWORD"
                      valueFrom:
                        secretKeyRef:
                          name: elasticsearch-master-credentials
                          key: password
                  
                  logstashConfig:
                    logstash.yml: |
                      http.host: 0.0.0.0
                      xpack.monitoring.enabled: false
                  
                  logstashPipeline:
                    logstash.conf: |
                      input {
                        beats {
                          port => 5044
                        }
                      }
                  
                      output {
                        elasticsearch {
                          hosts => ["https://elasticsearch-master:9200"]
                          cacert => "/usr/share/logstash/config/elasticsearch-master-certs/ca.crt"
                          user => '${ELASTICSEARCH_USERNAME}'
                          password => '${ELASTICSEARCH_PASSWORD}'
                        }
                      }
                  
                  secretMounts:
                    - name: "elasticsearch-master-certs"
                      secretName: "elasticsearch-master-certs"
                      path: "/usr/share/logstash/config/elasticsearch-master-certs"
                  
                  service:
                    type: ClusterIP
                    ports:
                      - name: beats
                        port: 5044
                        protocol: TCP
                        targetPort: 5044
                      - name: http
                        port: 8080
                        protocol: TCP
                        targetPort: 8080
                  
                  resources:
                    requests:
                      cpu: "200m"
                      memory: "200Mi"
                    limits:
                      cpu: "1000m"
                      memory: "1536Mi" 


- extraEnvs: Sets environment variables for Elasticsearch authentication using Kubernetes secrets.
- logstashConfig: Configures Logstash settings, including enabling HTTP and disabling monitoring.
- logstashPipeline: Configures Logstash to listen on port 5044 for incoming logs from Filebeat and forward them to Elasticsearch.
- secretMounts: Mounts the Elasticsearch CA certificate for secure communication between Logstash and Elasticsearch.
- service: Configures Logstash’s service type as ClusterIP, making it accessible only within the cluster.


Now to deploy the logstash, execute the following command:

            helm install elk-logstash elastic/logstash -f logstash.yaml

- ![Image11](https://github.com/user-attachments/assets/97101699-e4eb-40f7-8f69-fcb2e915fe16)


**Deploy the filebeat**:

Filebeat is a lightweight shipper for forwarding and centralizing log data. We’ll configure Filebeat to collect logs from the logging application and forward them to Logstash.
In the _filebeat-values.yaml_ file we will add with the following content:


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
                
- filebeat.inputs: Configures Filebeat to collect logs from container directories. The path /var/log/containers/*.log is where Kubernetes stores container logs.
- processors: Adds Kubernetes metadata to the logs to provide context, such as pod names and namespaces.
- output.logstash: Configures Filebeat to send logs to Logstash at port 5044.

Now, to deploy the filebeat use the following command:

            helm install elk-filebeat elastic/filebeat -f filebeat.yaml

- ![Image12](https://github.com/user-attachments/assets/c1f7148e-be35-4faf-8b19-d1fd3e6ecc1c)

  After successful deployment, run the command below:

              kubectl get all -n logging



- ![Image13](https://github.com/user-attachments/assets/d84756ca-2638-4e62-8750-179679d44c52)

- ![Image14](https://github.com/user-attachments/assets/3988acd6-2b50-4c18-bcb1-545c87f87cb6)


Now that Kibana is installed and running, you can access it to visualize and analyze the logs collected by Filebeat and processed by Logstash.

Find the NodePort assigned to Kibana:

      kubectl get svc elk-kibana-kibana -n elk -o jsonpath="{.spec.ports[0].nodePort}"

- ![Image15](https://github.com/user-attachments/assets/4176d582-050d-44f1-8ead-988cc1587b5d)

  Open your web browser and navigate to:

            http://<EXTERNAL-IP>:<NODE-PORT>

  Replace _<EXTERNAL-IP>_ with the IP address of your Kubernetes cluster and _<NODE-PORT>_ with the NodePort value 

- ![Imaage16](https://github.com/user-attachments/assets/ef4c68a9-1a43-4866-b33a-dbbdb71a09f9)

Once you access Kibana, you can start exploring your log data.

- ![Image17](https://github.com/user-attachments/assets/a592edfb-68b2-4fa2-b59f-fef3e6cba61b)

Access the logs
- ![Image18](https://github.com/user-attachments/assets/c29433f7-e1b6-4888-a717-464f40780c29)

  - ![Image19](https://github.com/user-attachments/assets/c8a1950f-0a0a-470b-85c5-0513a22ab16c)



  



















































































































