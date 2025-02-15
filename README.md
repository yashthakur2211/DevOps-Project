# DevOps Project

This project integrates **Ansible, Jenkins, Docker Desktop, Minikube, Maven, and Apache Tomcat** to establish a robust CI/CD pipeline.

## 🚀 Prerequisites

Ensure you have the following installed before proceeding:

1. **Ansible**
2. **Jenkins** - [Official Website](https://www.jenkins.io/)
3. **Docker Desktop** - [Docker Website](https://www.docker.com/)
4. **Minikube** - [Minikube Start Guide](https://minikube.sigs.k8s.io/docs/start/)
5. **Maven** - [Apache Maven](https://maven.apache.org/)
6. **Apache Tomcat** - [Tomcat Website](https://tomcat.apache.org/)

## 📁 Project Structure

This project involves:

- **Building a Java web application** using Maven
- **Containerizing the application** with Docker
- **Deploying it on Minikube**
- **Managing infrastructure** using Ansible & Jenkins

---

## 🛠️ Installation & Configuration

### 1️⃣ Setup EC2 Instance

Launch an **Amazon Linux 2023** instance (t2.medium) in **ap-south-1** region.

### Update & Install Required Packages

```sh
sudo yum update -y
sudo yum install -y jre maven python git docker
```

### Configure Docker

```sh
sudo usermod -aG docker $USER && newgrp docker
sudo systemctl enable --now docker.service
sudo docker login
```

### Install Kubectl

```sh
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.3/2024-04-19/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
kubectl version --client
```

### Install Minikube

```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -Uvh minikube-latest.x86_64.rpm
minikube start
```

### Install Jenkins

```sh
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install fontconfig java-17-openjdk jenkins -y
```

---

## 🔨 Create a Maven Project

### **pom.xml**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>simple-webapp</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <build>
        <finalName>simple-webapp</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.1</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### **src/main/webapp/index.jsp**

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Simple Web App</title>
</head>
<body>
    <h1>Hello, World!</h1>
</body>
</html>
```

### Maven Build Commands

```sh
mvn clean package
mvn compile
mvn test
mvn verify
mvn install
```

---

## 📦 Dockerizing the Application

### **Dockerfile**

```Dockerfile
FROM tomcat:9.0
COPY target/simple-webapp.war /usr/local/tomcat/webapps/
```

Build and push Docker image:

```sh
docker build -t username/devopsproject .
docker images
docker push username/devopsproject
```

---

## ☸️ Deploying on Minikube

### **k8s-deployment.yml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-webapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-webapp
  template:
    metadata:
      labels:
        app: simple-webapp
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp-service
spec:
  type: NodePort
  selector:
    app: simple-webapp
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30007
```

Apply the deployment:

```sh
kubectl apply -f k8s-deployment.yml
```

---

## 🛠️ Jenkins Setup

### **Jenkinsfile**

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/simple-webapp.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("simple-webapp:latest")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy to Minikube') {
            steps {
                script {
                    sh 'kubectl apply -f k8s-deployment.yml'
                }
            }
        }
    }
}
```

---

## 🤖 Ansible Configuration

### **ansible/playbook.yml**

```yaml
---
- name: Deploy simple-webapp on Kubernetes
  hosts: localhost
  tasks:
    - name: Apply Kubernetes deployment
      command: kubectl apply -f k8s-deployment.yml
```

Run the playbook:

```sh
ansible-playbook ansible/playbook.yml
```

---

## ✅ Running the Project

1. **Build Maven Project**
   ```sh
   mvn clean package
   ```
2. **Run Docker Container**
   ```sh
   docker run -d -p 8080:8080 simple-webapp:latest
   ```
3. **Deploy to Minikube**
   ```sh
   kubectl apply -f k8s-deployment.yml
   ```
4. **Run Ansible Playbook**
   ```sh
   ansible-playbook ansible/playbook.yml
   ```
5. **Start Jenkins and Run Pipeline**
   ```sh
   java -jar jenkins.war
   ```

🎉 **Your DevOps pipeline is now set up!** 🚀

