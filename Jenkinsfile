pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2' // AWS Region
        AWS_ACCOUNT_ID = '231552173810' // Your AWS account ID
        ECR_REPO = 'vanakkam-world-repo' // ECR repository name
        CONTAINER_NAME = 'my-app' // Running container name
        PORT_MAPPING = '8085:8080' // Port mapping
        EC2_HOST = 'ubuntu@35.88.122.90' // EC2 instance details
        GITHUB_REPO = 'https://github.com/sakshara-github/vanakkam-world.git' // GitHub repository
        BRANCH = 'master' // GitHub branch
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                script {
                    // Clone the GitHub repository using SSH key stored in Jenkins
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/$BRANCH"]],
                        userRemoteConfigs: [[
                            url: GITHUB_REPO,
                            credentialsId: 'github' // SSH Key credential ID from Jenkins
                        ]]
                    ])
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-credentials', region: "$AWS_REGION")]) {
                    script {
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"
                    }
                }
            }
        }

        stage('Pull Latest Image from ECR') {
            steps {
                script {
                    sh "docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    // Connect to EC2 and deploy the container
                    sshagent(['ssh-credentials']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no $EC2_HOST <<EOF
                            docker stop $CONTAINER_NAME || true
                            docker rm $CONTAINER_NAME || true
                            docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest
                            docker run -d --name $CONTAINER_NAME -p $PORT_MAPPING $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest
                            docker image prune -f
                        EOF
                        """
                    }
                }
            }
        }
    }
}
