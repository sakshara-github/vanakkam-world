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
                    // Build the WAR file using Maven (or your build system)
                    // Adjust the command if you're using Gradle or another build tool
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                  
                       sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ."
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/',  DOCKER_CREDENTIALS) {
                   
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
