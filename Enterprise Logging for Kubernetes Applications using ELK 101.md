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

