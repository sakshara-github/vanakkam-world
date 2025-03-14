pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "my-vanakkam-repo"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build & Push to ECR') {
            steps {
                script {
                    def accountId = sh(script: "aws sts get-caller-identity --query Account --output text", returnStdout: true).trim()
                    def ecrUri = "${accountId}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"

                    // Authenticate to AWS ECR
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ecrUri}"

                    // Build and push the Docker image
                    sh "docker build -t ${ecrUri}:${IMAGE_TAG} ."
                    sh "docker push ${ecrUri}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(['ec2-key']) {
                        sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@your-ec2-ip << EOF
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ecrUri}
                        docker pull ${ecrUri}:${IMAGE_TAG}
                        docker stop my-container || true
                        docker rm my-container || true
                        docker run -d --name my-container -p 80:8080 ${ecrUri}:${IMAGE_TAG}
                        EOF
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed!"
        }
        success {
            echo "Deployment successful!"
        }
    }
}
