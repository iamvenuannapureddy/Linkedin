<h1>Project-14: Kubernetes Setup for Production</h1>

**Project Overview**:
In this project, we'll focus on deploying a **Kubernetes (K8s)** setup for a Java web application in a **production environment**. Kubernetes provides a highly available, scalable, and self-healing platform for container orchestration, making it a popular choice for production workloads. We'll cover setting up Kubernetes clusters, deploying microservices (or a monolithic Java app), configuring persistent storage, ensuring security, and setting up monitoring and auto-scaling.

### **Key Kubernetes Concepts**:
- **Kubernetes Cluster**: A group of nodes (worker and master) that run containers.
- **Pods**: The smallest deployable units in Kubernetes, which run one or more containers.
- **Services**: Expose pods to the outside world or internally within the cluster.
- **Ingress**: Manage external access to the services in the cluster.
- **Persistent Volumes (PV)**: Allow for persistent storage in Kubernetes.
- **ConfigMaps and Secrets**: Store configuration data and sensitive information securely.

---

### **Project Architecture**:
- **Kubernetes Cluster**: Hosted on **AWS (EKS)**, **Azure (AKS)**, or **Google Cloud (GKE)**.
- **Java Application**: Containerized using Docker and deployed as a pod in Kubernetes.
- **Database**: A managed service like **RDS/MySQL/Postgres** or **MongoDB** with persistent storage.
- **Ingress Controller**: For routing traffic (e.g., NGINX Ingress).
- **Helm**: Used for managing Kubernetes manifests.
- **Monitoring**: **Prometheus** and **Grafana** for metrics and dashboards.
- **CI/CD**: Integrated with **GitHub Actions** or **Jenkins** for automated deployment.

---

### **Project Steps**:

#### **Step 1: Set Up Kubernetes Cluster**

1. **Option 1: AWS EKS (Elastic Kubernetes Service)**:
   - **Create an EKS Cluster** using the AWS Management Console or AWS CLI.
   - Use **eksctl** for easy setup:
     ```bash
     eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --nodes 3 --nodes-min 2 --nodes-max 5
     ```
   - This will create an EKS cluster with auto-scaling nodes.

2. **Option 2: Azure AKS (Azure Kubernetes Service)**:
   - Use Azure CLI to create an AKS cluster:
     ```bash
     az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3 --enable-addons monitoring --generate-ssh-keys
     ```

3. **Option 3: Google GKE (Google Kubernetes Engine)**:
   - Use `gcloud` to create a GKE cluster:
     ```bash
     gcloud container clusters create my-cluster --zone us-central1-a --num-nodes=3
     ```

4. **Configure kubectl**:
   - After creating the cluster, configure **kubectl** (Kubernetes command-line tool) to interact with the cluster:
     ```bash
     aws eks update-kubeconfig --name my-cluster --region us-east-1
     ```

---

#### **Step 2: Containerize Your Application (Java with Docker)**
1. **Dockerfile**:
   - Create a `Dockerfile` for your Java application to containerize it. For a Spring Boot application, the `Dockerfile` could look like:

     ```dockerfile
     FROM openjdk:11-jre-slim
     COPY target/mywebapp.jar /usr/app/mywebapp.jar
     WORKDIR /usr/app
     EXPOSE 8080
     CMD ["java", "-jar", "mywebapp.jar"]
     ```

2. **Build and Push Docker Image**:
   - Build the Docker image and push it to a container registry like **Docker Hub** or **Amazon ECR**:
     ```bash
     docker build -t mywebapp .
     docker tag mywebapp:latest mydockerhub/mywebapp:latest
     docker push mydockerhub/mywebapp:latest
     ```

---

#### **Step 3: Create Kubernetes Deployment and Service**
1. **Deployment YAML**:
   - Define a Kubernetes Deployment that specifies how to deploy your application’s container:

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: mywebapp-deployment
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: mywebapp
       template:
         metadata:
           labels:
             app: mywebapp
         spec:
           containers:
           - name: mywebapp
             image: mydockerhub/mywebapp:latest
             ports:
             - containerPort: 8080
             env:
             - name: DATABASE_URL
               valueFrom:
                 secretKeyRef:
                   name: db-credentials
                   key: database-url
     ```

2. **Service YAML**:
   - Define a service to expose the Java application to the outside world:

     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: mywebapp-service
     spec:
       selector:
         app: mywebapp
       ports:
         - protocol: TCP
           port: 80
           targetPort: 8080
       type: LoadBalancer
     ```

   This service will expose your application to the internet by using a cloud provider's load balancer.

