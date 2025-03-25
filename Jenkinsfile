pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '231552173810'
        AWS_REGION = 'us-east-1'
        ECR_REPO_NAME = 'vanakkam-repo'
        IMAGE_TAG = 'latest'
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        GIT_CREDENTIALS_ID = 'git-token'
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-34-203-198-246.compute-1.amazonaws.com'
        SSH_KEY = credentials('ec2-ssh-credentials-updated')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', credentialsId: GIT_CREDENTIALS_ID, url: 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                sh """
                    docker build -t ${REPO_URL} .
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker push ${REPO_URL}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-credentials-updated']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            docker pull ${REPO_URL}
                            docker stop vanakkam-container || true
                            docker rm vanakkam-container || true
                            docker run -d --name vanakkam-container -p 92:8080 ${REPO_URL}
                        EOF
                    """
                }
            }
        }
    }
}
