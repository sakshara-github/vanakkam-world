pipeline {
    agent any

    environment {
        // Set Docker image name and tag
        DOCKER_IMAGE_NAME = 'your-docker-image-name'
        DOCKER_TAG = 'latest' // or specify your version

        // Kubernetes configuration
        KUBERNETES_CLUSTER = 'your-kubernetes-cluster'
        KUBE_CONFIG = '/path/to/your/kubeconfig'  // If you use a custom kubeconfig path
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from the repository
                checkout scm
            }
        }

        stage('Build WAR') {
            steps {
                script {
                    // Build your web application WAR file (optional, if not prebuilt)
                    // Adjust the command based on your build system (Maven, Gradle, etc.)
                    sh 'mvn clean package -DskipTests' // Maven build, adjust if needed
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image from the Dockerfile
                    sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to a repository (DockerHub or other)
                    // Make sure you're logged in to the Docker registry before this step
                    // You can configure Jenkins credentials for Docker login
                    sh """
                        docker login -u your-docker-username -p your-docker-password
                        docker push ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                script {
                    // Pull the Docker image from the registry
                    sh """
                        docker pull ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Set up Kubernetes context (use KUBECONFIG if needed)
                    sh """
                        export KUBECONFIG=${KUBE_CONFIG}  // Optional if your Kube config is in a custom location
                        kubectl config use-context ${KUBERNETES_CLUSTER}
                    """
                    
                    // Apply the Kubernetes Deployment configuration (assuming you have a Kubernetes YAML file for deployment)
                    // For example, you would define the Kubernetes Deployment manifest in your project
                    sh """
                        kubectl apply -f tomcat.yaml
                    """
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
