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

        stage('Build Docker Image') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    try {
                        echo "Building Docker image ${IMAGE_TAG}..."
                        sh "docker build -t ${IMAGE_TAG} ."
                    } catch (Exception e) {
                        error "Docker build failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('TMAS Vulnerability Scan') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'TMAS_API_KEY', variable: 'TMAS_API_KEY')]) {
                        echo "Scanning Docker image ${IMAGE_TAG} for vulnerabilities..."
                        try {
                            def scanOutput = sh(
                                script: "tmas scan docker:${IMAGE_TAG} -VMS --region ap-southeast-1",
                                returnStdout: true,
                                env: [TMAS_API_KEY: TMAS_API_KEY]
                            ).trim()

                            echo "Scan Output: ${scanOutput}"

                            if (scanOutput.contains('"severity": "Critical"')) {
                                error "Critical severity vulnerability detected! Pipeline stopped."
                            } else {
                                echo "No critical vulnerabilities detected. Continuing pipeline."
                            }
                        } catch (Exception e) {
                            error "Vulnerability scan failed: ${e.getMessage()}"
                        }
                    }
                }
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

        stage('Push Docker Image') {
            steps {
                script {
                    try {
                        echo "Pushing Docker image ${IMAGE_TAG} to DockerHub..."
                        sh "docker push ${IMAGE_TAG}"
                    } catch (Exception e) {
                        error "Failed to push Docker image: ${e.getMessage()}"
                    }
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
                        try {
                            def deploymentExists = sh(
                                script: "kubectl get deployment flask-app-deployment-prod -n default -o json",
                                returnStatus: true
                            )

                            if (deploymentExists == 0) {
                                echo "Updating Kubernetes deployment with new Docker image ${env.DOCKER_IMAGE}..."
                                sh "kubectl set image -n default deployment/flask-app-deployment-prod flask-app=${env.DOCKER_IMAGE} --record"
                            } else {
                                echo "Creating new Kubernetes deployment..."
                                sh "sed -i 's|image: .*|image: ${env.DOCKER_IMAGE}|' deployment.yaml"
                                sh "kubectl apply -f deployment.yaml"
                            }

                            echo "Deployment initiated. Monitoring rollout status..."
                            sh "kubectl rollout status deployment/flask-app-deployment-prod -n default"
                        } catch (Exception e) {
                            error "Deployment to EKS failed: ${e.getMessage()}"
                        }
                    }
                }
            }
        }
    }
}
