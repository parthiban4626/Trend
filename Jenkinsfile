pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "parthiban46/trend-app"
        IMAGE_TAG = "latest"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER_NAME = "trend-eks-cluster"
        GIT_CREDENTIALS_ID = "github-creds"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/parthiban4626/Trend.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}

