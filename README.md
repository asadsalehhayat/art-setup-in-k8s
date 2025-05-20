# ART Application Deployment on Kubernetes

This guide provides step-by-step instructions to deploy the ART application on a Kubernetes cluster using Docker.

## 1. Download and Extract ART

Download the ART application archive and extract the `art.war` file:

```bash
wget https://netix.dl.sourceforge.net/project/art/art/8.7/art-8.7.zip
unzip art-8.7.zip
```


After unzipping, ensure that the art.war file is located at art/art-8.7/art.war relative to the directory where you will place your Dockerfile. For example, if your Dockerfile is in ./art-deployment/, then art.war should be in ./art-deployment/art/art-8.7/art.war.

## 2. Create the Dockerfile
Create a file named Dockerfile in the root of your deployment directory (e.g., art-deployment/) with the following content:


## 3. Build the Docker Image
Navigate to the directory containing your Dockerfile and the art directory (which contains art-8.7/art.war), then build the Docker image.

docker build -t art:latest .

This command builds a Docker image named art with the tag latest. This image will be used by your Kubernetes Deployment.

## 4. Create the Kubernetes Deployment and Service File
Create a file named art-deployment.yaml with the following Kubernetes configuration. This file defines both a Service to expose ART and a Deployment to manage its Pods.

Service (art-svc): Exposes the ART application on port 80 internally and on NodePort 31180 externally. It targets port 8080 on the ART container.

Deployment (art-deployment): Creates a single replica of the ART application using the art:latest Docker image.

## 5. Deploy to Kubernetes
Apply the Kubernetes YAML file to your cluster. First, ensure the asad namespace exists, then deploy the resources.

kubectl create namespace asad # Skip if namespace already exists
kubectl apply -f art-deployment.yaml

## 6. Verify the Deployment
After applying the configuration, verify that your Pods and Service are running as expected.

kubectl get pods -n asad -l app=art
kubectl get svc -n asad art-svc

You should see your ART Pod in a Running state and the art-svc showing the NodePort mapping (e.g., 80:31180/TCP).

## 7. Access ART
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


For production clusters, push your image to a Docker registry (e.g., Docker Hub, Google Container Registry) and update the image field in art-deployment.yaml to point to the registry path (e.g., your-registry/art:latest).

Pod CrashLoopBackOff:

Check the logs of the failing Pod to diagnose the issue:

kubectl logs -n asad <pod-name>

(Replace <pod-name> with the actual name of your ART Pod, e.g., art-deployment-xxxxx-yyyyy).

Service not accessible:

Verify that targetPort: 8080 in the Service matches containerPort: 8080 in the Deployment.

Ensure that the chosen nodePort (31180) is not already in use on your Kubernetes Nodes and is within the allowed NodePort range (typically 30000-32767).

Check network policies or firewall rules that might be blocking access to the NodePort.
