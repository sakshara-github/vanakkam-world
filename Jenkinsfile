pipeline {
    agent any
    environment {
        // Set Docker Hub credentials 
        DOCKER_REGISTRY = "docker.io" 
        DOCKER_IMAGE = "sudhakshara/webapp"  
        DOCKER_TAG = "latest"
        DOCKER_CREDENTIALS = 'dockerhub' 
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub or another source
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Build flask Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
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
   

       
