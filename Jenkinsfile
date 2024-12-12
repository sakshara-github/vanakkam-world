
       pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "sudhakshara/webapp"
        DOCKER_TAG = "latest"
        DOCKER_CREDENTIALS = 'dockerhub'
        KUBE_CONFIG = '/home/sudha_cubensquare/.kube/config'  
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the latest code from the repository
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Build WAR') {
            steps {
                script {
                    // Use Maven Docker image to build the WAR file
                    docker.image('maven:3.8.6-openjdk-11').inside {
                        // Build the WAR file using Maven
                        sh 'mvn clean package -DskipTests'
                           
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login to Docker registry and push the image
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        sh 'docker push ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy the application using the Kubernetes YAML file
                    sh 'kubectl apply -f tomcat.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
