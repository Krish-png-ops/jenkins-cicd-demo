pipeline {
    agent any

    environment {
        DEPLOY_PATH = '/usr/share/nginx/html'
        EXPECTED_VERSION = 'v1.0'
        STUDENT_NAME = 'Krishnav Wadhera'
        STUDENT_PRN = '1262243616'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code from repository..."
                checkout scm
            }
        }

        stage('Validate File') {
            steps {
                echo "Validating index.html existence and integrity..."
                sh '''
                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html not found!"
                        exit 1
                    fi
                    echo "SUCCESS: index.html exists."
                '''
            }
        }

        stage('Test & Quality Check') {
            steps {
                echo "Running validation checks on index.html content..."
                sh '''
                    grep -q "CI/CD Pipeline Demo" index.html || { echo "Missing Heading"; exit 1; }
                    grep -q "Krishnav Wadhera" index.html || { echo "Missing Student Name"; exit 1; }
                    grep -q "1262243616" index.html || { echo "Missing PRN"; exit 1; }
                    grep -q "${EXPECTED_VERSION}" index.html || { echo "Missing or incorrect version"; exit 1; }
                    echo "SUCCESS: All quality tests and student verification checks passed."
                '''
            }
        }

        stage('Deploy to Web Server') {
            steps {
                echo "Deploying index.html to Nginx web root at ${DEPLOY_PATH}..."
                sh '''
                    cp -f index.html ${DEPLOY_PATH}/index.html
                    chmod 644 ${DEPLOY_PATH}/index.html
                    echo "SUCCESS: Page deployed to Nginx."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Verifying HTTP response and deployed version from local server..."
                sh '''
                    HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/)
                    echo "HTTP Status Code: ${HTTP_STATUS}"
                    if [ "${HTTP_STATUS}" -ne 200 ]; then
                        echo "ERROR: Web server returned HTTP ${HTTP_STATUS} instead of 200"
                        exit 1
                    fi

                    curl -s http://localhost/ | grep -q "${EXPECTED_VERSION}" || {
                        echo "ERROR: Deployed page does not show expected version ${EXPECTED_VERSION}"
                        exit 1
                    }
                    echo "SUCCESS: Web page returned HTTP 200 and correctly displays ${EXPECTED_VERSION}!"
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! CI/CD demonstration for ${STUDENT_NAME} (${STUDENT_PRN}) is LIVE."
        }
        failure {
            echo "Pipeline failed. Check stage logs for details."
        }
    }
}
