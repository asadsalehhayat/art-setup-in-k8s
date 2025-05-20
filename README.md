# 🚀 ART (A Reporting Tool) Deployment Guide

This guide provides step-by-step instructions for deploying [ART (A Reporting Tool)](https://sourceforge.net/projects/art/) using:

- Docker 🐳  
- containerd (`ctr`) 🔧  
- Kubernetes ☸️  

---

## 📦 Step 1: Download ART

Download and extract the WAR file:

```bash
wget https://netix.dl.sourceforge.net/project/art/art/8.7/art-8.7.zip
unzip art-8.7.zip
Ensure the following path exists:

bash
Copy
Edit
art/art-8.7/art.war
🛠 Step 2: Create Dockerfile
Create a Dockerfile in your project directory with the following contents:

Dockerfile
Copy
Edit
FROM tomcat:9.0

COPY art/art-8.7/art.war /usr/local/tomcat/webapps/

EXPOSE 8080

CMD ["catalina.sh", "run"]
Then build the image:

bash
Copy
Edit
docker build -t art:latest .
🐳 Step 3: Run ART in Docker
Run the container using:

bash
Copy
Edit
docker run -d --name art -p 8080:8080 art:latest
Open your browser and go to:

bash
Copy
Edit
http://localhost:8080/art
🔧 Step 4: Run ART with containerd (ctr)
4.1 Export Docker image as a tarball
bash
Copy
Edit
docker save -o art.tar art:latest
4.2 Import into containerd
bash
Copy
Edit
sudo ctr -n k8s.io images import art.tar
4.3 Run container using ctr
bash
Copy
Edit
sudo ctr -n k8s.io run -d --rm -p 8080:8080 docker.io/library/art:latest art
📌 Or use nerdctl if installed:

bash
Copy
Edit
nerdctl run -d -p 8080:8080 art:latest
Access the app at:
http://localhost:8080/art

☸️ Step 5: Deploy ART on Kubernetes
5.1 Create a namespace
bash
Copy
Edit
kubectl create namespace asad
5.2 Create art-deployment.yaml
Save the following as art-deployment.yaml:

yaml
Copy
Edit
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
5.3 Apply the deployment
bash
Copy
Edit
kubectl apply -f art-deployment.yaml
5.4 Access the app
Get the Node IP:

bash
Copy
Edit
kubectl get nodes -o wide
Then visit in browser:

arduino
Copy
Edit
http://<NodeIP>:31180/art
If using Minikube:

bash
Copy
Edit
minikube service art-svc -n asad --url
✅ Summary
Environment	Command
Docker	docker run -d -p 8080:8080 art:latest
containerd	ctr -n k8s.io run -d -p 8080:8080 art
Kubernetes	kubectl apply -f art-deployment.yaml
URL	http://localhost:8080/art or http://<NodeIP>:31180/art

📎 References
ART on SourceForge

Tomcat Docker Image

Kubernetes Documentation

containerd

yaml
Copy
Edit
