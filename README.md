# fold-service
Fold Backend Service
# CI/CD Deployment for Fold Backend using Jenkins, Docker, and AWS EC2

## Overview

This project implements a **CI/CD pipeline** to automatically build, package, and deploy a backend service using **Jenkins**, **Docker**, **Docker Hub**, and **AWS EC2**.

Whenever a new commit is pushed to the GitHub repository, Jenkins automatically:

1. Pulls the latest code
2. Builds a Docker image
3. Pushes the image to Docker Hub
4. Deploys the updated container to the application server

This ensures a **fully automated deployment workflow** with minimal manual intervention.

---

# Architecture

```
Developer
   │
   │ git push
   ▼
GitHub Repository
   │
   │ Webhook Trigger
   ▼
Jenkins CI Server (EC2)
   │
   │ Build Docker Image
   │ Push Image → Docker Hub
   ▼
Application Server (EC2)
   │
   │ Pull latest Docker Image
   │ Restart Container
   ▼
Running Backend Service
```

---

# Infrastructure Setup

## Jenkins Server (EC2)

Responsibilities:

* CI/CD pipeline execution
* Docker image build
* Push image to Docker Hub
* SSH deployment to application server

Installed software:

* Jenkins
* Docker
* Git

---

## Application Server (EC2)

Responsibilities:

* Pull Docker images from Docker Hub
* Run application containers
* Host the backend service

Installed software:

* Docker

---

# Technologies Used

* Jenkins
* Docker
* Docker Hub
* AWS EC2
* GitHub
* SSH
* Environment Variables for secrets

---

# Repository

GitHub Repository:

```
https://github.com/shouvickp/fold-service
```

Docker Image Repository:

```
docker.io/shouvickp/fold-backend
```

---

# CI/CD Pipeline Workflow

The pipeline consists of several stages:

1. Clone Repository
2. Build Docker Image
3. Push Image to Docker Hub
4. Cleanup Docker Images
5. Deploy Container to Application Server

---

# Jenkins Pipeline

The CI/CD pipeline is defined using a Jenkinsfile.

```
pipeline {
    agent any

    environment {
        DOCKER_URL = "docker.io/shouvickp"
        APP_SERVER = "ubuntu@13.126.188.229"
        DOCKER_IMAGE = "shouvickp/fold-backend"
        VERSION = "latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/shouvickp/fold-service.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$VERSION .'
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    withCredentials ([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sh '''
                        echo $ARTIFACTORY_PASSWORD | docker login -u ${ARTIFACTORY_USER} --password-stdin
                        docker push $DOCKER_IMAGE:$VERSION
                        '''
                    }
                }
            }
        }

        stage('Cleanup Docker') {
            steps {
                sh 'docker system prune -f'
            }
        }

        stage('Deploy to Server') {
            steps {
                withCredentials([
                    string(credentialsId: 'MONGO_URI', variable: 'MONGO_URI'),
                    string(credentialsId: 'JWT_SECRET', variable: 'JWT_SECRET')
                ]) {
                    sh """
                    ssh $APP_SERVER '
                    docker pull $DOCKER_IMAGE:$VERSION
                    docker stop fold-backend || true
                    docker rm fold-backend || true
                    docker run -d -p 8000:8000 \
                    -e MONGO_URI="${MONGO_URI}" \
                    -e JWT_SECRET="${JWT_SECRET}" \
                    --name fold-service \
                    $DOCKER_IMAGE:$VERSION
                    '
                    """
                }
            }
        }
    }
}
```

---

# Secrets Management

Sensitive values are stored securely in **Jenkins Credentials**.

Configured secrets:

```
dockerhub
MONGO_URI
JWT_SECRET
```

These secrets are injected into the pipeline during runtime.

---

# Deployment Process

When a developer pushes new code to the repository:

```
git add .
git commit -m "update feature"
git push origin main
```

The following process occurs automatically:

1. GitHub triggers Jenkins using a webhook
2. Jenkins pulls the latest code
3. Jenkins builds a Docker image
4. Jenkins logs in to Docker Hub
5. Jenkins pushes the image
6. Jenkins connects to the EC2 application server via SSH
7. The old container is stopped and removed
8. A new container is started with updated code

---

# Docker Deployment Command

The container is deployed using:

```
docker run -d -p 8000:8000 \
-e MONGO_URI=<mongo-uri> \
-e JWT_SECRET=<jwt-secret> \
--name fold-service \
shouvickp/fold-backend:latest
```

---

# Application Access

Once deployed, the application is available at:

```
http://<SERVER_IP>:8000
```

Example:

```
http://13.126.188.229:8000
```

---

# Automatic Trigger using GitHub Webhooks

GitHub is configured to notify Jenkins when new commits are pushed.

Webhook URL:

```
http://<JENKINS_SERVER_IP>:8080/github-webhook/
```

Trigger event:

```
Push events
```

---

# Docker Cleanup

The pipeline removes unused Docker images to prevent disk space issues:

```
docker system prune -f
```

This keeps the Jenkins server clean and avoids storage exhaustion.

---

# Verification

Check running containers on the application server:

```
docker ps
```

Expected output:

```
CONTAINER ID   IMAGE                     PORTS
xxxx           shouvickp/fold-backend    0.0.0.0:8000->8000
```

---