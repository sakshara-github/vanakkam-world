pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '529088272063'
        AWS_REGION = 'eu-north-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_Jenkins_Access_Key_ID') // Ensure this is correctly set up
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Jenkins_Secret_Access_Key') // Ensure this is correctly set up
        ECR_REPO_NAME = 'vw-repo'
        IMAGE_TAG = 'latest'
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        GIT_BRANCH = 'master' // Corrected the branch name from 'main ' to 'main'
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'
        SSH_KEY = credentials('ec2-ssh-credentials-updated')
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-13-53-36-200.eu-north-1.compute.amazonaws.com'
        GIT_CREDENTIALS_ID = 'crendentials' // ID of the stored credentials in Jenkins
        CONTAINER_NAME = "my-vw-container"
    }

    tools {
        // Specify the Maven version installed on Jenkins
        maven 'mymaven'  // Replace 'mymaven' with the exact name of your Maven installation in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${GIT_REPO}",
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building the project using Maven...'
                // Use the 'mymaven' tool to build the project
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t ${REPO_URL} ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                echo 'Logging in to AWS ECR...'
                // Log in to AWS ECR
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                echo 'Pushing Docker Image to AWS ECR...'
                sh "docker push ${REPO_URL}"
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
