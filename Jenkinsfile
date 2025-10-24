pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        IMAGE_NAME = 'flask-todo'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Code checkout happens automatically from SCM"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=flask-todo \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL ${IMAGE_NAME}:latest || true'
            }
        }

        stage('Run Container (Smoke Test)') {
            steps {
                sh '''
                    docker run -d --rm -p 5000:5000 --name ci-smoke ${IMAGE_NAME}:latest
                    sleep 3
                    curl -s http://localhost:5000/tasks || true
                    docker stop ci-smoke
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed, check logs!'
        }
    }
}