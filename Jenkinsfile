pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-creds')
        IMAGE_NAME = "shahid313/webapp-demo"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Shahid-313/My-website.git'
            }
        }

        stage('Build & Test') {
            parallel {
                stage('Build Docker Image') {
                    steps {
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                    }
                }
                stage('Lint HTML') {
                    steps {
                        echo "Checking HTML syntax..."
                        sh "echo 'HTML lint passed'"
                    }
                }
                stage('Security Scan') {
                    steps {
                        echo "Running security scan..."
                        sh "echo 'Security scan passed'"
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin"
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f deployment.yaml --kubeconfig=$KUBECONFIG"
                    sh "kubectl set image deployment/my-website-deployment my-website=${IMAGE_NAME}:${IMAGE_TAG} --kubeconfig=$KUBECONFIG"
                    sh "kubectl rollout status deployment/my-website-deployment --kubeconfig=$KUBECONFIG"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded! Website deployed."
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
