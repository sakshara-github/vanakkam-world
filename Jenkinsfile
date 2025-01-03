pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sudhakshara/vw-image"
        DOCKER_TAG = "latest"
        KUBECONFIG = '/home/sudha_cubensquare/.kube/config' 
        DOCKER_CREDENTIALS = 'dockerhub'
    }
     tools {
    maven 'mymaven'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub or another source
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git'
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }
        stage('Build with maven') {
            steps {
                 echo 'Building the project using Maven...'
                sh 'mvn clean install'
            }
        }
        stage('Push to Dockerhub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/',  DOCKER_CREDENTIALS) {
                        sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                        
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                        sh "kubectl apply -f tomcat.yaml"
                    }
                }
            }
     }
}      
