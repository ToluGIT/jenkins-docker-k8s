# Flask Todo Application with Jenkins CI/CD Pipeline

A containerized Flask web application demonstrating a complete CI/CD pipeline using Jenkins, Docker, and Kubernetes deployment to Amazon EKS.

## Architecture Overview

This project implements a modern DevOps workflow:
- Flask web application with todo management functionality
- Docker containerization for consistent deployment
- Jenkins pipeline for automated CI/CD
- Kubernetes deployment on Amazon EKS
- Automated testing with pytest

## Project Structure

```
├── app.py                  # Flask application entry point
├── requirements.txt        # Python dependencies
├── Dockerfile             # Docker image configuration
├── Jenkinsfile            # Jenkins pipeline definition
├── deployment.yaml        # Kubernetes deployment manifest
├── test_app.py           # Unit tests
├── templates/
│   └── index.html        # Frontend template
├── flask.service         # Systemd service configuration
├── steps.sh              # Setup script
└── tasks.txt            # Project notes
```

## Application Features

### Flask Todo Application
- Create and manage todo items
- In-memory storage for task persistence
- Responsive web interface
- RESTful POST/GET operations

### Key Components
- **Backend**: Flask 3.0.1 with Jinja2 templating
- **Frontend**: HTML/CSS interface with form handling
- **Storage**: Dictionary-based in-memory storage
- **Testing**: Comprehensive pytest suite

## Prerequisites

### Development Environment
- Python 3.12+
- Docker Engine
- kubectl configured for EKS access
- Jenkins server with required plugins

### Required Jenkins Plugins
- Docker Pipeline
- Kubernetes CLI
- AWS Credentials
- Pipeline: AWS Steps

### AWS Infrastructure
- Amazon EKS cluster
- DockerHub registry access
- IAM roles with appropriate permissions

## Local Development

### Installation
```bash
# Clone repository
git clone https://github.com/ToluGIT/jenkins-docker-k8s.git
cd jenkins-docker-k8s

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Running Locally
```bash
# Start development server
python app.py

# Access application
http://localhost:5000
```

### Testing
```bash
# Run unit tests
pytest

# Run with coverage
pytest --cov=app
```

## Docker Deployment

### Build Image
```bash
docker build -t flask-todo:latest .
```

### Run Container
```bash
docker run -p 5000:5000 flask-todo:latest
```

## Jenkins Pipeline

### Pipeline Stages

1. **Checkout Code**
   - Repository checkout with delay for stability

2. **Environment Setup**
   - Python virtual environment creation
   - Dependency installation from requirements.txt

3. **Unit Testing**
   - Automated pytest execution
   - Test result validation

4. **DockerHub Authentication**
   - Secure credential handling
   - Registry login verification

5. **Docker Image Build**
   - Multi-stage build process
   - Image tagging with build number

6. **Image Registry Push**
   - Automated push to DockerHub
   - Tag management and versioning

7. **Manual Approval**
   - Human approval gate
   - Custom image tag specification

8. **EKS Production Deployment**
   - Kubernetes deployment update
   - Rolling update strategy
   - Resource allocation and limits

### Required Jenkins Credentials
- `dockerusr`: DockerHub username
- `dockerpwd`: DockerHub password
- `aws`: AWS access keys for EKS
- `kubeconfig`: Kubernetes configuration file

## Kubernetes Deployment

### Deployment Configuration
- **Replicas**: 1 instance
- **Resource Limits**: 1 CPU, 500Mi memory
- **Resource Requests**: 0.5 CPU, 200Mi memory
- **Container Port**: 5000
- **Image Pull Secret**: regcred for DockerHub access

### Manual Deployment
```bash
# Update image in deployment.yaml
sed -i 's|image: .*|image: toluid/tester:BUILD_NUMBER|' deployment.yaml

# Apply to cluster
kubectl apply -f deployment.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
```

## Configuration

### Environment Variables
- `IMAGE_NAME`: Docker image repository name
- `IMAGE_TAG`: Dynamic tagging with build numbers
- `KUBECONFIG`: Kubernetes cluster configuration

### Docker Configuration
- **Base Image**: python:3.12.0b3-alpine3.18
- **Working Directory**: /application
- **Exposed Port**: 5000
- **Entry Point**: python app.py


### Debugging Commands
```bash
# Check application logs
kubectl logs deployment/flask-app-deployment-prod

# Describe deployment status
kubectl describe deployment flask-app-deployment-prod

# Port forward for local testing
kubectl port-forward deployment/flask-app-deployment-prod 5000:5000
```

## Security Considerations

- Secure credential management through Jenkins
- Image vulnerability scanning recommended
- Network policies for Kubernetes deployment
- Regular dependency updates for security patches
