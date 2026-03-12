pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Security: Gitleaks') {
            steps {
                echo 'Scanning for secrets and credentials...'
                sh '''
                    docker run --rm \
                    -v $(pwd):/path \
                    zricethezav/gitleaks:latest \
                    detect --source /path --verbose \
                    --no-git || true
                '''
            }
        }

        stage('Security: Trivy') {
            steps {
                echo 'Scanning for vulnerabilities...'
                sh '''
                    trivy fs \
                    --severity HIGH,CRITICAL \
                    --exit-code 0 \
                    --format table \
                    .
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh '''
                    if [ -f "pom.xml" ]; then
                        echo "Java/Maven project detected"
                        mvn clean package -DskipTests || true
                    elif [ -f "package.json" ]; then
                        echo "Node.js project detected"
                        npm install || true
                    elif [ -f "requirements.txt" ]; then
                        echo "Python project detected"
                        pip install -r requirements.txt || true
                    else
                        echo "No specific build file found, skipping build"
                    fi
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    if [ -f "Dockerfile" ]; then
                        docker build -t mccs-pipeline:${BUILD_NUMBER} .
                        echo "Docker image built successfully"
                    else
                        echo "No Dockerfile found, skipping Docker build"
                    fi
                '''
            }
        }

        stage('Security: Image Scan') {
            steps {
                echo 'Scanning Docker image...'
                sh '''
                    if docker image inspect mccs-pipeline:${BUILD_NUMBER} > /dev/null 2>&1; then
                        trivy image \
                        --severity HIGH,CRITICAL \
                        --exit-code 0 \
                        mccs-pipeline:${BUILD_NUMBER}
                    else
                        echo "No image to scan, skipping"
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh 'echo Deployment stage - configure based on your target environment'
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs above.'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
