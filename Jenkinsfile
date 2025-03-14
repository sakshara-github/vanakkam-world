pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' // Change as needed
        AWS_ACCOUNT_ID = '529088272063' // Replace with your AWS Account ID
        ECR_REPO_NAME = 'my-vanakkam-repo'
        IMAGE_TAG = "latest"
        DOCKER_IMAGE = "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG"
        EC2_USER = 'ubuntu'
        EC2_HOST = '15.207.88.224' // Replace with EC2 instance's public IP
        SSH_CREDENTIALS_ID = 'EC2_SSH_KEY'
        AWS_CREDENTIALS_ID = 'AWS_CREDENTIALS_ID'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/sakshara-github/vanakkam-world.git' // Replace with your repo
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: AWS_CREDENTIALS_ID, usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_REGION
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    """
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
                sh "docker tag $DOCKER_IMAGE $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG"
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh "docker push $DOCKER_IMAGE"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST << 'EOF'
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    docker pull $DOCKER_IMAGE
                    docker stop my-container || true
                    docker rm my-container || true
                    docker run -d --name my-container -p 8080:8080 $DOCKER_IMAGE
                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
