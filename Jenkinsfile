pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '529088272063'  // Your AWS account ID
        AWS_REGION = 'eu-north-1'        // AWS Region
        // Use Jenkins credentials to fetch the AWS Access Key and Secret Access Key securely
        AWS_ACCESS_KEY_ID = credentials('AWS_Jenkins_Access_Key_ID')  // Access Key ID from Jenkins credentials
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Jenkins_Secret_Access_Key')  // Secret Access Key from Jenkins credentials
        ECR_REPO_NAME = 'vw-repo'        // ECR Repository Name
        IMAGE_TAG = 'latest'             // Image tag for the Docker image
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"  // Full ECR URL
        GIT_BRANCH = 'main'              // Git branch to checkout
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'  // Git repository URL
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code from GitHub
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    echo 'Building Docker Image...'
                    sh 'docker build -t ${REPO_URL} .'
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    // Login to AWS ECR using the AWS CLI
                    echo 'Logging in to AWS ECR...'
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    // Push Docker image to ECR
                    echo 'Pushing Docker Image to ECR...'
                    sh 'docker push ${REPO_URL}'
                }
            }
        }

        stage('Pull Image from ECR') {
            steps {
                script {
                    // Pull the Docker image from ECR
                    echo 'Pulling Docker Image from ECR...'
                    sh 'docker pull ${REPO_URL}'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

        
  
