pipeline {
    agent any

    environment {
        IMAGE_NAME = "myapp"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        APP_ENV    = "production"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m venv jenkins-venv
                    . jenkins-venv/bin/activate

                    pip install --upgrade pip
                    pip install -r app/requirements.txt

                    PYTHONPATH=. pytest
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image \
                        --severity HIGH,CRITICAL \
                        --format table \
                        -o trivy-report.txt \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    cat trivy-report.txt
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    export IMAGE_NAME=${IMAGE_NAME}
                    export IMAGE_TAG=${IMAGE_TAG}
                    export APP_ENV=${APP_ENV}

                    docker compose up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10

                    curl -f http://localhost:5000/health

                    echo ""
                    echo "Application health check passed!"
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                    docker image prune -f
                '''
            }
        }
    }

    post {

        always {
            archiveArtifacts artifacts: 'trivy-report.txt',
                allowEmptyArchive: true
        }

        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }
    }
}
