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
                    sh "git reset --hard origin/master"  // Ensures we are on the latest commit
                    sh "git fetch --all"
                    sh "git pull origin master"  // Force update to latest code
                }
            }
        }

        stage('Check for Changes') {
            steps {
                script {
                    def changedFiles = sh(script: "git diff --name-only HEAD~1 HEAD", returnStdout: true).trim()
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.IMAGE_TAG}"

                    if (changedFiles) {
                        echo "Changes detected in: ${changedFiles}. Image will be rebuilt."
                        env.BUILD_IMAGE = "true"
                    } else {
                        echo "No changes detected. Skipping build."
                        env.BUILD_IMAGE = "false"
                    }
                }
            }
        }

        stage('Force Build & Push Docker Image') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                sh """
                    echo "Building Docker Image: ${REPO_URL}"
                    docker build --no-cache -t ${REPO_URL} .
                    docker push ${REPO_URL}
                    echo "Successfully pushed image: ${REPO_URL}"
                """
            }
        }

        stage('Verify Image on ECR') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                sh """
                    echo "Checking if image exists in ECR..."
                    aws ecr list-images --repository-name ${ECR_REPO_NAME} --region ${AWS_REGION}
                """
            }
        }

        stage('Deploy to EC2') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                sshagent(['ec2-ssh-credentials-updated']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "bash -c '
                            echo "Logging in to ECR..."
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                            
                            echo "Stopping old container..."
                            docker stop ${CONTAINER_NAME} || true
                            
                            echo "Removing old container..."
                            docker rm -f ${CONTAINER_NAME} || true
                            
                            echo "Removing old Docker image..."
                            docker rmi ${REPO_URL} || true
                            
                            echo "Pulling new image..."
                            docker pull ${REPO_URL}
                            
                            echo "Running new container..."
                            docker run -d --name ${CONTAINER_NAME} --restart always -p 94:8080 ${REPO_URL}
                            
                            echo "Deployment complete."
                        '"
                    """
                }
            }
        }
    }
}
