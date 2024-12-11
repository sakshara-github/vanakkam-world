pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sudhakshara/my-webapp"
        DOCKER_TAG = "latest"         // Corrected line
        DOCKER_USERNAME = 'sudhakshara'
        DOCKER_CREDENTIALS = 'dockerhub'
        KUBECONFIG = '/home/sudha_cubensquare/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from the repository
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git',   // Replace with your Git repository URL
            }
        }

        stage('Build Tomcat Image') {
            steps {
                script {
                    // Build the Docker image from Dockerfile
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to Docker registry
                    docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_CREDENTIALS') {
                        sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy the application using kubectl
                    sh "kubectl apply -f tomcat.yaml"
                }
            }
        }
    }
}
 
    

    

