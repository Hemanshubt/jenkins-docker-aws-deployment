# 🚀 CI/CD Pipeline: AWS + Jenkins + Docker + Maven

[![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=Jenkins&logoColor=white)](https://jenkins.io/)
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Maven](https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=Apache%20Maven&logoColor=white)](https://maven.apache.org/)

**Automated deployment of a Java Web Application on Docker Containers using a Jenkins CI/CD Pipeline on AWS EC2.**

---

## 🏗️ Architecture Overview

```mermaid
graph LR
    A[Source Code] -->|Push| B(GitHub)
    B -->|Webhook/Poll| C{Jenkins Server}
    subgraph AWS Cloud
    C -->|Build & Test| D[Maven]
    D -->|Artifact .war| C
    C -->|SCP Artifact| E{Docker Host}
    subgraph EC2 Instance 2
    E -->|Build Image| F[Docker Engine]
    F -->|Run| G[Tomcat Container]
    end
    end
    G -->|Access| H[End User]
```

---

## 📋 Project Agenda

1.  **Infrastructure Setup**: Provision AWS EC2 instances.
2.  **Toolchain Configuration**: Install & Setup Jenkins, Git, and Maven.
3.  **Environment Integration**: Connect GitHub and Maven with Jenkins.
4.  **Docker Orchestration**: Setup Docker Host and customize containers.
5.  **Pipeline Automation**: Automate end-to-end Build & Deploy process.
6.  **Verification**: Test the live application.

---

## 🛠️ Step-by-Step Implementation

### Phase 1: Infrastructure Setup (AWS EC2)

1. **Launch Jenkins Server**:
    - Instance: Linux EC2 (t2.micro).
    - Security Group: Allow SSH (22) and Jenkins (8080).
2. **Access & Install Jenkins**:
    - Connect via SSH.
    - Install Java OpenJDK 11 and Jenkins.
    - Access via `http://<EC2-IP>:8080`.

<details>
<summary>📂 View Setup Commands & Screenshots</summary>

![EC2 Setup](https://miro.medium.com/v2/resize:fit:750/format:webp/0*dV_5siwtpbY49t_K.png)

```bash
# Register Repo
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Dependencies
sudo yum install java-11-openjdk -y
sudo yum install jenkins -y
sudo systemctl enable --now jenkins
```
</details>

---

### Phase 2: Tool Configuration (Git & Maven)

1. **Git Integration**:
    - Install Git on Jenkins EC2.
    - Install `GitHub Integration` plugin in Jenkins.
2. **Maven Setup**:
    - Download and extract Maven in `/opt`.
    - Configure Environment Variables (`JAVA_HOME`, `M2_HOME`).
    - Setup "Global Tool Configuration" in Jenkins.

<details>
<summary>📂 View Configuration Details</summary>

![Jenkins Plugins](https://miro.medium.com/v2/resize:fit:750/format:webp/0*_3Y_nVohpK-DSmDF.png)

```bash
# Maven Setup
cd /opt
sudo wget https://archive.apache.org/dist/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz
sudo tar -xvzf apache-maven-3.8.6-bin.tar.gz
```
</details>

---

### Phase 3: Docker Host Orchestration

1. **Provision Docker Host**:
    - Launch separate Linux EC2.
    - Install & Start Docker Engine.
2. **Customize Tomcat Image**:
    - Docker default Tomcat images often have empty `webapps` folders.
    - Create a custom `Dockerfile` to fix the 404 error by copying `webapps.dist` to `webapps`.

<details>
<summary>📂 View Docker Customization</summary>

![Docker Container](https://miro.medium.com/v2/resize:fit:750/format:webp/1*hE087O2THC2TsIr-0fW3dA.png)

**Custom Dockerfile:**
```dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
COPY ./*.war /usr/local/tomcat/webapps
```
</details>

---

### Phase 4: CI/CD Pipeline Automation

1. **Integrate Jenkins with Docker**:
    - Create `dockeradmin` user on Docker Host.
    - Install `Publish Over SSH` plugin on Jenkins.
    - Configure SSH settings to connect Jenkins to Docker Host.
2. **Create Pipeline Job**:
    - Set Source Code Management (GitHub URL).
    - Configure "Build Periodically" or "GitHub hook trigger".
    - **Post-build Action**: Send artifacts over SSH and execute deployment commands.

<details>
<summary>📂 View Automation Script</summary>

**Execute over SSH:**
```bash
cd /opt/docker;
docker stop registerapp || true;
docker rm registerapp || true;
docker build -t regapp:v1 .;
docker run -d --name registerapp -p 8087:8080 regapp:v1
```
</details>

---

## ✅ Deployment Success

Once the pipeline runs, your application is live!

![Final Success](https://miro.medium.com/v2/resize:fit:750/format:webp/1*s6KDOzNqAUgYPenBojBnqQ.png)

Access the app: `http://<Docker-Host-IP>:8087/webapp/`

---

## 🏁 Conclusion

This project demonstrates a robust CI/CD workflow, bridging the gap between developers and production using modern DevOps tools on AWS cloud infrastructure.

---
*Maintained by [Heman](https://github.com/hemanshuu)*
