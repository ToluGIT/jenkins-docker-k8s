pipeline {
    agent any

    environment {
        IMAGE_NAME = 'toluid/tester'
        IMAGE_TAG = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
        KUBECONFIG = credentials('kubeconfig')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Set Up Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Unit Testing') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
                junit '**/test-reports/*.xml'  // Optional: Archive test reports
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([
                    string(credentialsId: 'dockerusr', variable: 'DOCKER_USER'),
                    string(credentialsId: 'dockerpwd', variable: 'DOCKER_PASS')
                ]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image ${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }
        
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    echo "Scanning Docker image: ${IMAGE_TAG}"
                    
                    def scanOutput = sh(script: "tmas scan docker:${IMAGE_TAG} -VMS --region ap-southeast-1", returnStdout: true).trim()
                    echo "Scan Output: ${scanOutput}"

                    if (scanOutput.contains('"severity": "Critical"')) {
                        error("Critical severity vulnerability detected! Pipeline will be terminated.")
                    } else {
                        echo("No critical severity vulnerabilities detected. Pipeline will continue.")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image ${IMAGE_TAG}"
                    sh "docker push ${IMAGE_TAG}"
                }
            }
        }

        stage('Approval Required') {
            steps {
                script {
                    def userInput = input(
                        id: 'userInput', 
                        message: 'Approve and Provide Docker Image Tag', 
                        parameters: [string(name: 'DOCKER_IMAGE', description: 'Enter the Docker image tag')]
                    )
                    env.DOCKER_IMAGE = "${IMAGE_NAME}:${userInput}"
                }
            }
        }

        stage('Deploy to EKS Prod Environment') {
            steps {
                script {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: 'aws')]) {
                        def deploymentExists = sh(script: "kubectl get deployment flask-app-deployment-prod -n default -o json", returnStatus: true)

                        if (deploymentExists == 0) {
                            echo "Updating Kubernetes deployment with new Docker image ${env.DOCKER_IMAGE}"
                            sh "kubectl set image -n default deployment/flask-app-deployment-prod flask-app=${env.DOCKER_IMAGE} --record"
                        } else {
                            echo "Creating new Kubernetes deployment"
                            sh "sed -i 's|image: .*|image: ${env.DOCKER_IMAGE}|' deployment.yaml"
                            sh "kubectl apply -f deployment.yaml"
                        }

                        echo "Kubernetes deployment completed successfully"
                        sh "kubectl rollout status deployment/flask-app-deployment-prod -n default"
                    }
                }
            }
        }
    }
}
