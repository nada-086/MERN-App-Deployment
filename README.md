## Project Overview

This architecture shows a **CI/CD pipeline** that automates the process of building, testing, scanning, and deploying a MERN stack application using tools like **Jenkins**, **Docker**, **Trivy**, **ArgoCD**, and **Kubernetes**.

---

## 🧑‍💻 **Step 1: Developer Pushes Code**

- A **developer** writes code and pushes it to **GitHub**.

---

## 🛠️ **Step 2: Jenkins CI Job (Continuous Integration)**

Once code is pushed to GitHub, a **Jenkins pipeline** is triggered to handle CI tasks:

1. **Pull code** from GitHub.  
2. **Install dependencies**  
3. **Run static code analysis** using **Trivy** to check for code quality, bugs, and vulnerabilities.

After passing the code analysis:

1. **Build Docker image** of the application.
2. **Scan Docker image** with **Trivy** to check for security vulnerabilities.
3.  **Push the image** to the **Artifact Registry** (a Docker image repository).

---

## 🚀 **Step 3: ArgoCD CD Job (Continuous Deployment)**

After the Docker image is pushed:

1. **ArgoCD Image Updater** detects the new image in the Artifact Registry.  
2. It **updates the image version** in the Kubernetes manifests or Helm charts stored in GitHub.  
3. These updates are **written back to GitHub** (GitOps approach).

---

## 🌐 **Step 4: ArgoCD Deploys to Kubernetes**

1. **ArgoCD** monitors GitHub for changes.  
2. It **pulls the latest manifests** and **deploys the new image** to the Kubernetes cluster.  
3. **Prometheus** collects performance and metrics data.  
4. **Grafana** visualizes the metrics for monitoring.



# Setup Instructions
1. **Install Jenkins**  
   - Install Jenkins on your machine by following the official Jenkins installation guide for your operating system.
 [Jenkins](https://www.jenkins.io/doc/book/installing/linux/)
   - Install plugins needed (Docker, Docker pipeline, SonarQube Scanner)
     
3. **Run SonarQube as a Docker Container**  
   1. **Start SonarQube using Docker with the following command:**  
   ```bash
   docker run -d --name SonarQube -p 9000:9000 SonarQube:lts
   ```
    Access SonarQube at **http://\<your-server-ip\>:9000**.

   2. **SonarQube Initial Configuration:**
   
      - Set up an admin account and create a new token for Jenkins.
      - **Navigate to:**  Administration > Security > Users > Tokens.
      - Create a token and give it a name, e.g., jenkins-token.
   
4. **install trivy on the machine for security scanning**
   [Trivy](https://trivy.dev/v0.57/getting-started/installation/)
``` bash
cat << EOF | sudo tee -a /etc/yum.repos.d/trivy.repo
[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://aquasecurity.github.io/trivy-repo/rpm/public.key
EOF
sudo yum -y update
sudo yum -y install trivy
```
# Configurations Instructions
**1. Jenkins**
   - Add SonarQube credintials to docker
   - Add Docker credintials
   - Add github repo
   - Create pipeline & add Jenkins File
