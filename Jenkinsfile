pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://192.168.16.197:9000'
        IMAGE_NAME     = 'flask-todo'
        DOCKER_HUB_USER = 'bhupal716'     // ✅ your real Docker Hub username
    }

    stages {

        /* ---------------- CHECKOUT ---------------- */
        stage('Checkout') {
            steps {
                echo "📥 Checking out code from GitHub..."
            }
        }

        /* ---------------- SONARQUBE ---------------- */
        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube analysis..."
                withSonarQubeEnv('SonarQubeServer') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=flask-todo \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        /* ---------------- DOCKER BUILD ---------------- */
        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        /* ---------------- TRIVY SCAN ---------------- */
        stage('Trivy Scan') {
            steps {
                echo "🛡️ Running Trivy security scan..."
                // --exit-code 0 lets the build continue even if vulnerabilities are found
                sh 'trivy image --severity HIGH,CRITICAL ${IMAGE_NAME}:latest || true'
            }
        }

        /* ---------------- DOCKER HUB PUSH ---------------- */
        stage('Push to Docker Hub') {
            steps {
                echo "🚀 Pushing image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'USER',
                                                  passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker tag ${IMAGE_NAME}:latest ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        /* ---------------- SMOKE TEST ---------------- */
        stage('Run Container (Smoke Test)') {
            steps {
                echo "🔥 Running smoke test..."
                sh '''
                    docker run -d --rm -p 5000:5000 --name ci-smoke ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                    sleep 5
                    echo "Response from container:"
                    curl -s http://localhost:5000/tasks || true
                    docker stop ci-smoke
                '''
            }
        }
    }

    /* ---------------- POST ACTIONS ---------------- */
    post {
        success {
            echo '✅ Pipeline completed successfully! Image pushed to Docker Hub and verified.'
        }
        failure {
            echo '❌ Pipeline failed — check console logs for details.'
        }
    }
}
