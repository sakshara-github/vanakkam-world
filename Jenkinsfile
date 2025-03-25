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
        GIT_BRANCH = 'master'
        GIT_REPO = 'https://github.com/sakshara-github/vanakkam-world.git'
        SSH_KEY = credentials('ec2-ssh-credentials-updated')
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-34-203-198-246.compute-1.amazonaws.com'
        GIT_CREDENTIALS_ID = 'git-token' // ID of the stored credentials in Jenkins
        CONTAINER_NAME = "vanakkam-container"
    }

    stages {
        stage('Checkout') {
            steps {
                     checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/cubensquare/fms-wex.git',
                        credentialsId: GIT_CREDENTIALS_ID
                    ]]
                ])
            }
        }

        stage('Build with Maven') {
            steps {
                // Use Maven to build the project
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                sh "docker build -t ${REPO_URL} ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                // Log in to AWS ECR
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                // Push the Docker image to AWS ECR
                sh "docker push ${REPO_URL}"
            }
        }

        stage('Deploy to EC2') {
            steps {
        // Use SSH agent with credentials
        sshagent(['ec2-ssh-credentials-updated']) {
            sh """
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                       # Log in to ECR
                        echo "Logging into ECR..."
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        
                        # Check if the container exists and stop/remove if running
                        echo "Checking if container '${CONTAINER_NAME}' is running..."
                        if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                            echo "Stopping container '${CONTAINER_NAME}'..."
                            docker stop ${CONTAINER_NAME}
                        fi

                        echo "Checking if container '${CONTAINER_NAME}' exists..."
                        if [ \$(docker ps -a -q -f name=${CONTAINER_NAME}) ]; then
                            echo "Removing container '${CONTAINER_NAME}'..."
                            docker rm ${CONTAINER_NAME}
                        fi

                        # Pull the latest Docker image
                        echo "Pulling Docker image..."
                        docker pull ${REPO_URL}
                        
                        # Run the container
                        echo "Running new container '${CONTAINER_NAME}'..."
                        docker run -d --name ${CONTAINER_NAME} -p 92:8080 ${REPO_URL}
                    """
        }
    }
}
    }
}
