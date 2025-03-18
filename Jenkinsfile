pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '231552173810'
        AWS_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESSKEY_ID') // Ensure this is correctly set up
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRETKEY_ID') // Ensure this is correctly set up
        ECR_REPO_NAME = 'jenkins-rpo'
        IMAGE_TAG = 'latest'
        REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        GIT_BRANCH = 'master'
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-54-209-123-35.compute-1.amazonaws.com'
        GIT_CREDENTIALS_ID = 'github' // ID of the stored credentials in Jenkins
        CONTAINER_NAME = "my-container"
        SSH_KEY_ID = 'ssh-key' // Added SSH key ID
    }

    tools {
        maven 'mymaven'  // Replace 'mymaven' with the exact name of your Maven installation in Jenkins
    }

    stages {
        stage('Checkout Code') {
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
                sh 'mvn clean install'
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
            steps {
                sh "docker build -t ${REPO_URL} ."
                sh "docker push ${REPO_URL}"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([SSH_KEY_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                    docker pull ${REPO_URL} &&
                    docker stop ${CONTAINER_NAME} || true &&
                    docker rm ${CONTAINER_NAME} || true &&
                    docker run -d --name ${CONTAINER_NAME} -p 8084:8080 ${REPO_URL}
                    '
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
