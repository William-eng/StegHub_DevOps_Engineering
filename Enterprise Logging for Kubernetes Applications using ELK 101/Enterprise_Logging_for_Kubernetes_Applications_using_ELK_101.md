![image](https://github.com/user-attachments/assets/a895538d-0bc4-42a6-861e-1e962b6ef3b2)# Enterprise Logging for Kubernetes Applications using ELK 101
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


## USING FLUENTBIT
Another way to go about this is using fluentbit

## Installation and Configuration
## 1. Install the ECK Operator & Fluent Bit agent(s)

Install the elastic eck operator with the command 

      kubectl create ns observer
      
      helm repo add elastic https://helm.elastic.co
      helm repo add fluent https://fluent.github.io/helm-charts
      
      helm install elastic-operator elastic/eck-operator -n observer --kubeconfig /path/to/your/kubeconfig  # Replace with your kubeconfig path

- ![Image20](https://github.com/user-attachments/assets/f0aada72-9faa-4d62-aff9-d64e61addf81)

- ![Image21](https://github.com/user-attachments/assets/9919bd90-5d59-4f5b-a67c-7a1e93a269c3)


check the operator with the command 

      kubectl get po -n observer

- ![Image22](https://github.com/user-attachments/assets/f82fee12-2f91-490a-8da7-17d43f487dff)


Now create the elastic search  with yaml file 

            apiVersion: elasticsearch.k8s.elastic.co/v1
            kind: Elasticsearch
            metadata:
              name: quickstart-es
              namespace: obs-system
            spec:
              version: 8.17.1 # Use your desired version
              nodeSets:
              - name: default
                count: 1
                config:
                  node.store.allow_mmap: false # Important for some environments
              http:
                service:
                  spec:
                    type: LoadBalancer # Or NodePort if LoadBalancer not available

##  2. Deploy Elasticsearch Cluster

Use the command 

      kubectl apply -f elastic.yml

- ![Image23](https://github.com/user-attachments/assets/9f496bfd-7a1f-42e5-8746-9d0c92037f2b)


## 3. Deploy Kibana Instance

Integrate the kibana to elastic-search with the yaml file :

            apiVersion: kibana.k8s.elastic.co/v1
            kind: Kibana
            metadata:
              name: quickstart-kb
              namespace: obs-system
            spec:
              version: 8.17.1 # Use your desired version
              count: 1
              elasticsearchRef:
                name: quickstart-es
              http:
                service:
                  spec:
                    type: NodePort # Or LoadBalancer if available and preferred


use the command :

            kubectl apply -f kibana.yml
- ![Image24](https://github.com/user-attachments/assets/fa727c8d-2918-4617-b84e-d22a3619d082)



now we can see the running configurations with the command

            kubectl get all -n <namespace>

we can see the services, deployments and statefulsets created by our little configuration file

let's get the user name and password for the elastic searcch to populate our fluent bit agent

            kubectl get secrets -n observer | grep elastic 
            kubectl get secrets -n observer quickstart-es-es-elastic-user -o yaml

decode the base64 secret with the command:

            echo "<elastic-secret>"  | base64 -d

now let's check the elastic search service IP-adress so we can implement it in our fluentbit yaml file:

      kubectl get svc -n observer | grep es


edit the flentbit yaml file with this in this configuration:

            kind: DaemonSet
            
            # replicaCount -- Only applicable if kind=Deployment
            replicaCount: 1
            
            image:
              repository: cr.fluentbit.io/fluent/fluent-bit
              # Overrides the image tag whose default is {{ .Chart.AppVersion }}
              # Set to "-" to not use the default value
              tag:
              digest:
              pullPolicy: IfNotPresent
            
            testFramework:
              enabled: true
              namespace:
              image:
                repository: busybox
                pullPolicy: Always
                tag: latest
                digest:
            
            imagePullSecrets: []
            nameOverride: ""
            fullnameOverride: ""
            
            serviceAccount:
              create: true
              annotations: {}
              name:
            
            rbac:
              create: true
              nodeAccess: false
              eventsAccess: false
            
            # Configure podsecuritypolicy
            # Ref: https://kubernetes.io/docs/concepts/policy/pod-security-policy/
            # from Kubernetes 1.25, PSP is deprecated
            # See: https://kubernetes.io/blog/2022/08/23/kubernetes-v1-25-release/#pod-security-changes
            # We automatically disable PSP if Kubernetes version is 1.25 or higher
            podSecurityPolicy:
              create: false
              annotations: {}
            
            # OpenShift-specific configuration
            openShift:
              enabled: false
              securityContextConstraints:
                # Create SCC for Fluent-bit and allow use it
                create: true
                name: ""
                annotations: {}
                # Use existing SCC in cluster, rather then create new one
                existingName: ""
            
            podSecurityContext: {}
            #   fsGroup: 2000
            
            hostNetwork: false
            dnsPolicy: ClusterFirst
            
            dnsConfig: {}
            #   nameservers:
            #     - 1.2.3.4
            #   searches:
            #     - ns1.svc.cluster-domain.example
            #     - my.dns.search.suffix
            #   options:
            #     - name: ndots
            #       value: "2"
            #     - name: edns0
            
            hostAliases: []
            #   - ip: "1.2.3.4"
            #     hostnames:
            #     - "foo.local"
            #     - "bar.local"
            
            securityContext: {}
            #   capabilities:
            #     drop:
            #     - ALL
            #   readOnlyRootFilesystem: true
            #   runAsNonRoot: true
            #   runAsUser: 1000
            
            service:
              type: ClusterIP
              port: 2020
              internalTrafficPolicy:
              loadBalancerClass:
              loadBalancerSourceRanges: []
              labels: {}
              # nodePort: 30020
              # clusterIP: 172.16.10.1
              annotations: {}
            #   prometheus.io/path: "/api/v1/metrics/prometheus"
            #   prometheus.io/port: "2020"
            #   prometheus.io/scrape: "true"
              externalIPs: []
              # externalIPs:
              #  - 2.2.2.2
            
            
            serviceMonitor:
              enabled: false
              #   namespace: monitoring
              #   interval: 10s
              #   scrapeTimeout: 10s
              #   selector:
              #    prometheus: my-prometheus
              #  ## metric relabel configs to apply to samples before ingestion.
              #  ##
              #  metricRelabelings:
              #    - sourceLabels: [__meta_kubernetes_service_label_cluster]
              #      targetLabel: cluster
              #      regex: (.*)
              #      replacement: ${1}
              #      action: replace
              #  ## relabel configs to apply to samples after ingestion.
              #  ##
              #  relabelings:
              #    - sourceLabels: [__meta_kubernetes_pod_node_name]
              #      separator: ;
              #      regex: ^(.*)$
              #      targetLabel: nodename
              #      replacement: $1
              #      action: replace
              #  scheme: ""
              #  tlsConfig: {}
            
              ## Bear in mind if you want to collect metrics from a different port
              ## you will need to configure the new ports on the extraPorts property.
              additionalEndpoints: []
              # - port: metrics
              #   path: /metrics
              #   interval: 10s
              #   scrapeTimeout: 10s
              #   scheme: ""
              #   tlsConfig: {}
              #   # metric relabel configs to apply to samples before ingestion.
              #   #
              #   metricRelabelings:
              #     - sourceLabels: [__meta_kubernetes_service_label_cluster]
              #       targetLabel: cluster
              #       regex: (.*)
              #       replacement: ${1}
              #       action: replace
              #   # relabel configs to apply to samples after ingestion.
              #   #
              #   relabelings:
              #     - sourceLabels: [__meta_kubernetes_pod_node_name]
              #       separator: ;
              #       regex: ^(.*)$
              #       targetLabel: nodename
              #       replacement: $1
              #       action: replace
            
            prometheusRule:
              enabled: false
            #   namespace: ""
            #   additionalLabels: {}
            #   rules:
            #   - alert: NoOutputBytesProcessed
            #     expr: rate(fluentbit_output_proc_bytes_total[5m]) == 0
            #     annotations:
            #       message: |
            #         Fluent Bit instance {{ $labels.instance }}'s output plugin {{ $labels.name }} has not processed any
            #         bytes for at least 15 minutes.
            #       summary: No Output Bytes Processed
            #     for: 15m
            #     labels:
            #       severity: critical
            
            dashboards:
              enabled: false
              labelKey: grafana_dashboard
              labelValue: 1
              annotations: {}
              namespace: ""
            
            lifecycle: {}
            #   preStop:
            #     exec:
            #       command: ["/bin/sh", "-c", "sleep 20"]
            
            livenessProbe:
              httpGet:
                path: /
                port: http
            
            readinessProbe:
              httpGet:
                path: /api/v1/health
                port: http
            
            resources: {}
            #   limits:
            #     cpu: 100m
            #     memory: 128Mi
            #   requests:
            #     cpu: 100m
            #     memory: 128Mi
            
            ## only available if kind is Deployment
            ingress:
              enabled: false
              ingressClassName: ""
              annotations: {}
              #  kubernetes.io/ingress.class: nginx
              #  kubernetes.io/tls-acme: "true"
              hosts: []
              # - host: fluent-bit.example.tld
              extraHosts: []
              # - host: fluent-bit-extra.example.tld
              ## specify extraPort number
              #   port: 5170
              tls: []
              #  - secretName: fluent-bit-example-tld
              #    hosts:
              #      - fluent-bit.example.tld
            
            ## only available if kind is Deployment
            autoscaling:
              vpa:
                enabled: false
            
                annotations: {}
            
                # List of resources that the vertical pod autoscaler can control. Defaults to cpu and memory
                controlledResources: []
            
                # Define the max allowed resources for the pod
                maxAllowed: {}
                # cpu: 200m
                # memory: 100Mi
                # Define the min allowed resources for the pod
                minAllowed: {}
                # cpu: 200m
                # memory: 100Mi
            
                updatePolicy:
                  # Specifies whether recommended updates are applied when a Pod is started and whether recommended updates
                  # are applied during the life of a Pod. Possible values are "Off", "Initial", "Recreate", and "Auto".
                  updateMode: Auto
            
              enabled: false
              minReplicas: 1
              maxReplicas: 3
              targetCPUUtilizationPercentage: 75
              #  targetMemoryUtilizationPercentage: 75
              ## see https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics
              customRules: []
              #     - type: Pods
              #       pods:
              #         metric:
              #           name: packets-per-second
              #         target:
              #           type: AverageValue
              #           averageValue: 1k
              ## see https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-configurable-scaling-behavior
              behavior: {}
            #      scaleDown:
            #        policies:
            #          - type: Pods
            #            value: 4
            #            periodSeconds: 60
            #          - type: Percent
            #            value: 10
            #            periodSeconds: 60
            
            ## only available if kind is Deployment
            podDisruptionBudget:
              enabled: false
              annotations: {}
              maxUnavailable: "30%"
            
            nodeSelector: {}
            
            tolerations: []
            
            affinity: {}
            
            labels: {}
            
            annotations: {}
            
            podAnnotations: {}
            
            podLabels: {}
            
            ## How long (in seconds) a pods needs to be stable before progressing the deployment
            ##
            minReadySeconds:
            
            ## How long (in seconds) a pod may take to exit (useful with lifecycle hooks to ensure lb deregistration is done)
            ##
            terminationGracePeriodSeconds:
            
            priorityClassName: ""
            
            env: []
            #  - name: FOO
            #    value: "bar"
            
            # The envWithTpl array below has the same usage as "env", but is using the tpl function to support templatable string.
            # This can be useful when you want to pass dynamic values to the Chart using the helm argument "--set <variable>=<value>"
            # https://helm.sh/docs/howto/charts_tips_and_tricks/#using-the-tpl-function
            envWithTpl: []
            #  - name: FOO_2
            #    value: "{{ .Values.foo2 }}"
            #
            # foo2: bar2
            
            envFrom: []
            
            # This supports either a structured array or a templatable string
            extraContainers: []
            
            # Array mode
            # extraContainers:
            #   - name: do-something
            #     image: busybox
            #     command: ['do', 'something']
            
            # String mode
            # extraContainers: |-
            #   - name: do-something
            #     image: bitnami/kubectl:{{ .Capabilities.KubeVersion.Major }}.{{ .Capabilities.KubeVersion.Minor }}
            #     command: ['kubectl', 'version']
            
            flush: 1
            
            metricsPort: 2020
            
            extraPorts: []
            #   - port: 5170
            #     containerPort: 5170
            #     protocol: TCP
            #     name: tcp
            #     nodePort: 30517
            
            extraVolumes: []
            
            extraVolumeMounts: []
            
            updateStrategy: {}
            #   type: RollingUpdate
            #   rollingUpdate:
            #     maxUnavailable: 1
            
            # Make use of a pre-defined configmap instead of the one templated here
            existingConfigMap: ""
            
            networkPolicy:
              enabled: false
            #   ingress:
            #     from: []
            
            luaScripts:
              setIndex.lua: |
                function set_index(tag, timestamp, record)
                    index = "abhishek-"
                    if record["kubernetes"] ~= nil then
                        if record["kubernetes"]["namespace_name"] == "logging" then
                            return -1, timestamp, record  -- Skip logs from the logging namespace
                        end
                        if record["kubernetes"]["namespace_name"] ~= nil then
                            if record["kubernetes"]["container_name"] ~= nil then
                                record["es_index"] = index
                                    .. record["kubernetes"]["namespace_name"]
                                    .. "-"
                                    .. record["kubernetes"]["container_name"]
                                return 1, timestamp, record
                            end
                            record["es_index"] = index
                                .. record["kubernetes"]["namespace_name"]
                            return 1, timestamp, record
                        end
                    end
                    return 1, timestamp, record
                end
            
            ## https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/classic-mode/configuration-file
            config:
              service: |
                [SERVICE]
                    Daemon Off
                    Flush {{ .Values.flush }}
                    Log_Level {{ .Values.logLevel }}
                    Parsers_File /fluent-bit/etc/parsers.conf
                    Parsers_File /fluent-bit/etc/conf/custom_parsers.conf
                    HTTP_Server On
                    HTTP_Listen 0.0.0.0
                    HTTP_Port {{ .Values.metricsPort }}
                    Health_Check On
            
              ## https://docs.fluentbit.io/manual/pipeline/inputs
              inputs: |
                [INPUT]
                    Name tail
                    Path /var/log/containers/*.log
                    multiline.parser docker, cri
                    Tag kube.*
                    Mem_Buf_Limit 5MB
                    Skip_Long_Lines On
            
                [INPUT]
                    Name systemd
                    Tag host.*
                    Systemd_Filter _SYSTEMD_UNIT=kubelet.service
                    Read_From_Tail On
            
              ## https://docs.fluentbit.io/manual/pipeline/filters
              filters: |
                [FILTER]
                    Name kubernetes
                    Match kube.*
                    Merge_Log On
                    Keep_Log Off
                    K8S-Logging.Parser On
                    K8S-Logging.Exclude On
            
                [FILTER]
                    Name lua
                    Match kube.*
                    script /fluent-bit/scripts/setIndex.lua
                    call set_index
            
              ## https://docs.fluentbit.io/manual/pipeline/outputs
              outputs: |
                [OUTPUT]
                    Name es
                    Match kube.*
                    Type  _doc
                    Host  <elasticsearch_service_ip>
                    Port 9200
                    HTTP_User elastic
                    HTTP_Passwd CHANGE_ME
                    tls On
                    tls.ca_file /fluent-bit/tls/tls.crt
                    tls.crt_file /fluent-bit/tls/tls.crt        
                    tls.verify Off
                    Logstash_Format On
                    Logstash_Prefix logstash
                    Retry_Limit False
                    Suppress_Type_Name On
            
                [OUTPUT]
                    Name es
                    Match kube.*
                    Type  _doc
                    Host  <elasticsearch_service_ip>
                    Port 9200
                    HTTP_User elastic
                    HTTP_Passwd CHANGE_ME
                    tls On
                    tls.ca_file /fluent-bit/tls/tls.crt
                    tls.crt_file /fluent-bit/tls/tls.crt        
                    tls.verify Off
                    Logstash_Format On
                    Logstash_Prefix logstash
                    Retry_Limit False
                    Suppress_Type_Name On
            
              ## https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/classic-mode/upstream-servers
              ## This configuration is deprecated, please use `extraFiles` instead.
              upstream: {}
            
              ## https://docs.fluentbit.io/manual/pipeline/parsers
              customParsers: |
                [PARSER]
                    Name docker_no_time
                    Format json
                    Time_Keep Off
                    Time_Key time
                    Time_Format %Y-%m-%dT%H:%M:%S.%L
            
              # This allows adding more files with arbitrary filenames to /fluent-bit/etc/conf by providing key/value pairs.
              # The key becomes the filename, the value becomes the file content.
              extraFiles: {}
            #     upstream.conf: |
            #       [UPSTREAM]
            #           upstream1
            #
            #       [NODE]
            #           name       node-1
            #           host       127.0.0.1
            #           port       43000
            #     example.conf: |
            #       [OUTPUT]
            #           Name example
            #           Match foo.*
            #           Host bar
            
            # The config volume is mounted by default, either to the existingConfigMap value, or the default of "fluent-bit.fullname"
            volumeMounts:
              - name: config
                mountPath: /fluent-bit/etc/conf
            
            daemonSetVolumes:
              - name: varlog
                hostPath:
                  path: /var/log
              - name: varlibdockercontainers
                hostPath:
                  path: /var/lib/docker/containers
              - name: etcmachineid
                hostPath:
                  path: /etc/machine-id
                  type: File
            
            daemonSetVolumeMounts:
              - name: varlog
                mountPath: /var/log
              - name: varlibdockercontainers
                mountPath: /var/lib/docker/containers
                readOnly: true
              - name: etcmachineid
                mountPath: /etc/machine-id
                readOnly: true
            
            command:
              - /fluent-bit/bin/fluent-bit
            
            args:
              - --workdir=/fluent-bit/etc
              - --config=/fluent-bit/etc/conf/fluent-bit.conf
            
            # This supports either a structured array or a templatable string
            initContainers: []
            
            # Array mode
            # initContainers:
            #   - name: do-something
            #     image: bitnami/kubectl:1.22
            #     command: ['kubectl', 'version']
            
            # String mode
            # initContainers: |-
            #   - name: do-something
            #     image: bitnami/kubectl:{{ .Capabilities.KubeVersion.Major }}.{{ .Capabilities.KubeVersion.Minor }}
            #     command: ['kubectl', 'version']
            
            logLevel: info
            
            hotReload:
              enabled: false
              image:
                repository: ghcr.io/jimmidyson/configmap-reload
                tag: v0.11.1
                digest:
                pullPolicy: IfNotPresent
              resources: {}



After editing that, we can deploy our  fluentbit with the command


      helm install fluent-bit fluent/fluent-bit  -f fluentbit.yml -n observer 


Notice an error here in pod creation  why this error? let's check it out

            kubectl logs <fluentbit-pod -n observer>

            


here we see _could not create TLS backend_, this is because we have referenced tls-cert, but we have not mounted the secret to the volume

let's fix that:
edit the configuration with the command

            kubectl edit daemonsets.apps -n observer fluent-bit 


edit the mountpath by adding this:

      volumeMounts:
      - mountPath: /fluent-bit/tls
        name: tls-certs
        readOnly: true # Important for security
      volumes:
      - name: tls-certs
        secret:
          secretName: quickstart-es-http-certs-public



now let's confirm the pods are running:

 let's now get the Password for the fluentbit so we can query the logs sent to elastic search

 ## Now let's visulaise our kibana
First let's set the elastic search to cluster IP and leave only the kibana as Nodeport

Then we access the kibana on port 31896 as shown 

 


- [DisplayofFilebeat_stack](https://github.com/user-attachments/assets/d240980f-a3bb-4299-ad37-1909921942dc)




















































































