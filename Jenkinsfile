pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-creds')
        IMAGE_NAME = "shahid313/webapp-demo"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Shahid-313/My-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Docker image build karna
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Docker Hub me login aur push
                    sh "echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    // Kubernetes me deploy karna
                    sh "kubectl apply -f deployment.yaml --kubeconfig=$KUBECONFIG"
                    // Naya image rollout karna
                    sh "kubectl set image deployment/my-website-deployment my-website=${IMAGE_NAME}:${IMAGE_TAG} --kubeconfig=$KUBECONFIG"
                    sh "kubectl rollout status deployment/my-website-deployment --kubeconfig=$KUBECONFIG"
                }
            }
        }
    }
}