---

#### **Step 4: Configure Ingress for Routing**
1. **Ingress Controller**:
   - Install an **NGINX Ingress Controller** to manage traffic routing to your services:
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
     ```

2. **Ingress Resource**:
   - Create an ingress resource to define routing rules:

     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: mywebapp-ingress
     spec:
       rules:
       - host: mywebapp.example.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: mywebapp-service
                 port:
                   number: 80
     ```

---

#### **Step 5: Persistent Storage for Databases**
1. **Persistent Volume (PV) and Persistent Volume Claim (PVC)**:
   - For databases like MySQL or MongoDB, set up persistent storage:

     ```yaml
     apiVersion: v1
     kind: PersistentVolume
     metadata:
       name: my-pv
     spec:
       capacity:
         storage: 20Gi
       accessModes:
         - ReadWriteOnce
       persistentVolumeReclaimPolicy: Retain
       storageClassName: gp2
       awsElasticBlockStore:
         volumeID: <volume-id>
         fsType: ext4
     ```

     ```yaml
     apiVersion: v1
     kind: PersistentVolumeClaim
     metadata:
       name: my-pvc
     spec:
       accessModes:
         - ReadWriteOnce
       resources:
         requests:
           storage: 20Gi
     ```

2. **Mount PVC to Database Pod**:
   - In your database pod definition, mount the PVC:

     ```yaml
     spec:
       containers:
       - name: mysql
         image: mysql:5.7
         volumeMounts:
         - name: mysql-storage
           mountPath: /var/lib/mysql
       volumes:
       - name: mysql-storage
         persistentVolumeClaim:
           claimName: my-pvc
     ```

---

#### **Step 6: Set Up Monitoring and Logging**
1. **Prometheus and Grafana for Monitoring**:
   - Use **Prometheus** to scrape metrics from Kubernetes and **Grafana** for visualizing these metrics:
     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     helm repo update
     helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack
     ```

2. **Fluentd for Log Aggregation**:
   - Install **Fluentd** to collect logs from your pods and send them to a centralized logging platform (e.g., **Elasticsearch**, **CloudWatch**):
     ```bash
     helm install fluentd stable/fluentd
     ```

---

#### **Step 7: Enable Auto-Scaling**
1. **Horizontal Pod Autoscaler (HPA)**:
   - Set up **HPA** to automatically scale your pods based on CPU or memory usage:

     ```yaml
     apiVersion: autoscaling/v1
     kind: HorizontalPodAutoscaler
     metadata:
       name: mywebapp-hpa
     spec:
       scaleTargetRef:
         apiVersion: apps/v1
         kind: Deployment
         name: mywebapp-deployment
       minReplicas: 3
       maxReplicas: 10
       targetCPUUtilizationPercentage: 75
     ```

---

#### **Step 8: Continuous Deployment with GitHub Actions or Jenkins**
1. **GitHub Actions CI/CD**:
   - Integrate your Kubernetes deployment with GitHub Actions for automated deployments. Add a step in your workflow to deploy to Kubernetes:

     ```yaml
     - name: Deploy to Kubernetes
       uses: azure/k8s-deploy@v1
       with:
         kubeconfig: ${{ secrets.KUBECONFIG }}
         manifests: |
           ./k8s/deployment.yaml
           ./k8s/service.yaml
         images: mydockerhub/mywebapp:latest
     ```

2. **Jenkins Pipeline**:
   - For Jenkins, you can use **Kubernetes Plugin** to deploy from Jenkins pipelines.

---

#### **Step 9: Secure the Application**
1. **RBAC (Role-Based Access Control)**:
   - Ensure your Kubernetes cluster has **RBAC** policies enabled to control who can access the resources.

2. **Secrets Management**:
   - Use **Kubernetes Secrets** to store sensitive data like database credentials:

     ```yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: db-credentials
     type: Opaque
     data:
       database-url: bXlzcWw6Ly9teXVzZXI6bXlwYXNzQG15ZGI=
     ```

---

#### **Step 10: Monitor and Optimize the Production Setup**
1. **Use Prometheus Alerts**: Set up alerting rules to get notified when CPU, memory, or storage usage crosses a threshold.
2. **Cluster Autoscaler**: Enable the cluster autoscaler to automatically add/remove worker nodes based on the resource demand.


.
