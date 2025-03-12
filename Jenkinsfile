pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '529088272063'
        AWS_REGION = 'eu-north-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_Jenkins_Access_Key_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Jenkins_Secret_Access_Key')
        ECR_REPO_NAME = 'vw-repo'
        IMAGE_TAG = 'latest'
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        GIT_BRANCH = 'master'
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'
    }

    tools {
        // Specify the Maven version installed on Jenkins
        maven 'mymaven'  // Replace 'mymaven' with the exact name of your Maven installation
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'your-credentials-id', branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building the project using Maven...'
                sh 'mvn clean install'
            }
        }

        stage('Verify Docker Installation') {
            steps {
                script {
                    // Verify Docker installation
                    sh 'docker --version'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
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

      
