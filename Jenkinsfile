pipeline {
    agent any
    environment {
        // Set Docker Hub credentials (if using Docker Hub)
        DOCKER_REGISTRY = "docker.io" // Example: "docker.io"
        DOCKER_IMAGE = "sudhakshara/webapp"  // Example: "yourusername/yourapp"
        DOCKER_TAG = "latest"
        
        // Jenkins credentials for Docker login
        DOCKER_CREDENTIALS = 'dockerhub' // Jenkins credentials ID for Docker Hub login
    }
    stages {
        stage('Pull Docker Image') {
            steps {
                script {
                    // Pull the base Docker image (this may not be necessary as you are building your own image)
                    sh 'docker pull tomcat:8-jre8'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the custom Docker image
                    sh 'docker build -t $DOCKER_REGISTRY/$DOCKER_IMAGE:$DOCKER_TAG .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                     docker.withRegistry('https://index.docker.io/v1/',  DOCKER_CREDENTIALS) {
                        sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                    }

                   
                }
            }
        }
        stage('Deploy Using YAML') {
            steps {
                script {
                    // Assuming you're deploying to Kubernetes using kubectl and a deployment.yaml
                    // If using Docker Compose, this step can be modified accordingly
                    sh 'kubectl apply -f tomcat.yaml'
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment finished successfully!'
        }
        failure {
            echo 'Something went wrong during the deployment.'
        }
    }
}

   

       
