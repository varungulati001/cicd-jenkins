# DevOps Environment Setup Guide

This guide provides detailed instructions to set up a complete CI/CD environment using Jenkins, Docker, Nexus, SonarQube, AWS CLI, and Kubernetes on an Ubuntu server.

---

## Table of Contents
1. [System Update](#system-update)
2. [Jenkins Installation](#jenkins-installation)
3. [Docker Installation](#docker-installation)
4. [Nexus and SonarQube Setup](#nexus-and-sonarqube-setup)
5. [Jenkins Pipeline and Git Integration](#jenkins-pipeline-and-git-integration)
6. [Maven and Trivy Setup](#maven-and-trivy-setup)
7. [AWS CLI, eksctl, and kubectl Setup](#aws-cli-eksctl-and-kubectl-setup)
8. [Kubernetes Deployment](#kubernetes-deployment)
9. [Docker and Jenkins Configuration](#docker-and-jenkins-configuration)
10. [Plugins Installed](#plugins-installed)

---

### 1. System Update

```bash
sudo apt update
sudo apt upgrade -y
2. Jenkins Installation
Install Jenkins along with OpenJDK.
```
bash
Copy code
# Import Jenkins key and add the repository
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt install openjdk-17-jdk -y
sudo apt-get install jenkins -y

# Enable and start Jenkins service
sudo systemctl enable jenkins
sudo systemctl start jenkins
3. Docker Installation
Install Docker and add Jenkins to the Docker group.

bash
Copy code
curl https://get.docker.com | bash
sudo usermod -aG docker jenkins
newgrp docker
sudo systemctl enable docker

# Restart Jenkins for Docker changes to take effect
sudo systemctl stop jenkins
sudo systemctl start jenkins
4. Nexus and SonarQube Setup
Nexus as a Docker Container
bash
Copy code
mkdir nexus && chmod 777 nexus
docker run -d --name nexus -p 8081:8081 -v /home/ubuntu/nexus:/nexus-data sonatype/nexus3:latest
SonarQube as a Docker Container
bash
Copy code
mkdir sonar && chmod 777 sonar
docker run -d --name sonar -p 9000:9000 -v /home/ubuntu/sonar:/opt/sonarqube/data \
                                        -v /home/ubuntu/sonar/extension:/opt/sonarqube/extension \
                                        -v /home/ubuntu/sonar/logs:/opt/sonarqube/logs \
                                        sonarqube:lts-community
5. Jenkins Pipeline and Git Integration
Create Git Credentials in Jenkins.

Install Eclipse Plugin.

Set Up Pipeline in Jenkins using the following script:

groovy
Copy code
pipeline {
    agent any
    stages {
        stage('Git Checkout') {
            steps {
                git branch: "prod", credentialsId: 'git_cred', url: 'https://github.com/varungulati001/cicd-jenkins.git'
            }
        }
    }
}
Enable Continuous Integration by configuring GitHub Webhook:

GitHub: Go to Repo Settings > Webhooks and paste Jenkins URL with /github-webhook/ at the end.
Jenkins: Go to Pipeline > Configure > Build Triggers and check "GitHub hook trigger for GITScm polling".
6. Maven and Trivy Setup
Maven Integration in Jenkins
Install Maven Integration plugin.
Go to Manage Jenkins > Global Tool Configuration, scroll to Maven and add a new Maven installation.
Trivy Installation
bash
Copy code
sudo su -c 'echo "jenkins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers'
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

# Run Trivy scan
trivy fs .
7. AWS CLI, eksctl, and kubectl Setup
AWS CLI Installation
bash
Copy code
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
eksctl Installation
bash
Copy code
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
kubectl Installation
bash
Copy code
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod 777 kubectl
sudo mv kubectl /usr/local/bin
8. Kubernetes Deployment
Create EKS Cluster
bash
Copy code
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --version 1.28 \
  --nodegroup-name ng-high-ip \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 5 \
  --max-pods-per-node 110 \
  --ssh-access \
  --ssh-public-key <YOUR_SSH_KEY_NAME>
Deployment and Service YAML
Deployment (deployment-services.yaml):

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: board-deployment
  labels:
    app: boardgame
spec:
  replicas: 5
  selector:
    matchLabels:
      app: boardgame
  template:
    metadata:
      labels:
        app: boardgame
    spec:
      containers:
      - name: boardgame
        image: varungulati01/boardshack:latest
        ports:
        - containerPort: 8080
Service (svc.yaml):

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: boardgame-svc
spec:
  selector:
    app: boardgame
  type: LoadBalancer
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 8080
9. Docker and Jenkins Configuration
Install Docker and Docker Pipeline plugins in Jenkins.
Add Docker credentials to Jenkins' global credentials.
Add Docker login and stages to your Jenkinsfile.
Dockerfile Example:

Dockerfile
Copy code
FROM adoptopenjdk/openjdk11
EXPOSE 8080
ENV APP_HOME /usr/src/app
COPY target/*.jar $APP_HOME/app.jar
WORKDIR $APP_HOME
CMD ["java", "-jar", "app.jar"]
10. Plugins Installed
Eclipse Temurin JDK
Pipeline Maven
SonarQube Scanner
Docker
Docker Pipeline
Kubernetes
