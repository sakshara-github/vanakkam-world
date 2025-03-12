pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '529088272063'      // Your AWS account ID
        AWS_REGION = 'eu-north-1'            // AWS Region
        AWS_ACCESS_KEY_ID = credentials('AWS_Jenkins_Access_Key_ID')  // Access Key ID from Jenkins credentials
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Jenkins_Secret_Access_Key')  // Secret Access Key from Jenkins credentials
        ECR_REPO_NAME = 'vw-repo'            // ECR Repository Name
        IMAGE_TAG = 'latest'                 // Image tag for the Docker image
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"  // Full ECR URL
        GIT_BRANCH = 'master'                // Git branch to checkout (change to 'master' if that is your default branch)
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'  // Git repository URL
    }
    tools{
        maven 'mymaven'
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
     
                            

        
  
