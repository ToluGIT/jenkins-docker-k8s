# **CI/CD Pipeline with Flask Sample Application**
=================================================

This project demonstrates a **complete CI/CD pipeline** using Jenkins for a simple Flask-based web application. The primary focus is to **showcase the CI/CD process**, including:

1. **Testing and building a Docker image**.  
2. **Vulnerability scanning** using TMAS.  
3. **Pushing the image to DockerHub**.  
4. **Deploying to AWS EKS** (Elastic Kubernetes Service).

---
* * *

**Goal of the Project**
-----------------------

* * *

The primary objective is to **demonstrate a complete CI/CD process** using **Jenkins, Docker, and Kubernetes**. The **Flask app is a sample microservice**, designed to walk through:

1.  **Automated Testing**.
2.  **Containerization with Docker**.
3.  **Vulnerability Scanning**.
4.  **Pushing to DockerHub**.
5.  **Deploying to AWS-hosted Kubernetes (EKS)**.

* * *

**Pipeline Overview**
---------------------

* * *

The pipeline is defined in the **Jenkinsfile**, automating all key steps. Below is a high-level breakdown of the process:

### **Pipeline Steps (Jenkinsfile Snippet):**

        pipeline {
    stages {
        stage('Run Unit Tests') { ... }
        stage('Build Docker Image') { ... }
        stage('TMAS Vulnerability Scan') { ... }
        stage('Push Docker Image') { ... }
        stage('Deploy to EKS') { ... }
      }
    }
    
**Vulnerability Scanning (TMAS Integration)**
---------------------------------------------

* * *

One critical step in the pipeline is **vulnerability scanning** using **TMAS** (Trend Micro Application Security). This ensures that no Docker image with **critical vulnerabilities** gets deployed to production.

### **How the Scan Works:**

*   **Scan Command**: The image is scanned after it is built using the following command:

        tmas scan docker:${IMAGE_TAG} -VMS --region ap-southeast-1

*   **Critical Vulnerabilities**: If any **"Critical" severity** vulnerabilities are found, the pipeline **fails immediately** and stops further deployment.
    

### **Code Snippet from Jenkinsfile:**

    stage('TMAS Vulnerability Scan') {
        steps {
            script {
                withCredentials([string(credentialsId: 'TMAS_API_KEY', variable: 'TMAS_API_KEY')]) {
                    def scanOutput = sh(
                        script: "tmas scan docker:${IMAGE_TAG} -VMS --region ap-southeast-1",
                        returnStdout: true
                    ).trim()
    
                    if (scanOutput.contains('"severity": "Critical"')) {
                        error "Critical vulnerability detected! Pipeline stopped."
                    } else {
                        echo "No critical vulnerabilities detected."
                    }
                }
            }
        }
     }
* * *

**Deployment to AWS EKS**
-------------------------

* * *

The pipeline deploys the app to an **AWS EKS cluster** using `kubectl`. If a deployment already exists, it **updates the container image**. Otherwise, it **creates a new deployment**.

### **Deployment Commands:**
      kubectl set image -n default deployment/flask-app-deployment-prod \
          flask-app=toluid/tester:latest --record
      kubectl rollout status deployment/flask-app-deployment-prod -n default

**CI/CD Pipeline Workflow**
---------------------------

* * *

1.  **Checkout Code**: Jenkins pulls the latest code from the version control system.
2.  **Run Unit Tests**: The code is validated using `pytest` to ensure basic functionality.
3.  **Build Docker Image**: A Docker image is built for the Flask app.
4.  **TMAS Vulnerability Scan**: The Docker image is scanned for vulnerabilities. The pipeline halts if any **critical issues** are found.
5.  **Push to DockerHub**: The image is pushed to **DockerHub**.
6.  **Manual Approval**: A manual checkpoint requires a **human review** before proceeding with production deployment.
7.  **Deploy to EKS**: The app is deployed or updated on an **AWS EKS cluster**

* * *


---------

**Sample Flask Application Overview**
-------------------------------------

* * *

The **Flask app** is a minimal task manager with the following features:

*   **Add Tasks**: Users can add tasks via a web form.
*   **Delete Tasks**: Tasks can be deleted by providing the task ID.
*   **In-Memory Storage**: Tasks are stored in a **dictionary** without persistence (this is only for demo purposes).


---------------------------------------------------------------------------------------------------------------------------------------------------------

**Why Use This Project?**
-------------------------

* * *

This project demonstrates a **complete CI/CD process** that:

*   Automates testing, building, scanning, and deployment of containerized applications.
*   Follows **security best practices** with vulnerability scanning using **TMAS**.
*   Uses **Jenkins** to integrate and deploy applications to **AWS EKS**.

**Conclusion**
--------------

* * *

This project showcases a **complete CI/CD pipeline** with Jenkins, Docker, and Kubernetes. The focus is on demonstrating the **process of automating builds, tests, vulnerability scans, and deployments**. While the Flask app is simple, it serves as a perfect example to highlight the key steps involved in a DevOps workflow.
  

