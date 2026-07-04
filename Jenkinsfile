pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "eldho10/cicd-demo-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Eldho2827/cicd-demo-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh "kubectl set image deployment/cicd-demo-app cicd-demo-app=${DOCKER_IMAGE}:${DOCKER_TAG} --record"
                    sh "kubectl apply -f k8s/service.yaml"
                }
            }
        }

        stage('Verify Rollout') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh "kubectl rollout status deployment/cicd-demo-app --timeout=120s"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully - zero-downtime deployment done.'
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
        }
    }
}
