pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '34.215.221.166'  // Your EC2 instance public/private IP
        CONTAINER_NAME = 'my-container'  // Name of your running container
        IMAGE_NAME = 'vanakkam-world-repo'  // Example: my-ecr-repo:latest (already on EC2)
        AWS_REGION = 'us-west-2'  // AWS region (e.g., us-east-1)
        AWS_ACCOUNT_ID = '231552173810'  // AWS Account ID

        SSH_CREDENTIALS_ID = 'ssh-credentials'  // Jenkins SSH credential ID
        AWS_CREDENTIALS_ID = 'aws-credentials'  // Jenkins AWS credential ID
        GIT_CREDENTIALS_ID = 'github'  // Jenkins GitHub credential ID
        GIT_REPO_URL = 'https://github.com/sakshara-github/vanakkam-world.git'  // GitHub repo
        GIT_BRANCH = 'master'  // Branch to checkout
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                git branch: GIT_BRANCH,
                    credentialsId: GIT_CREDENTIALS_ID,
                    url: GIT_REPO_URL
            }
        }

        stage('Restart Container on EC2') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            echo "Logging into AWS ECR..."
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                            
                            echo "Stopping existing container..."
                            docker stop ${CONTAINER_NAME} || true

                            echo "Removing existing container..."
                            docker rm ${CONTAINER_NAME} || true

                            echo "Restarting container with existing image..."
                            docker run -d --name ${CONTAINER_NAME} -p 83:8080 ${IMAGE_NAME}
                        EOF
                    """
                }
            }
        }
    }
}
