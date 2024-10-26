# **CI/CD Pipeline with Flask Sample Application**

This project demonstrates a **complete CI/CD pipeline** using Jenkins for a simple Flask-based web application. The primary focus is to **showcase the CI/CD process**, including:

* **Checkout Code**: Jenkins pulls the latest code from the repository.
* **Set Up Environment**: A Python virtual environment is created, and dependencies are installed.
* **Run Unit Tests**: The code is validated using `pytest` to ensure functionality.
* **Build Docker Image**: A Docker image is built only if tests pass.
* **Vulnerability Scan**: The Docker image is scanned for vulnerabilities using TMAS.
* **Login to DockerHub**: The image is pushed to DockerHub after successful login.
* **Push Docker Image**: The built image is pushed to DockerHub for version control.
* **Approval Required**: A manual approval step ensures human oversight before deployment.
* **Deploy to EKS**: The app is deployed or updated in an AWS EKS cluster.

---

## **Pipeline Overview**

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

------

## **Key Features Aligned with DevOps Best Practices**
------------------------------------------------------

This pipeline is designed with several **DevOps best practices** to ensure quality, security, and efficiency throughout the software delivery lifecycle

### ** Automated Error Handling and Resilience**
- **`try-catch` blocks** are implemented to **gracefully handle errors** at every critical stage (unit testing, image build, vulnerability scanning, and deployment).  
- If an error occurs (e.g., failed test or scan), the pipeline **stops immediately**, providing **meaningful error messages** for faster debugging.

#### **Example Error Handling Snippet:**
		stage('Run Unit Tests') {
			steps {
				script {
					try {
						echo "Running unit tests with pytest..."
						sh '''
						. venv/bin/activate
						pytest
						'''
					} catch (Exception e) {
						error "Unit tests failed: ${e.getMessage()}"
					}
				}
			}
	       	   }
  
----------------

------
**Container Image Scanning (TMAS Integration)**
---------------------------------------------


One critical step in the pipeline is **vulnerability scanning** using **TMAS**. This ensures that no Docker image with **critical vulnerabilities** gets deployed to production.

### **How the Scan Works:**

*   **Scan Command**: The image is scanned after it is built using the following command:

        tmas scan docker:${IMAGE_TAG} -VMS --region ap-southeast-1

*   **Critical Vulnerabilities**:   If any critical vulnerabilities are detected, the scan stops the pipeline, ensuring only secure images are deployed.
    


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

The pipeline deploys the app to an AWS EKS cluster using 'kubectl'. If a deployment already exists, it updates the container image. Otherwise, **it creates a new deployment**.
The 'kubectl rollout status' command monitors deployment success, allowing for 'rollbacks' in case of failures.

### **Kubernetes Deployment Snippet:**
                try {
                    def deploymentExists = sh(
                        script: "kubectl get deployment flask-app-deployment-prod -n default -o json",
                        returnStatus: true
                    )
                
                    if (deploymentExists == 0) {
                        echo "Updating Kubernetes deployment..."
                        sh "kubectl set image -n default deployment/flask-app-deployment-prod flask-app=${env.DOCKER_IMAGE} --record"
                    } else {
                        echo "Creating new Kubernetes deployment..."
                        sh "kubectl apply -f deployment.yaml"
                    }
                
                    echo "Monitoring rollout status..."
                    sh "kubectl rollout status deployment/flask-app-deployment-prod -n default"
                } catch (Exception e) {
                    error "Deployment to EKS failed: ${e.getMessage()}"
                }

**Sample Flask Application Overview**
-------------------------------------

The **Flask app** is a minimal task manager with the following features:

*   **Add Tasks**: Users can add tasks via a web form.
*   **Delete Tasks**: Tasks can be deleted by providing the task ID.
*   **In-Memory Storage**: Tasks are stored in a **dictionary** without persistence (this is only for demo purposes).


---------------------------------------------------------------------------------------------------------------------------------------------------------

**Why Use This Project?**
-------------------------

This project demonstrates a **complete CI/CD process** that:

*   Automates testing, building, scanning, and deployment of containerized applications.
*   Follows **security best practices** with vulnerability scanning using **TMAS**.
*   Uses **Jenkins** to integrate and deploy applications to **AWS EKS**.

**Conclusion**
--------------

This project showcases a **complete CI/CD pipeline** with Jenkins, Docker, and Kubernetes. The focus is on demonstrating the **process of automating builds, tests, vulnerability scans, and deployments**. While the Flask app is simple, it serves as a perfect example to highlight the key steps involved in a DevOps workflow.
  

