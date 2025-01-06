pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub'('dockerhub-credentials') // DockerHub credentials configured in Jenkins
        DOCKERHUB_REPO = 'sudhakshara/vw-image'
        DOCKER_IMAGE_TAG = 'latest'
        KUBECONFIG = '/home/sudha_cubensquare/.kube/config'// Kubernetes kubeconfig credentials
    }
    tools {
    maven 'mymaven'
    }
    stages {
        stage('Checkout Code') {
            steps {
                echo 'Cloning the repository...'
                git url: 'https://github.com/sudhakshara/vanakkam-world', branch: 'master'
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building the project using Maven...'
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t $DOCKERHUB_REPO:$DOCKER_IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $DOCKERHUB_REPO:$DOCKER_IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying the application to Kubernetes...'
              
                        export KUBECONFIG=$KUBECONFIG
                        kubectl apply -f tomcat.yaml
                      
                    '''
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
