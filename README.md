# Ultimate CI/CD Pipeline Documentation

## 📘 Project Overview

This project demonstrates a robust DevOps Continuous Integration/Continuous Deployment (CI/CD) pipeline for a Java application, deployed on a Kubernetes cluster using Jenkins. It integrates code quality analysis, security scanning, artifact management, containerization, orchestration, and monitoring.

---

## 🎯 Purpose and Objectives

- Automate software delivery from compilation to deployment
- Establish a Jenkins-based CI/CD pipeline
- Integrate DevOps tools:
  - Maven
  - SonarQube
  - Trivy
  - Nexus
  - Docker
  - Kubernetes
  - Prometheus + Grafana
- Improve code quality and detect vulnerabilities
- Achieve consistent and reliable Kubernetes deployments
- Set up email notifications and system monitoring

---

## 🧰 Tools Used

| Tool              | Purpose                                          |
|-------------------|--------------------------------------------------|
| Jenkins           | CI/CD orchestration                              |
| Maven             | Build automation and dependency management       |
| SonarQube         | Static code analysis                             |
| Trivy             | Docker image vulnerability scanning              |
| Nexus Repository  | Artifact storage and versioning                  |
| Docker            | Containerization                                 |
| Kubernetes        | Orchestration and deployment                     |
| Gmail Integration | Email notifications                              |
| Prometheus        | Monitoring                                       |
| Grafana           | Visualization                                    |
| AWS               | Infrastructure provisioning                      |

---

## 🧱 Infrastructure Setup (AWS EC2 Instances)

1. **Kubernetes Master Node**
2. **Kubernetes Worker Nodes (2)**
3. **Jenkins Server**
4. **SonarQube Server**
5. **Nexus Repository**
6. **Monitoring Server (Prometheus + Grafana)**

Security groups and network rules are configured to allow inter-VM communication, especially for Kubernetes and monitoring.

---

## ☸️ Kubernetes Cluster Setup (Kubeadm)

- Disable swap
- Load kernel modules
- Set sysctl params
- Install CRI-O runtime and Kubernetes components
- Initialize master and join worker nodes using `kubeadm`
- Apply Calico as the network plugin

---

## 🧪 Jenkins Server Setup

- Install OpenJDK 17
- Add Jenkins GPG key and repository
- Install Jenkins and start service

---

## 🐳 Docker Installation

Installed on Jenkins, SonarQube, and Nexus servers using secure APT method.

---

## 📦 Nexus Setup

- Run as a Docker container on port 8081
- Access and retrieve admin password from container
- Use for storing Maven artifacts

---

## 🔍 SonarQube Setup

- Run as a Docker container on port 9000
- Access SonarQube web UI for analysis reports

---

## 🔐 Git Repository Setup

- Create a private GitHub repository
- Generate Personal Access Token
- Use `git clone`, `git add`, `commit`, `push`
- Authenticate using the token

Repo: `https://github.com/Shubham-Stunner/BoardGame.git`

---

## 🔄 Jenkins Pipeline

```groovy
pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Shubham-Stunner/BoardGame.git'
            }
        }
        stage('Compile') {
            steps { sh 'mvn compile' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
        }
        stage('Trivy FS Scan') {
            steps { sh 'trivy fs --format table -o trivy-fs-report.html .' }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame -Dsonar.java.binaries=.'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build') {
            steps { sh 'mvn package' }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3') {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Docker Build & Scan') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t stunnershubham/boardgame:latest .'
                        sh 'trivy image --format table -o trivy-image-report.html stunnershubham/boardgame:latest'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push stunnershubham/boardgame:latest'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(...) {
                    sh 'kubectl apply -f deployment-service.yaml'
                    sh 'kubectl get pods -n webapps'
                }
            }
        }
    }
    post {
        always {
            script {
                emailext(
                    subject: "${env.JOB_NAME} - Build ${env.BUILD_NUMBER} - ${currentBuild.result ?: 'UNKNOWN'}",
                    body: "<html><body><h2>Status: ${currentBuild.result}</h2></body></html>",
                    to: 'shubhammukherji654@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}
```

---

## 📈 Monitoring Setup

### Prometheus
- Runs on port 9090
- Configuration in `prometheus.yml` includes blackbox exporter targets

### Grafana
- Installed via `.deb` package
- Runs on port 3000
- Import Prometheus as data source
- Dashboards visualize metrics from Kubernetes, Jenkins, and Blackbox

---

## ✅ Conclusion

The Ultimate CICD Pipeline automates code integration, testing, packaging, deployment, and monitoring, resulting in:
- Faster releases
- Better code quality
- Secure containerization
- Scalable Kubernetes deployments
- Real-time observability

---

## 📚 References

- [Jenkins Docs](https://www.jenkins.io/doc/)
- [Maven Docs](https://maven.apache.org/)
- [SonarQube Docs](https://docs.sonarqube.org/latest/)
- [Trivy Docs](https://github.com/aquasecurity/trivy)
- [Nexus Docs](https://help.sonatype.com/repomanager3)
- [Docker Docs](https://docs.docker.com/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
