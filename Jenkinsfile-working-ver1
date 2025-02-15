pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "141282679348"
        AWS_REGION = "ap-northeast-2"
        ECR_REPO = "helmnodejenkeks"
        IMAGE_NAME = "helm-node-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: '2efe5291-5873-4b0d-8c73-7ef1d2bbfa7b', url: 'https://github.com/akshay-dvps/helm-node-jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }

        stage('Tag and Push Image to ECR') {
            steps {
                script {
                    sh "docker tag ${IMAGE_NAME}:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"
                }
            }
        }
    }
}
