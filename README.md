# MindCircuitr13

Welcome to **MindCircuitr13**, a collection of hands-on practices and projects in  **AWS**, and **CI/CD pipeline development**. This repository showcases DevOps concepts and implementations, ranging from infrastructure provisioning to deploying containerized applications with Kubernetes. Explore, learn, and feel free to contribute!

---

## Repository Overview

###  **CI/CD Pipeline for Java Applications**
- **Description**: End-to-end Jenkins pipeline automating the CI/CD process for Java applications.
- **Key Tools**: Git, Jenkins, Maven, SonarQube, Docker Hub, ArgoCD, and Kubernetes (EKS).
- **Features**:
  - Automating builds and static analysis using SonarQube.
  - Creating Docker images and pushing to Docker Hub.
  - Deploying to EKS using ArgoCD.
- **GitHub Link**: [https://github.com/KatamreddyVishnu/mindcircuit13](#)

![image](https://github.com/user-attachments/assets/928d0c7a-d158-410d-bde3-ea0a826f359a)



---

## Prerequisites

### Software Requirements
- AWS CLI
- Jenkins
- Docker
- Maven
- SonarQube
- Kubernetes CLI (kubectl) & eksctl
- ArgoCD

### Server Requirements
- A Linux server (t2.large) for Jenkins, Docker, and SonarQube setup.
- EKS Kubernetes cluster.

---

## Key Project: CI/CD Pipeline for Java Applications

### Objective:
To establish a robust CI/CD pipeline for automating the build, test, and deployment process.

### Flow:
1. **Checkout Code**: Cloning from GitHub.
2. **Static Analysis**: SonarQube scan for quality checks.
3. **Build Artifact**: Using Maven to create JAR/WAR files.
4. **Build Docker Image**: Dockerizing the application.
5. **Push to Docker Hub**: Pushing images for deployment.
6. **Update Deployment**: Adjusting Kubernetes manifests and syncing with Git.
7. **Deploy with ArgoCD**: Automating Kubernetes deployments.

### Setup Guide:
1. **Jenkins Installation**:
   ```bash
   sudo yum update -y
   sudo yum install java-17-amazon-corretto -y
   sudo yum install jenkins -y
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   ```

2. **Docker Installation**:
   ```bash
   sudo yum install docker -y
   sudo systemctl start docker
   sudo usermod -aG docker jenkins
   sudo chmod 666 /var/run/docker.sock
   ```

3. **AWS CLI Configuration**:
   ```bash
   aws configure
   ```

4. **Kubernetes CLI Setup**:
   - Install `kubectl` and `eksctl`.
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   ```

5. **EKS Cluster Creation**:
   ```bash
   eksctl create cluster --name mcappcluster --nodegroup-name mcng --node-type t3.micro --nodes 8 --managed
   ```

6. **ArgoCD Setup**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

---

## Additional Files

### Jenkinsfile
```groovy
pipeline {
    agent any


       tools {
        maven 'maven3'
    }

    stages {
      stage('checkout') {
            steps {
                echo 'Cloning GIT HUB Repo '
				git branch: 'main', url: 'https://github.com/Vishnu15-dev/mindcircuit13.git'
            }  
        }
		
		
		
	 stage('sonar') {
            steps {
                echo 'scanning project'
                sh 'ls -ltr'
                
                sh ''' mvn sonar:sonar \\
                      -Dsonar.host.url=http://52.90.208.180:9000 \\
                      -Dsonar.login=squ_5a3d5a03fb7df89a1dc234984be37ffaaffac326'''
            }
    	}
		
		
		
        stage('Build Artifact') {
            steps {
                echo 'Build Artifact'
				sh 'mvn clean package'
            }
        }
		
		
		
        stage('Docker Image') {
            steps {
                echo 'Docker Image building'
				sh 'docker build -t katamreddyvishnu/batch13:${BUILD_NUMBER} .'
            }
        }
		
		
       stage('Push to Dockerhub') {
            steps {
			 script {
			withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) 
			{
            sh 'docker login -u katamreddyvishnu -p ${dockerhub}'
			
			 }
			   sh 'docker push katamreddyvishnu/batch13:${BUILD_NUMBER}'
			   
           
				}
				
            }
        }
		
		
    stage('Update Deployment File') {
		
		 environment {
            GIT_REPO_NAME = "mindcircuit13"
            GIT_USER_NAME = "Vishnu15-dev"
        }
		
            steps {
                echo 'Update Deployment File'
				withCredentials([string(credentialsId: 'githubtoken', variable: 'githubtoken')]) 
				{
                  sh '''
                    git config user.email "vishnu@gmail.com"
                    git config user.name "Vishnu"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/batch13:.*/batch13:${BUILD_NUMBER}/g" deploymentfiles/deployment.yml
                    git add .
                    
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"

                    git push https://${githubtoken}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
				  
                 }
				
            }
        }
		
		
			
    }

}
```

### deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mc-app
  labels:
    app: mc-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: mc-app
  template:
    metadata:
      labels:
        app: mc-app
    spec:
      containers:
      - name: mc-app
        image: katamreddyvishnu/batch13:3
        ports:
        - containerPort: 8080
```

### service.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mc-app-service
spec:
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: 8080
  selector:
    app: mc-app
```

---




