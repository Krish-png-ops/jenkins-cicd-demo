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
                echo "Checking out source code from GitHub repository main branch..."
                checkout scm
                sh 'git log -1 --format="Commit: %H | Author: %an | Message: %s"'
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
                    
                    # Page title / heading validation
                    grep -qi "CI/CD Pipeline Demo" index.html || { echo "ERROR: Missing Heading"; exit 1; }
                    
                    # Student name and PRN validation
                    grep -q "${STUDENT_NAME}" index.html || { echo "ERROR: Missing Student Name (${STUDENT_NAME})"; exit 1; }
                    grep -q "${STUDENT_PRN}" index.html || { echo "ERROR: Missing PRN (${STUDENT_PRN})"; exit 1; }
                    
                    # Version validation (ensures Version label and v1.0 are present with flexible whitespace)
                    grep -qi "Version" index.html || { echo "ERROR: Missing 'Version' label"; exit 1; }
                    grep -q "${EXPECTED_VERSION}" index.html || { echo "ERROR: Missing Version string (${EXPECTED_VERSION})"; exit 1; }
                    grep -Eqi "Version[[:space:]]+v1\\.0|v1\\.0" index.html || { echo "ERROR: Missing Version v1.0 pattern"; exit 1; }
                    
                    echo "SUCCESS: Code quality, student details, and version validation passed."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker container image for build #${BUILD_NUMBER}..."
                sh '''
                    docker build -t ${ECR_REPO}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    echo "SUCCESS: Docker image built with tags: latest and ${BUILD_NUMBER}."
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo "Authenticating with Amazon ECR via EC2 IAM role and pushing image..."
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest
                    echo "SUCCESS: Image pushed to Amazon ECR: ${IMAGE_NAME}:${BUILD_NUMBER} and latest"
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                echo "Configuring Kubernetes cluster and deploying unique build tag..."
                sh '''
                    if [ -f /var/lib/jenkins/.kube/config ]; then
                        export KUBECONFIG=/var/lib/jenkins/.kube/config
                    else
                        export KUBECONFIG=/home/ec2-user/.kube/config
                    fi

                    # Update or create ECR pull secret in Kubernetes
                    kubectl delete secret ecr-secret --ignore-not-found
                    ECR_TOKEN=$(aws ecr get-login-password --region ${AWS_REGION})
                    kubectl create secret docker-registry ecr-secret \
                        --docker-server=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com \
                        --docker-username=AWS \
                        --docker-password="${ECR_TOKEN}"
                    
                    # Apply deployment and service manifests
                    kubectl apply -f k8s/service.yaml
                    kubectl apply -f k8s/deployment.yaml

                    # Update deployment image to the unique build tag
                    echo "Setting deployment image to unique build tag: ${IMAGE_NAME}:${BUILD_NUMBER}..."
                    kubectl set image deployment/task-tracker task-tracker=${IMAGE_NAME}:${BUILD_NUMBER}
                    
                    # Wait for deployment rollout completion
                    kubectl rollout status deployment/task-tracker --timeout=180s
                    echo "SUCCESS: Kubernetes deployment rolled out with unique build tag ${IMAGE_NAME}:${BUILD_NUMBER}."
                '''
            }
        }

        stage('Verification') {
            steps {
                echo "Verifying Kubernetes pods, deployment and application availability..."
                sh '''
                    if [ -f /var/lib/jenkins/.kube/config ]; then
                        export KUBECONFIG=/var/lib/jenkins/.kube/config
                    else
                        export KUBECONFIG=/home/ec2-user/.kube/config
                    fi

                    kubectl get pods -l app=task-tracker -o wide
                    kubectl get deployment task-tracker -o wide
                    kubectl get svc task-tracker -o wide
                    
                    # Verify deployed image matches this build number
                    DEPLOYED_IMAGE=$(kubectl get deployment task-tracker -o jsonpath='{.spec.template.spec.containers[0].image}')
                    echo "Verified deployed container image: ${DEPLOYED_IMAGE}"
                    if [ "${DEPLOYED_IMAGE}" != "${IMAGE_NAME}:${BUILD_NUMBER}" ]; then
                        echo "ERROR: Deployed image does not match build tag ${IMAGE_NAME}:${BUILD_NUMBER}!"
                        exit 1
                    fi

                    # Check pod running state
                    RUNNING_PODS=$(kubectl get pods -l app=task-tracker --no-headers | grep "Running" | wc -l)
                    if [ "${RUNNING_PODS}" -lt 1 ]; then
                        echo "ERROR: No pods running for task-tracker!"
                        exit 1
                    fi
                    
                    # Probe local application endpoint on port 30080
                    echo "Testing application HTTP response on port 30080..."
                    curl -fs -I http://127.0.0.1:30080/ || {
                        echo "ERROR: Application endpoint did not return HTTP 200 OK!"
                        exit 1
                    }

                    echo "SUCCESS: All verification checks passed. Application is live on port 30080."
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Task Tracker application for ${STUDENT_NAME} (${STUDENT_PRN}) is deployed on Kubernetes (Image: ${IMAGE_NAME}:${BUILD_NUMBER})."
        }
        failure {
            echo "Pipeline failed. Check stage logs for details."
        }
    }
}
