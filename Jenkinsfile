pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-2'
        ECR_REPO = '141282679348.dkr.ecr.ap-northeast-2.amazonaws.com'
        IMAGE_NAME = 'helm-node-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate with AWS ECR') {
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    """
                }
            }
        }

        stage('Get Latest Image Tag & Increment') {
            steps {
                script {
                    // Fetch the latest tag from ECR
                    def latestTag = sh(script: """
                        aws ecr describe-images --repository-name $IMAGE_NAME --region $AWS_REGION \
                        --query 'sort_by(imageDetails,&imagePushedAt)[-1].imageTags[0]' --output text
                    """, returnStdout: true).trim()

                    echo "Latest tag from ECR: ${latestTag}"

                    // Check if latestTag is empty (first build case)
                    if (latestTag == "None" || !latestTag.startsWith("v")) {
                        latestTag = "v1.0"
                    } else {
                        def parts = latestTag.tokenize('.')
                        def major = parts[0] // 'v1'
                        def minor = parts[1] as Integer // '0'
                        minor += 1 // Increment minor version
                        latestTag = "${major}.${minor}"
                    }

                    echo "New Image Tag: ${latestTag}"
                    env.IMAGE_TAG = latestTag // Set the new image tag as an environment variable
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    """
                }
            }
        }

        stage('Tag and Push to ECR') {
            steps {
                script {
                    sh """
                        docker tag $IMAGE_NAME:$IMAGE_TAG $ECR_REPO/$IMAGE_NAME:$IMAGE_TAG
                        docker push $ECR_REPO/$IMAGE_NAME:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Verify Pushed Image') {
            steps {
                script {
                    sh """
                        aws ecr list-images --repository-name $IMAGE_NAME --region $AWS_REGION
                    """
                }
            }
        }
    }
}

