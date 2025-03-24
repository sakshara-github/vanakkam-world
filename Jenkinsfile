pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '231552173810'
        AWS_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_Jenkins_Credentials')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Jenkins_Credentials')
        ECR_REPO_NAME = 'vanakkam-repo'
        IMAGE_TAG = 'latest'
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        SSH_KEY = credentials('ec2-ssh-credentials-updated')
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-34-236-152-71.compute-1.amazonaws.com'
        CONTAINER_NAME = "vanakkam-container"
        GIT_CREDENTIALS_ID = 'git-token'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/sakshara-github/vanakkam-world.git',
                        credentialsId: GIT_CREDENTIALS_ID
                    ]]
                ])
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker build --no-cache -t 231552173810.dkr.ecr.us-east-1.amazonaws.com/vanakkam-repo:latest .
                    docker push 231552173810.dkr.ecr.us-east-1.amazonaws.com/vanakkam-repo:latest

                """
            }
        }

        
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-credentials-updated']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "bash -c '
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com &&
                            docker stop ${CONTAINER_NAME} || true &&
                            docker rm ${CONTAINER_NAME} || true &&
                            docker pull ${REPO_URL} &&
                            docker run -d --name ${CONTAINER_NAME} -p 89:8080 ${REPO_URL}
                        '"
                    """
                }
            }
        }
    }
}
