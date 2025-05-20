ART (A Reporting Tool) Kubernetes Deployment
This README.md provides comprehensive instructions for deploying ART (A Reporting Tool) within a Kubernetes cluster.

Table of Contents
Overview

Prerequisites

Deployment Steps

1. Download and Extract ART

2. Create the Dockerfile

3. Build the Docker Image

4. Create the Kubernetes Deployment and Service File

5. Deploy to Kubernetes

6. Verify the Deployment

7. Access ART

Troubleshooting

Overview
This guide outlines the process of containerizing the ART application using Docker and deploying it onto a Kubernetes cluster. ART will be exposed via a NodePort Service, making it accessible from outside the cluster.

Prerequisites
Before you begin, ensure you have the following installed and configured:

Kubernetes Cluster: A running Kubernetes cluster (e.g., Minikube, kind, or a cloud-managed cluster like GKE, EKS, AKS).

kubectl: The Kubernetes command-line tool, configured to connect to your target cluster.

Docker: Docker Engine installed on your local machine to build the container image.

ART WAR File: The art.war file. You can download the ART distribution from https://netix.dl.sourceforge.net/project/art/art/8.7/art-8.7.zip.

Deployment Steps
Follow these steps to deploy ART to your Kubernetes cluster.

1. Download and Extract ART
First, download the ART application archive and extract the art.war file.

wget https://netix.dl.sourceforge.net/project/art/art/8.7/art-8.7.zip
unzip art-8.7.zip

After unzipping, ensure that the art.war file is located at art/art-8.7/art.war relative to the directory where you will place your Dockerfile. For example, if your Dockerfile is in ./art-deployment/, then art.war should be in ./art-deployment/art/art-8.7/art.war.

2. Create the Dockerfile
Create a file named Dockerfile in the root of your deployment directory (e.g., art-deployment/) with the following content:

# Use the official Tomcat image as a base
FROM tomcat:9.0

# Copy your WAR file to the Tomcat webapps directory
COPY art/art-8.7/art.war /usr/local/tomcat/webapps/

# Expose the default Tomcat port
EXPOSE 8080

# Start Tomcat
CMD ["catalina.sh", "run"]

This Dockerfile sets up a Tomcat 9.0 server and deploys the art.war application into its webapps directory.

3. Build the Docker Image
Navigate to the directory containing your Dockerfile and the art directory (which contains art-8.7/art.war), then build the Docker image.

docker build -t art:latest .

This command builds a Docker image named art with the tag latest. This image will be used by your Kubernetes Deployment.

4. Create the Kubernetes Deployment and Service File
Create a file named art-deployment.yaml with the following Kubernetes configuration. This file defines both a Service to expose ART and a Deployment to manage its Pods.

apiVersion: v1
kind: Service
metadata:
  name: art-svc
  namespace: asad
  labels:
    app: art
spec:
  type: NodePort
  selector:
    app: art
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 31180
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: art
  name: art-deployment
  namespace: asad
spec:
  replicas: 1
  selector:
    matchLabels:
      app: art
  template:
    metadata:
      labels:
        app: art
    spec:
      containers:
      - image: art:latest
        name: art
        ports:
        - containerPort: 8080

Service (art-svc): Exposes the ART application on port 80 internally and on NodePort 31180 externally. It targets port 8080 on the ART container.

Deployment (art-deployment): Creates a single replica of the ART application using the art:latest Docker image.

5. Deploy to Kubernetes
Apply the Kubernetes YAML file to your cluster. First, ensure the asad namespace exists, then deploy the resources.

kubectl create namespace asad # Skip if namespace already exists
kubectl apply -f art-deployment.yaml

6. Verify the Deployment
After applying the configuration, verify that your Pods and Service are running as expected.

kubectl get pods -n asad -l app=art
kubectl get svc -n asad art-svc

You should see your ART Pod in a Running state and the art-svc showing the NodePort mapping (e.g., 80:31180/TCP).

7. Access ART
Once the deployment is successful, you can access ART using the IP address of any of your Kubernetes Nodes and the specified NodePort.

Get Node IP:

If using Minikube:

minikube ip

For other clusters, get the EXTERNAL-IP of one of your worker nodes:

kubectl get nodes -o wide

Access ART in your browser:
Open your web browser and navigate to:

http://<Node-IP-Address>:31180/art/

Replace <Node-IP-Address> with the actual IP address obtained in the previous step. The /art/ path is crucial as Tomcat deploys WAR files under their base name.

Troubleshooting
ImagePullBackOff:

Ensure your Docker image art:latest is available to your Kubernetes cluster.

If using Minikube, load the image into Minikube's Docker daemon:

eval $(minikube docker-env)
docker build -t art:latest .
eval $(minikube docker-env -u) # Switch back if needed

For production clusters, push your image to a Docker registry (e.g., Docker Hub, Google Container Registry) and update the image field in art-deployment.yaml to point to the registry path (e.g., your-registry/art:latest).

Pod CrashLoopBackOff:

Check the logs of the failing Pod to diagnose the issue:

kubectl logs -n asad <pod-name>

(Replace <pod-name> with the actual name of your ART Pod, e.g., art-deployment-xxxxx-yyyyy).

Service not accessible:

Verify that targetPort: 8080 in the Service matches containerPort: 8080 in the Deployment.

Ensure that the chosen nodePort (31180) is not already in use on your Kubernetes Nodes and is within the allowed NodePort range (typically 30000-32767).

Check network policies or firewall rules that might be blocking access to the NodePort.
