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
        GIT_BRANCH = 'master'
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-13-53-36-200.eu-north-1.compute.amazonaws.com'
        GIT_CREDENTIALS_ID = 'crendentials' // ID of the stored credentials in Jenkins
        CONTAINER_NAME = "my-vw-container"
    }

    tools {
        maven 'mymaven'  // Replace 'mymaven' with the exact name of your Maven installation in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*master/${GIT_BRANCH}"]],
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
                sshagent(['your-ssh-key-id']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                    docker pull ${REPO_URL} &&
                    docker stop ${CONTAINER_NAME} || true &&
                    docker rm ${CONTAINER_NAME} || true &&
                    docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${REPO_URL}
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
