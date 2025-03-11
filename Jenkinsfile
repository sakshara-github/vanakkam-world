pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub'
                               // DockerHub username
        DOCKER_IMAGE = "sudhakshara/vw-image"
        DOCKER_IMAGE_TAG = "latest"
    }

    tools {
        maven 'mymaven'  // Ensure this tool is configured in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building the project using Maven...'
                sh 'mvn clean install'
            }
        }

        stage('Verify Docker Installation') {
            steps {
                script {
                    // Verify Docker installation
                    sh 'docker --version'
                }
            }
        }

        stage('Build the Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        stage('Push to Dockerhub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        sh "docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
