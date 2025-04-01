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
        EC2_HOST = 'ec2-54-88-95-224.compute-1.amazonaws.com'
        CONTAINER_NAME = "vanakkam-container"
        GIT_CREDENTIALS_ID = 'git-token'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/sakshara-github/vanakkam-world.git',
                        credentialsId: GIT_CREDENTIALS_ID
                    ]]
                ])
            }
        }

        stage('Check for Relevant Changes') {
            steps {
                script {
                    def changedFiles = sh(script: "git diff --name-only HEAD~1 | grep -E '(Dockerfile|index.html)' || echo ''", returnStdout: true).trim()
                    
                    if (changedFiles) {
                        echo "Changes detected in: ${changedFiles}. Image will be rebuilt."
                        env.BUILD_IMAGE = "true"
                    } else {
                        echo "No relevant changes detected. Skipping image build."
                        env.BUILD_IMAGE = "false"
                    }
                }
            }
        }

        stage('Build Maven Project') {
            steps {
                script {
                    def mvnHome = tool name: 'maven', type: 'maven'
                    sh "${mvnHome}/bin/mvn clean install"
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Build and Push Docker Image') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                sh """
                    docker build -t ${REPO_URL} .
                    docker push ${REPO_URL}
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
                            
                            docker pull ${REPO_URL} &&
                            
                            docker stop ${CONTAINER_NAME} || true &&
                            docker rm ${CONTAINER_NAME} || true &&
                            
                            docker run -d --name ${CONTAINER_NAME} --restart always -p 92:8080 ${REPO_URL}
                        '"
                    """
                }
            }
        }
    }
}
