pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '231552173810'
        AWS_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_Jenkins_Credentials')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Jenkins_Credentials')
        ECR_REPO_NAME = 'vanakkam-repo'
        SSH_KEY = credentials('ec2-ssh-credentials-updated')
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-54-242-160-90.compute-1.amazonaws.com'
        CONTAINER_NAME = "vanakkam-container"
        GIT_CREDENTIALS_ID = 'git-token'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()  // Remove all old files to prevent cache issues
            }
        }

        stage('Checkout Latest Code') {
            steps {
                script {
                    checkout([$class: 'GitSCM',
                        branches: [[name: '*/master']],
                        userRemoteConfigs: [[
                            url: 'https://github.com/sakshara-github/vanakkam-world.git',
                            credentialsId: GIT_CREDENTIALS_ID
                        ]]])
                    sh "git reset --hard origin/master"
                    sh "git pull origin master"
                }
            }
        }

        stage('Build Maven Project') {
            steps {
                script {
                    def mvnHome = tool name: 'maven', type: 'maven'
                    sh "${mvnHome}/bin/mvn clean package"  // Ensure .war file is generated
                }
            }
        }

        stage('Verify WAR File') {
            steps {
                script {
                    if (!fileExists('webapp/target/webapp.war')) {
                        error("WAR file is missing! Build failed.")
                    }
                    echo "WAR file exists. Proceeding with Docker build."
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

        stage('Force Build & Push Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.IMAGE_TAG}"
                }
                sh """
                    echo "Building Docker Image: ${REPO_URL}"
                    docker build --no-cache -t ${REPO_URL} .
                    docker push ${REPO_URL}
                    echo "Successfully pushed image: ${REPO_URL}"
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-credentials-updated']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "bash -c '
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                            
                            docker stop ${CONTAINER_NAME} || true
                            docker rm -f ${CONTAINER_NAME} || true
                            docker rmi ${REPO_URL} || true
                            
                            docker pull ${REPO_URL}
                            
                            docker run -d --name ${CONTAINER_NAME} --restart always -p 94:8080 ${REPO_URL}
                        '"
                    """
                }
            }
        }
    }
}
