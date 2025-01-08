

       
   pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub'  // DockerHub username
        DOCKERHUB_REPO = 'sudhakshara/vanakkam-image'
        DOCKER_IMAGE_TAG = 'latest'
        KUBECONFIG = '/home/sudha_cubensquare/.kube/config' // Kubernetes kubeconfig credentials
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

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f tomcat.yaml"
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
