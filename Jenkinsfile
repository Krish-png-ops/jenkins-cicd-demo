pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '660463066160'
        ECR_REPO = 'task-tracker'
        IMAGE_NAME = '660463066160.dkr.ecr.ap-south-1.amazonaws.com/task-tracker'
        STUDENT_NAME = 'Krishnav Wadhera'
        STUDENT_PRN = '1262243616'
        EXPECTED_VERSION = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code from GitHub repository..."
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                echo "Running validation and quality tests on project files..."
                sh '''
                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html not found!"
                        exit 1
                    fi
                    grep -q "CI/CD Pipeline Demo" index.html || { echo "ERROR: Missing Heading"; exit 1; }
                    grep -q "${STUDENT_NAME}" index.html || { echo "ERROR: Missing Student Name"; exit 1; }
                    grep -q "${STUDENT_PRN}" index.html || { echo "ERROR: Missing PRN"; exit 1; }
                    grep -q "${EXPECTED_VERSION}" index.html || { echo "ERROR: Missing Version"; exit 1; }
                    echo "SUCCESS: Code quality and integrity validation passed."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker container image..."
                sh '''
                    docker build -t ${ECR_REPO}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    echo "SUCCESS: Docker image built successfully."
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo "Authenticating with Amazon ECR and pushing image..."
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker push ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    echo "SUCCESS: Image pushed to Amazon ECR repository: ${IMAGE_NAME}:latest"
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                echo "Configuring Kubernetes cluster and deploying application..."
                sh '''
                    export KUBECONFIG=/home/ec2-user/.kube/config
                    # Update or create ECR pull secret in Kubernetes
                    kubectl delete secret ecr-secret --ignore-not-found
                    ECR_TOKEN=$(aws ecr get-login-password --region ${AWS_REGION})
                    kubectl create secret docker-registry ecr-secret \
                        --docker-server=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com \
                        --docker-username=AWS \
                        --docker-password="${ECR_TOKEN}"
                    
                    # Apply deployment and service manifests
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    
                    # Wait for deployment rollout
                    kubectl rollout status deployment/task-tracker --timeout=180s
                    echo "SUCCESS: Kubernetes deployment and NodePort service applied."
                '''
            }
        }

        stage('Verification') {
            steps {
                echo "Verifying Kubernetes pods, deployment and application availability..."
                sh '''
                    export KUBECONFIG=/home/ec2-user/.kube/config
                    kubectl get pods -l app=task-tracker -o wide
                    kubectl get deployment task-tracker
                    kubectl get svc task-tracker
                    
                    # Check pod running state
                    RUNNING_PODS=$(kubectl get pods -l app=task-tracker --no-headers | grep "Running" | wc -l)
                    if [ "${RUNNING_PODS}" -lt 1 ]; then
                        echo "ERROR: No pods running for task-tracker!"
                        exit 1
                    fi
                    
                    echo "SUCCESS: All verification checks passed. Application is live on port 30080."
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Task Tracker application for ${STUDENT_NAME} (${STUDENT_PRN}) is deployed on Kubernetes."
        }
        failure {
            echo "Pipeline failed. Check stage logs for details."
        }
    }
}
