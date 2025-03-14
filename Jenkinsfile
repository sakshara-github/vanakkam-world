pipeline {
    agent any
    
    environment {
        AWS_ACCOUNT_ID = "529088272063"
        AWS_REGION = "ap-south-1"
        ECR_REPOSITORY = "my-vanakkam-repo"
        IMAGE_TAG = "latest"
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
    }

    tools {
        maven 'mymaven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Authenticate to AWS ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Build & Push Docker Image') {
  
