pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "parthiban46/trend-app"  // your DockerHub repo
        GIT_CREDENTIALS_ID = 'github-creds'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        AWS_CREDENTIALS_ID = 'aws-creds'
        CLUSTER_NAME = "trend-eks-cluster"
        CLUSTER_REGION = "ap-south-1"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/parthiban4626/Trend.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh """
                        aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${CLUSTER_REGION}
                        kubectl apply -f k8s/
                    """
                }
            }
        }
    }
[O
    post {
        failure {
            echo 'Build failed.'
        }
        success {
            echo 'Application successfully deployed!'
        }
    }
}











