# Jenkins Documentation
This document outlines the stages of the Jenkins pipeline, including their purpose and functionality.
## Pipeline Steps

**1. Add SonarQube and Docker Credentials to Docker**

**2. Add GitHub Repository**

**3. Create Pipeline & Add Jenkinsfile**

---
### Jenkins Pipeline Stages Explanation

### Stage 1: Cleanup Working Directory
```bash
stages {
    stage ('Cleanup Working Directory') {
        steps {
            script {
                cleanWs()
            }
        }
    }
}
```
- The function cleans the workspace directory used by the Jenkins job.
It deletes:
- All files and folders in the workspace
- Any leftover files from previous builds

### Stage 2 : trivy file scan
``` bash
stage('trivy file scan') {
    steps {
        sh "trivy fs --format table -o fs.html ."
    }
}
```
- This stage runs a file system vulnerability scan using Trivy.

### Stage 3: SonarQube: Code Analysis

```bash
    stage('SonarQube: Code Analysis') {
      steps {
        withSonarQubeEnv("sonar"){
          sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=bookstore  -Dsonar.projectKey=bookstore -X"
        }
      }
    }
```
- **withSonarQubeEnv("sonar"):**
    - This sets up the environment to use a configured SonarQube instance (named `"sonar"` in Jenkins settings).
    - It injects environment variables like `SONAR_HOME`, `SONAR_HOST_URL`, etc., required for the scanner to work.
- **sh "$SONAR_HOME/bin/sonar-scanner ..."**:
    - Executes the **SonarQube Scanner** (CLI tool).
    - It runs a scan on the source code and sends the results to the SonarQube server.
    - The flags:
        - `Dsonar.projectName=bookstore`: Human-readable project name in SonarQube.
        - `Dsonar.projectKey=bookstore`: Unique identifier used by SonarQube.
        - `X`: Enables debug output for troubleshooting.
## Stage 4: SonarQube: Code Quality Gates
``` bash 
    stage("SonarQube: Code Quality Gates"){
      steps {
        timeout(time: 7, unit: "MINUTES"){
          waitForQualityGate abortPipeline: false
        }
      }
    }
```
- **waitForQualityGate**:
    - This waits for SonarQube to finish analyzing the results and return a **quality gate status**.
    - A **quality gate** is a set of rules (e.g., no new bugs, maintain test coverage) that must be passed for the build to be considered healthy.
- **timeout(time: 7, unit: "MINUTES")**:
    - Sets a timeout to wait for the result (up to 7 minutes).
    - Prevents Jenkins from hanging indefinitely.
- **abortPipeline: false**:
    - If the quality gate fails, **the pipeline will continue** (i.e., it logs the failure but does not stop the build).
    - If you set this to `true`, the pipeline would stop on a failed quality gate.
## Stage 5: Build Frontend Docker Image
  📋 Description
This Jenkins pipeline stage builds a Docker image for the frontend part of the application and pushes it to Docker Hub. Each image is tagged with the current Jenkins build number for versioning and traceability.
 🔧 Steps Explained
1-Login to Docker Hub :
  Uses Jenkins credentials (ID: dockerhub) to securely authenticate with Docker Hub.
``` bash 
withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
}
```
2-Navigate to the frontend directory then Build Docker Image
``` bash 

     sh '''
          cd ./src/frontend
          docker build --tag nadaessa/bookstore-frontend:v${BUILD_NUMBER} .
          docker push nadaessa/bookstore-frontend:v${BUILD_NUMBER}
        '''
```
3-Push Docker Image to Docker Hub
``` bash 
docker push nadaessa/bookstore-frontend:v${BUILD_NUMBER}
```
### Stage 6: Build Backend Docker Image
📋 Description
This Jenkins pipeline stage builds a Docker image for the backend part of the application and pushes it to Docker Hub. Each image is tagged with the current Jenkins build number for versioning and traceability.

🔧 Steps Explained

Login to Docker Hub
Uses Jenkins credentials (ID: dockerhub) to securely authenticate with Docker Hub.

``` bash
withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
}
```
2-Navigate to the backend directory & Build Docker Image
  Changes to the backend directory, builds the Docker image, and tags it with the Jenkins build number.
``` bash

sh '''
    cd ./src/backend
    docker build --tag nadaessa/bookstore-backend:v${BUILD_NUMBER} .
    docker push nadaessa/bookstore-backend:v${BUILD_NUMBER}
'''
```
3-Push Docker Image to Docker Hub
 Uploads the built image to Docker Hub.

``` bash
docker push nadaessa/bookstore-backend:v${BUILD_NUMBER}
```








